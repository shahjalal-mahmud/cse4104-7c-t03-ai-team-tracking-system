# CourseLens – Documentation Index and Conventions

| Field                | Value                                                                                            |
|----------------------|--------------------------------------------------------------------------------------------------|
| Project              | CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System                   |
| Team                 | CSE4104-7C-T03 (Section 7C)                                                                      |
| Document set version | 1.2-DRAFT (not locked)                                                                           |
| Status               | Draft for team review. After review, the team sets this to **LOCKED** and records the date here. |
| Lock date            | _(fill in when locked)_                                                                          |

---

## 1. Purpose of this document set

This set replaces "assumptions" with decisions. It is written so that a developer, or an AI assistant used through a web chat interface, can build CourseLens by following the documents literally. If something is not written here, the builder must **ask**, not guess (see Section 6).

Each document is self-contained: it repeats the rules it needs and does not point to other documents by rule ID. Because of this, renumbering a rule in one document never breaks another document. Where a long definition lives in one place only (AI input and prompt, UI routes and texts, the build plan), the document points to that file and section by name.

## 2. Document set

| File                                       | Contents                                                              | Use it when                                |
|--------------------------------------------|-----------------------------------------------------------------------|--------------------------------------------|
| `00-index-and-conventions.md`              | This file: scope, phases, conventions, rules for AI                   | Always paste first                         |
| `01-business-rules.md`                     | Every rule the system enforces (BR-xx) and the numeric limits         | Writing validation and service logic       |
| `02-functional-requirements.md`            | What the system does (FR-xx), by module                               | Planning any feature                       |
| `03-non-functional-requirements.md`        | Quality targets (NFR-xx)                                              | Security, performance, UX decisions        |
| `04-roles-and-permissions.md`              | Roles, token claims, guards, permission and visibility matrix         | Writing auth middleware and access checks  |
| `05-use-cases.md`                          | Step-by-step user flows (UC-xx)                                       | Writing controllers, services and UI flows |
| `06-data-model.md`                         | ERD, tables, columns, keys, relations, enumerations                   | Writing migrations and models              |
| `07-api-specification.md`                  | Every endpoint, request, response and error code                      | Writing routes, controllers, API client    |
| `08-architecture-and-project-structure.md` | Stack versions, folder structure, layering, env vars, deployment      | Project setup, any new file                |
| `09-ai-integration-spec.md`                | Exact AI input, commit references, output schema, prompt              | Writing the AI review feature              |
| `10-ui-routes-and-text.md`                 | Frontend routes, fixed UI texts, status labels                        | Writing any frontend page                  |
| `11-build-plan.md`                         | Ordered build steps and what each step contains                       | Starting any new piece of work             |

## 3. Precedence when two documents disagree

Documents should never disagree. If they do, **stop and report the conflict to the human**; do not choose silently. For the human's tie-break, use this order (highest wins):

1. `01-business-rules.md`
2. `06-data-model.md`
3. `07-api-specification.md`
4. `09-ai-integration-spec.md`
5. `02-functional-requirements.md`
6. `04-roles-and-permissions.md`
7. `05-use-cases.md`
8. `10-ui-routes-and-text.md`
9. `03-non-functional-requirements.md`
10. `08-architecture-and-project-structure.md`
11. `11-build-plan.md`

After the tie-break, fix the losing document so the conflict disappears.

## 4. Conventions

### 4.1 Requirement language

- **MUST** / **SHALL**: mandatory. **SHOULD**: expected unless there is a recorded reason. **MAY**: optional.
- Priority tags: **Must** and **Should**.

### 4.2 Identifier schemes

IDs are numbered serially inside their own file.

| Prefix           | Meaning                    | Example    |
|------------------|----------------------------|------------|
| `BR-nn`          | Business rule              | BR-13      |
| `FR-<MODULE>-nn` | Functional requirement     | FR-GH-06   |
| `NFR-<AREA>-nn`  | Non-functional requirement | NFR-SEC-03 |
| `UC-nn`          | Use case                   | UC-08      |

Functional modules: `AUTH`, `TEAM`, `MIL`, `SUB`, `VAL`, `GH`, `AI`, `DASH`, `STU`, `CFG`.

### 4.3 Naming

| Where                                   | Style                            | Example                                                 |
|-----------------------------------------|----------------------------------|---------------------------------------------------------|
| Database tables and columns             | `snake_case`, plural table names | `submissions.submitted_at`                              |
| JSON keys in API requests and responses | `camelCase`                      | `submittedAt`                                           |
| Enumeration values                      | lowercase `snake_case` strings   | `ai_reviewed`                                           |
| Error codes                             | `UPPER_SNAKE_CASE`               | `TEAM_NAME_TAKEN`                                       |
| Backend files                           | `camelCase.js`, suffix by role   | `teamService.js`, `team.routes.js`, `teamController.js` |
| React components                        | `PascalCase.jsx`                 | `SubmissionForm.jsx`                                    |
| Environment variables                   | `UPPER_SNAKE_CASE`               | `GEMINI_API_KEY`                                        |
| Git commits                             | Conventional Commits             | `feat(backend): add team creation endpoint`             |

### 4.4 Date, time and number rules

- All timestamps are stored in UTC (`timestamptz`) and sent in API responses as ISO 8601 UTC strings ending in `Z`.
- The UI displays and the metrics calculate in **Asia/Dhaka (UTC+6, no daylight saving)**. Display format: `DD MMM YYYY, hh:mm A` (for example `09 Nov 2026, 11:59 PM`). Weeks start on Monday.
- IDs are UUID v4 strings.
- Scores are decimals with at most 2 decimal places. Percentages are numbers from 0 to 100 with at most 1 decimal place.
- Numeric limits (file size, cooldowns, commit caps and so on) have one source of truth: Section 9 of `01-business-rules.md`.

### 4.4.1 Team name format

Team names are generated by the server as `{course_code}-{section}-T{NN}` (example `CSE4104-7C-T03`). Users never type the team number.

### 4.5 Language

All user-facing text is English. No internationalisation is built.

## 5. Scope and delivery phases

### 5.1 Phases

| Phase                    | Contains                           |
|--------------------------|------------------------------------|
| **Phase 1 – Core (MVP)** | All requirements tagged **Must**   |
| **Phase 2 – Important**  | All requirements tagged **Should** |

Nothing outside Phase 1 and Phase 2 is built. No code, tables or placeholders are generated for features that are not in the documents. The order of work is in `11-build-plan.md`.

### 5.2 Out of scope (never build)

Plagiarism detection; static code-quality analysis; running or testing student code; Google Classroom or LMS integration; native mobile apps; automatic grading without instructor approval; AI-suggested rubric scores or AI-filled grades; private-repository access; version control systems other than GitHub; automatic late-penalty marks; resubmission or submission versions; team deletion or deactivation; account deactivation; automatic risk flags or risk thresholds; audit logging; notifications (in-app or email); PDF reports; password reset by email; rate limiting of login, registration or general API routes; document file types other than PDF and Word (.docx); multi-course or multi-university tenancy (the schema allows several course offerings, but the UI manages exactly one active offering).

## 6. Rules for AI assistants and developers (no-assumption policy)

1. Use **only** the technologies, versions, names, endpoints, tables, columns, enums, error codes and messages defined in this document set.
2. Never invent an endpoint, column, enum value, environment variable, package, route, error code or API message. For frontend labels and button texts, use `10-ui-routes-and-text.md`; if a label is missing there, choose a short plain English label and list it at the end of the answer. For anything else that is missing, **stop and ask**, quoting the document and section that is silent.
3. If a request conflicts with a document, say which rule conflicts and ask the human to choose.
4. Every business rule is enforced **on the server**. The frontend may repeat checks for convenience but is never the only enforcement.
5. Return **complete files**, not snippets, with no placeholders such as `// ...rest of code` or `TODO` (unless the TODO is listed in the task).
6. Follow the layering in the architecture document: routes → middleware → controllers → services → models. SQL appears only in `models/`.
7. Use the response envelope and error codes in the API specification exactly.
8. Never send student names, email addresses, usernames, source code, file paths, repository URLs or team names to the AI service. The exact input is in `09-ai-integration-spec.md`.
9. Never let AI output change a grade, status or visibility by itself.
10. Build only the step of `11-build-plan.md` that the human names. Do not write code, tables or routes that belong to a later step.
11. State at the end of every answer: files created or changed, requirement and rule IDs covered, and anything that was unclear.

## 7. Glossary

| Term            | Meaning                                                                                                    |
|-----------------|------------------------------------------------------------------------------------------------------------|
| Course offering | One run of the course (CSE4104, Fall 2026). Exactly one is active.                                         |
| Instructor      | System administrator and grader. Database role `instructor`. Has read-only access to teams.                |
| Student         | Database role `student`. Becomes a **Team Leader** when `team_members.is_leader = true`.                   |
| Team Leader     | A contextual role, not a database role: the one student who leads a team. Leadership can be transferred.   |
| Milestone       | Weekly deliverable defined by the instructor, with document requirements and a rubric.                     |
| Submission      | One team's single response to one milestone. Stored only after all validation checks pass. Never edited.   |
| Validation      | The blocking checks run inside the submit request before anything is stored.                               |
| Sync            | One run that fetches commits from GitHub into the database.                                                |
| Period          | The date range analysed for a milestone.                                                                   |
| Counted commit  | A stored commit inside the period that is not excluded as a merge or bot commit.                           |
| AI report       | One Gemini-generated review attached to a submission. Always an advisory draft, visible to Instructors.    |
| AI flag         | A warning in an AI report that cites at least one existing commit as evidence.                             |
| Commit ref      | Short reference such as `c17` that stands for one commit inside one AI request. Mapped back to a SHA.      |
| Evidence        | Commit refs (stored as SHAs) that support an AI flag.                                                      |
| Label           | Anonymous name such as `Member A` used in AI requests.                                                     |
| Student status  | The reduced submission status that students and leaders see (`submitted` or `published`).                  |

## 8. How to lock the documents

1. Read every file and fix errors directly in the files.
2. Check that no two documents disagree (Section 3).
3. Set **Status** at the top of this file to `LOCKED` and add the date.
4. Commit with message `docs: lock documentation set`.