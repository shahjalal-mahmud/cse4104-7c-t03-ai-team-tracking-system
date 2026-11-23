# 09 – AI Integration Specification

This file defines exactly what is sent to Gemini, how commits are referenced, what comes back, and how it is checked. Prompt version: `v1.0`.

## 1. Purpose and principles

- The AI reads **commit metadata only** (no code) and writes an advisory summary and a small number of evidence-backed flags for the Instructor.
- AI output is a draft. It never creates, changes, approves or publishes a grade, never suggests scores, and never changes a status except `ai_reviewed`.
- Students never see AI content.
- Everything sent to Gemini is anonymised and sanitised first. Nothing that identifies a person, a team or a repository is sent.

## 2. When a review runs

Only an Instructor can request a review. Preconditions and limits (at least one counted commit, one pending review per submission, 10 requests per Instructor per hour, newest 400 commits) are defined in the business rules and the API specification. The review covers the analysis period of the submission's milestone.

## 3. Input: the only fields that are sent

The request body is built from this structure and from nothing else. Field names and order are fixed.

```json
{
  "promptVersion": "v1.0",
  "milestone": { "title": "Proposal", "guidelines": "sanitised text, at most 3000 characters" },
  "period": { "start": "2026-09-01T18:00:00Z", "end": "2026-10-01T17:59:00Z" },
  "team": { "countedCommits": 52, "activeDays": 14, "unassignedCommits": 3 },
  "members": [
    { "label": "Member A", "commits": 20, "activeDays": 8, "additions": 900,
      "deletions": 120, "filesChanged": 40, "sharePercent": 38.5 }
  ],
  "commits": [
    { "ref": "c1", "author": "Member A", "authoredAt": "2026-09-02T09:15:00Z",
      "message": "sanitised first line, at most 200 characters",
      "additions": 40, "deletions": 2, "filesChanged": 3 }
  ]
}
```

Rules:

- `members` lists every team member plus every `Unassigned Author n` label that appears in `commits`. Unassigned labels have the same metric fields.
- `commits` holds the newest 400 counted commits of the period, **ordered oldest first**, numbered `c1`, `c2`, … in that order.
- Never sent: names, GitHub usernames, email addresses, SHAs, commit URLs, repository URL or name, team name, file paths, file contents, diffs, branch names, student IDs, user IDs.

## 4. Anonymisation and sanitising

Applied to every commit message and to the milestone guidelines before the request is built, in this order:

1. **Labels.** Each team member gets a label by sorting members by student ID and naming them `Member A`, `Member B`, … Commit authors that are not matched to a member get `Unassigned Author 1`, `Unassigned Author 2`, … in order of first appearance (matched by author login, otherwise by author email). The mapping label → user ID (or `null` for unassigned authors) is stored in `ai_reports.label_map`. It never contains names.
2. **Identity replacement.** Every occurrence (case-insensitive) of a member's full name, GitHub username or registered email in the text is replaced by that member's label. Every occurrence of the team name is replaced by `[team]`.
3. **First line only.** For commit messages, keep only the first line.
4. **Strip risky text.** Replace with `[removed]`:
   - URLs (`http://`, `https://`, `www.`);
   - email addresses;
   - any token that contains `/` or `\`;
   - any token that ends in a file extension from this list: `js jsx ts tsx py java php css html json md sql yml yaml env`.
5. **Cap length.** Commit messages at 200 characters; guidelines at 3,000 characters.

A unit test builds a fixture whose messages and guidelines contain a member name, username, email, repository URL, team name and file path, runs the whole input builder, and asserts that none of them appears anywhere in the serialised request body.

## 5. Commit references

- Gemini only ever sees refs (`c1` … `cN`), never SHAs.
- The builder keeps a map ref → SHA for the duration of the job (it can be rebuilt from the same ordered list). It is not sent and not stored.
- When the response is processed, each cited ref is checked against the map. Unknown refs make the flag invalid. Valid refs are replaced by SHAs before the flag is stored in `ai_flags.evidence_commit_shas`.

## 6. Output: fixed JSON schema

Gemini must answer with JSON only, in exactly this shape (validated with zod; unknown keys are rejected):

```json
{
  "summary": "string, 1 to 1500 characters",
  "memberSummaries": [
    { "label": "Member A", "summary": "string, 1 to 600 characters" }
  ],
  "flags": [
    {
      "flagType": "low_information_commit_messages | off_topic_commits | repetitive_trivial_commits",
      "severity": "low | medium | high",
      "memberLabel": "Member A or null",
      "title": "string, 1 to 100 characters",
      "description": "string, 1 to 600 characters",
      "evidenceRefs": ["c3", "c7"]
    }
  ]
}
```

Limits: at most 10 flags; `memberSummaries` has at most one entry per label; `evidenceRefs` has 1 to 20 entries.

Flag meanings:

| `flagType`                        | Meaning                                                                |
| --------------------------------- | ---------------------------------------------------------------------- |
| `low_information_commit_messages` | Messages that say nothing useful (for example "update", "fix", "asdf") |
| `off_topic_commits`               | Commits whose message does not fit the milestone title and guidelines  |
| `repetitive_trivial_commits`      | Many near-identical, very small commits that look like padding         |

## 7. Checks after the response

1. Parse the text as JSON and validate it with the schema. If invalid: retry **once** with the validation error appended to the prompt; if still invalid, the report is `failed` with `AI_INVALID_OUTPUT`.
2. For every flag: it must have at least one ref and every ref must exist in the sent input; `memberLabel` must be `null` or a label present in the input. A flag that fails is dropped and counted in `droppedFlagCount`.
3. Member summaries whose label is not in the input are ignored.
4. In one transaction: store `ai_reports.result` (`summary`, `memberSummaries` by label, `droppedFlagCount`), insert the valid flags into `ai_flags` (with `member_user_id` taken from `label_map`, `null` for unassigned authors, and SHAs instead of refs), set the report `completed`, and call `recomputeSubmissionStatus`.
5. Network errors, timeouts, HTTP 429 and 5xx are retried with backoff (2 s, then 6 s). After the last retry the report is `failed` with `AI_UNAVAILABLE`.
6. Labels are mapped to names only when the Instructor views the report.

## 8. Gemini request settings

| Setting           | Value                                                                                     |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Model             | `GEMINI_MODEL` environment variable                                                       |
| Temperature       | 0.2                                                                                       |
| topP              | 0.9                                                                                       |
| Response format   | JSON only (response MIME type `application/json`)                                         |
| Timeout           | 30 seconds per attempt                                                                    |
| Stored per report | `model`, `prompt_version`, `input_hash` (SHA-256 of the exact request body), token counts |

## 9. Prompt text (version `v1.0`)

The system instruction, fixed in code:

```text
You are an assistant that helps a university instructor review how a student team used Git during one project milestone.

You receive anonymised commit metadata only. You never see source code. People are identified only by labels such as "Member A". Do not guess real names.

Your tasks:
1. Write a short neutral summary of the team's activity in the period.
2. Write one short summary per member label describing their activity pattern.
3. Raise flags only for these three problems: low_information_commit_messages, off_topic_commits, repetitive_trivial_commits.

Rules:
- Base every statement on the data you were given. Do not invent facts.
- Every flag must cite between 1 and 20 commit refs (for example "c12") from the input as evidence. Never cite a ref that is not in the input.
- Commit count, lines changed and activity share are NOT proof of effort or quality. Do not rank people, do not accuse anyone, and do not suggest grades or scores.
- Use a calm, factual tone. Use "severity" low unless the evidence is strong.
- If there is nothing to flag, return an empty flags array.
- Answer with JSON only, exactly matching the schema. No text outside the JSON.
```

The user message is the JSON request body from Section 3, followed by the output schema from Section 6. On the single retry, the zod error message is appended to the user message.