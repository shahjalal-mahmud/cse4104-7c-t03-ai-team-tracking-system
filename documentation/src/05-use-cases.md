# 05 – Use Cases

Every use case follows one template. **Main flow** steps alternate between actor action and system response. **Exceptions** reference the step number where they occur.

## 1. Use case summary

| ID    | Use case                         | Primary actor                   | Pri    | Phase |
| ----- | -------------------------------- | ------------------------------- | ------ | ----- |
| UC-01 | Register account                 | Visitor (becomes Student)       | Must   | 1     |
| UC-02 | Log in                           | Student, Instructor             | Must   | 1     |
| UC-03 | Register team                    | Student                         | Must   | 1     |
| UC-04 | Manage milestone and rubric      | Instructor                      | Must   | 1     |
| UC-05 | Submit milestone                 | Team Leader                     | Must   | 1     |
| UC-06 | Validate submission              | System                          | Must   | 1     |
| UC-07 | Sync GitHub commits              | System, Team Leader, Instructor | Must   | 1     |
| UC-08 | Generate AI review               | Instructor                      | Must   | 1     |
| UC-09 | Review, grade and publish        | Instructor                      | Must   | 1     |
| UC-10 | View team progress and feedback  | Student, Team Leader            | Must   | 1     |
| UC-11 | View teams                       | Instructor                      | Must   | 1     |
| UC-12 | Manage course settings           | Instructor                      | Should | 2     |
| UC-13 | Manage own team                  | Team Leader                     | Must   | 1     |
| UC-14 | Edit profile and change password | Student, Instructor             | Should | 2     |
| UC-15 | Create instructor account        | Instructor                      | Should | 2     |
| UC-16 | Set a new password for a student | Instructor                      | Must   | 1     |

---

## UC-01 Register account

| Field         | Detail                              |
| ------------- | ----------------------------------- |
| Actor         | Visitor                             |
| Trigger       | Visitor opens the registration page |
| Preconditions | Not logged in                       |

**Main flow**
1. Visitor enters full name, student ID, email, GitHub username, password, confirm password and registration code.
2. Client validates the formats and shows inline errors. Rules: student ID is exactly 11 digits; GitHub username is 1–39 characters (letters, digits, hyphens, not starting with a hyphen); email is valid, at most 254 characters; full name is 2–100 characters; password is 8–72 characters with at least one letter and one digit.
3. Client sends the request. System validates again, checks the registration code (when the course has one), checks uniqueness, hashes the password and creates the user with role `student`.
4. System returns a token and the user. Client stores the token and redirects to the student dashboard.

**Exceptions**
- 3a. Validation fails → HTTP 422 `VALIDATION_ERROR` with field details; client shows each message under its field.
- 3b. Duplicate email, student ID or GitHub username → HTTP 409 with the matching code (`EMAIL_TAKEN`, `STUDENT_ID_TAKEN`, `GITHUB_USERNAME_TAKEN`); client highlights that field.
- 3c. Wrong registration code → HTTP 422 `INVALID_REGISTRATION_CODE`.

**Postconditions:** a student user exists and a session is established.

## UC-02 Log in

| Field         | Detail              |
| ------------- | ------------------- |
| Actor         | Student, Instructor |
| Preconditions | Account exists      |

**Main flow**
1. User enters email and password.
2. System finds the user and compares the password with the bcrypt hash.
3. System returns a token and the user. Client stores the token.
4. Client redirects: Instructor → instructor dashboard; Student → student dashboard.

**Exceptions**
- 2a. Wrong email or password → 401 `INVALID_CREDENTIALS` (same message for both).
- Any later request with an expired or invalid token → 401 `UNAUTHENTICATED`; client deletes the token and redirects to the login page.

**Postconditions:** authenticated session; `last_login_at` updated.

## UC-03 Register team

| Field         | Detail                                                                |
| ------------- | --------------------------------------------------------------------- |
| Actor         | Student without a team                                                |
| Preconditions | Logged in as student; not a member of any team in the course offering |

**Main flow**
1. Student opens "Create your team" and enters the section (select from the allowed sections), project title and repository URL (a public `github.com` repository). The team number is not entered.
2. Client submits. System validates the input and the repository URL syntax.
3. System asks GitHub for the repository. It must exist and not be private. System takes the canonical URL, owner and name from the GitHub response and checks that no other team in the offering uses the same repository (case-insensitive).
4. In one transaction, system locks the course offering row, assigns the next team number (highest existing number for this offering and section plus 1; the first team in a section gets `01`; numbers are never reused), builds the name `{course_code}-{section}-T{NN}` (for example `CSE4104-7C-T03`), checks it against the course name pattern and its uniqueness, creates the team with the canonical repository URL, owner and name, and creates a team member row with `is_leader = true` and project role `Team Leader`.
5. System returns the team. Client redirects to the team page, which shows the generated team name.

**Exceptions**
- 2a. Already in a team → 409 `ALREADY_IN_TEAM`.
- 2b. Bad repository URL → 422 `INVALID_REPOSITORY_URL`.
- 2c. Section not allowed → 422 `VALIDATION_ERROR` on field `section`.
- 3a. Repository not found or private → 422 `REPOSITORY_UNREACHABLE`. GitHub rate limited → 422 `GITHUB_RATE_LIMITED`.
- 3b. Repository already used by another team → 409 `REPOSITORY_IN_USE`.
- 4a. The generated name does not match the name pattern → 422 `INVALID_TEAM_NAME`.
- 4b. Name already exists → 409 `TEAM_NAME_TAKEN` (a parallel creation lost the race).
- 4c. The section already has 99 teams → 409 `TEAM_LIMIT_REACHED`.
- Non-student user → 403 `FORBIDDEN`.

**Postconditions:** the team exists with one leader. Its repository URL can never be changed. A mistaken URL can only be corrected by the Instructor through the documented SQL procedure (see the architecture document).

## UC-04 Manage milestone and rubric

| Field | Detail     |
| ----- | ---------- |
| Actor | Instructor |

**Main flow (create and publish)**
1. Instructor opens "New milestone" and fills week number (1–14), title, description, guidelines, deadline, max marks and the "require live URL" toggle.
2. System creates the milestone with status `draft`.
3. Instructor adds document requirements: key, label, filename pattern and required flag. The pattern uses only the placeholder `{TEAM}` and must end with `.pdf` or `.docx`.
4. System saves the full list (replaces the previous list).
5. Instructor adds rubric criteria (title, description, max points).
6. System saves the full list and shows the running total versus max marks.
7. Instructor clicks **Publish**. System checks that there is at least one rubric criterion, that the criteria total equals max marks, that the deadline is in the future and that the deadline order across published milestones is respected; then sets status `published` and `published_at`.

**Alternate flows**
- A1. Edit a milestone that has submissions: only title, description, guidelines and deadline are accepted; any other field → 409 `MILESTONE_LOCKED`.
- A2. Delete: allowed only for a draft milestone or one without submissions; otherwise 409 `MILESTONE_LOCKED`.
- A3. Unpublish: allowed only while the milestone has no submissions.

**Exceptions**
- 2a. Week number already used in the course offering → 409 `MILESTONE_WEEK_TAKEN`.
- 7a. Rubric total differs from max marks → 422 `RUBRIC_TOTAL_MISMATCH`; the message shows both totals.
- 7b. Deadline in the past, no criteria, or deadline order broken → 422 `VALIDATION_ERROR`.

**Postconditions:** the milestone is visible to students once it is published.

## UC-05 Submit milestone

| Field         | Detail                                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Actor         | Team Leader                                                                                                              |
| Preconditions | The milestone is `published`; the team has no submission for this milestone; the team has at least the minimum team size |

**Main flow**
1. Leader opens the milestone page. The form shows only the fields the milestone requires: live URL (when required), one file input per document requirement (with the expected file name and the allowed file type displayed) and notes (max 2,000 characters). The repository is the team's repository and is shown read-only.
2. Leader fills the form and submits.
3. System records `receivedAt` immediately, checks the guards, then runs the full validation (UC-06) **before storing anything**. If any check fails, nothing is stored and the results of all checks are returned.
4. When every check passes, system uploads the files to private storage, then in one transaction creates the submission (with `submitted_at = receivedAt` and the late flag from comparing `receivedAt` with the deadline), stores the file rows and the validation results and sets the status `checked`.
5. System responds HTTP 201 with the submission and starts a GitHub sync in the background (UC-07, trigger `submission`).
6. Client shows the submission page with the validation results.

**Alternate flow**
- A1. (Phase 2) "Check my files": the leader runs the same validation without storing anything and sees the full per-check report.

**Exceptions**
- 3a. Not the leader → 403 `NOT_TEAM_LEADER`. Milestone not published → 409 `MILESTONE_NOT_PUBLISHED`.
- 3b. The team already has a submission for this milestone → 409 `SUBMISSION_EXISTS` (resubmission is not possible).
- 3c. Team smaller than the minimum team size → 409 `TEAM_TOO_SMALL`.
- 3d. Any validation failure → HTTP 422 with the per-check results (see UC-06); nothing is stored.
- 4a. Storage upload fails → 502 `UPSTREAM_ERROR`; nothing stored, uploaded objects deleted.
- 4b. The database transaction fails or a parallel submit won the race (unique constraint) → uploaded objects are deleted; a lost race returns 409 `SUBMISSION_EXISTS`.

**Postconditions:** exactly one submission exists for the team and milestone, with status `checked`. A submission received after the deadline is accepted and marked late; there is no cut-off and no automatic mark penalty.

## UC-06 Validate submission

| Field   | Detail                                                                          |
| ------- | ------------------------------------------------------------------------------- |
| Actor   | System                                                                          |
| Trigger | Every submit request (UC-05) and every dry-run check, before anything is stored |

**Main flow**
1. System loads the milestone and the team.
2. System runs all checks in this order. Every check is blocking:
    - required documents are present and non-empty, and the live URL (when required) is a well-formed `http` or `https` URL;
    - the team name matches the course name pattern;
    - every file name equals its pattern with `{TEAM}` replaced by the team name (case-sensitive);
    - every file extension is `pdf` or `docx`, equals the pattern's extension, and the real content type (detected from the file bytes) matches it;
    - every file is at most 10 MB;
    - the team's repository is reachable on GitHub and public;
    - the live URL (when required): in Phase 1 only its format is checked; in Phase 2 it must also return a final 2xx status within 10 seconds (one retry after 3 seconds on timeout), only `http` and `https` on ports 80 and 443 are accepted, and private, loopback and link-local addresses are refused, including after redirects.
3. System records one result per check (`passed`, `failed`, `skipped` or `error`, with a message) and returns the report.

**Exceptions**
- 2a. Missing document or bad live URL format → 422 `VALIDATION_ERROR` (field-level).
- 2b. File name wrong → 422 `FILENAME_MISMATCH` with expected and received names.
- 2c. Wrong type or content mismatch → 422 `INVALID_FILE_TYPE`. Too large → 422 `FILE_TOO_LARGE`.
- 2d. Repository not reachable or private → 422 `REPOSITORY_UNREACHABLE`. GitHub rate limited → 422 `GITHUB_RATE_LIMITED`.
- 2e. (Phase 2) Live URL not reachable or not allowed → 422 `LIVE_URL_UNREACHABLE` (a refused URL gives the message "URL is not allowed.").

**Postconditions:** if any check fails, the submission is rejected and nothing is stored; the leader fixes the problem or tries again later. If all checks pass, the report is stored with the submission.

## UC-07 Sync GitHub commits

| Field | Detail                                                                      |
| ----- | --------------------------------------------------------------------------- |
| Actor | System (trigger `submission`), Team Leader or Instructor (trigger `manual`) |

**Main flow**
1. Actor requests a sync. System checks that no run is active for the team and, for a manual sync, that the 5-minute cooldown since the last finished sync has passed. A submission-triggered sync ignores the cooldown.
2. System creates a run (`queued`) and returns HTTP 202 with the run id.
3. Background worker sets `running`, lists the branches, lists the commits per branch (only commits since the offering start date, newest 2,000 distinct commits kept), fetches details only for commits not yet stored, stores them in batches and attributes them to members in this order: (1) GitHub author login equals the member's GitHub username; (2) commit author email equals the member's registered email; (3) commit author email is a GitHub noreply address containing the member's GitHub username. Otherwise the commit is "Unassigned". Matching is case-insensitive. Only public repositories on `github.com` are read, with read-only access.
4. System sets the run `completed` with counts, and the UI updates.

**Exceptions**
- 1a. Active run → 409 `SYNC_IN_PROGRESS`. Cooldown → 429 `SYNC_COOLDOWN` with `retryAfterSeconds`.
- 3a. Repository not found or private → run `failed`, code `GITHUB_REPO_NOT_FOUND`.
- 3b. GitHub rate limited → run `failed`, code `GITHUB_RATE_LIMITED`; commits stored so far stay, and a later sync continues with the missing ones.
- 3c. More than 2,000 commits → run `completed` with `truncated = true`.
- 3d. The server restarts while a run is `queued` or `running` → on start the run is set to `failed` with code `INTERRUPTED`.

**Postconditions:** commits and attribution are up to date as of the run's finish time. Stored data is never deleted by a failed run. Attribution is recomputed without calling GitHub when a member is added or removed or a member changes their GitHub username.

## UC-08 Generate AI review

| Field         | Detail                                                           |
| ------------- | ---------------------------------------------------------------- |
| Actor         | Instructor                                                       |
| Preconditions | The submission exists; at least one counted commit in the period |

**Main flow**
1. Instructor clicks **Generate AI review** on the submission review page.
2. System checks the limits, creates a report with status `pending` and returns HTTP 202.
3. Background worker computes the metrics for the analysis period, anonymises and sanitises the data (see `09-ai-integration-spec.md`), numbers the commits `c1`, `c2`, ..., builds the prompt, calls Gemini and validates the JSON. Flags that do not cite at least one ref that exists in the sent input are dropped and counted. The worker maps refs back to SHAs, stores the result and flags, and sets the report `completed`; the submission status becomes `ai_reviewed`.
4. UI polls the report and displays it when completed.

**Exceptions**
- 2a. No counted commits → 422 `NO_COMMIT_DATA`. A pending report exists for the submission → 409 `AI_REVIEW_IN_PROGRESS`. More than 10 review requests by the Instructor in one hour → 429 `RATE_LIMITED`.
- 3a. Gemini unavailable after retries → report `failed`, code `AI_UNAVAILABLE`; UI shows "AI review is unavailable. You can still grade manually."
- 3b. Invalid output twice → report `failed`, code `AI_INVALID_OUTPUT`.

**Postconditions:** a completed AI draft exists, or a failed report with a reason. No grade is touched. At most the newest 400 commits of the period are sent. Earlier reports are kept; each request creates a new report.

## UC-09 Review, grade and publish

| Field         | Detail                                    |
| ------------- | ----------------------------------------- |
| Actor         | Instructor                                |
| Preconditions | Submission exists (AI report is optional) |

**Main flow**
1. Instructor opens the milestone dashboard, which lists all teams with team name, section, leader, submission status (or `not_submitted`), late flag, grade status and last sync time, and selects a team's submission.
2. System shows the validation report, metrics and charts (each with a table alternative), the latest AI report (if any) with evidence links, and the grade form.
3. Instructor verifies evidence by opening commit links on GitHub.
4. Instructor enters or edits each criterion score and comment, and overall feedback, then clicks **Save draft**.
5. Instructor clicks **Approve** (with a confirmation dialog). System checks that every criterion has a score and feedback of at least 10 characters, sets the grade `approved` and the submission status `graded`.
6. Instructor clicks **Publish**. System sets the grade `published` and the submission status `published`. Students now see the grade and feedback.

**Alternate flows**
- A1. Dismiss an AI report → it stays stored, is hidden by default and does not count for the `ai_reviewed` status.
- A2. Unpublish → the grade returns to `approved` and is hidden from students.
- A3. Edit an approved grade → it returns to `draft`.
- A4. (Phase 2) Search the team list by team name and filter by section, submission status and grade status.
- A5. (Phase 2) Export the grades of a milestone as CSV (team name, section, status, late, total score, max marks, grade status).

**Exceptions**
- 4a. Score out of range (0 to the criterion's max points) or more than 2 decimals → 422 `VALIDATION_ERROR`.
- 5a. Missing score or feedback shorter than 10 characters → 422 `GRADE_INCOMPLETE`.
- 4b. Edit of a published grade → 409 `GRADE_PUBLISHED`.
- Any other invalid transition → 409 `INVALID_GRADE_TRANSITION`.
- A non-Instructor tries to grade → 403 `FORBIDDEN`.

**Postconditions:** grade state updated. The total score is the sum of the criterion scores. AI output never creates, changes, approves or publishes a grade and never fills a score.

## UC-10 View team progress and feedback

| Field         | Detail               |
| ------------- | -------------------- |
| Actor         | Student, Team Leader |
| Preconditions | Member of a team     |

**Main flow**
1. Member opens the student dashboard; system shows the team summary and the list of published milestones with the student status (`submitted` or `published`), deadlines (Asia/Dhaka) and a countdown or "Deadline passed".
2. Member opens a milestone; system shows the guidelines, the submission status, the validation report of the submission (late flag, files, notes), and the grade and feedback only if the grade is published.
3. Member opens "Contribution"; system shows their own metrics and the team's charts for all time or a selected milestone period, with a disclaimer that commit metadata is not a definitive measure of individual effort or quality.
4. Member downloads a submitted file; system returns a signed URL valid for 60 seconds.
5. Member opens the team details and sees the member list (names, student IDs, GitHub usernames, project roles).

**Exceptions**
- No team → empty state with a "Create your team" button (UC-03).
- Another team's id in the URL → 403 `NOT_TEAM_MEMBER`.
- No sync yet → empty state "No commit data yet." with **Sync now** (Leader only).

**Postconditions:** none. Draft and approved-but-unpublished grades and all AI content are never shown to students.

## UC-11 View teams

| Field | Detail     |
| ----- | ---------- |
| Actor | Instructor |

**Main flow**
1. Instructor opens the team list (search by team name, filter by section).
2. Instructor opens a team; system shows the members, repository, sync status, all submissions and all-time contribution charts.
3. Instructor can click **Sync now** to start a manual sync (UC-07).

**Exceptions:** manual sync cooldown or active run → see UC-07.

**Postconditions:** none. The Instructor has read access to all teams but cannot edit a team, add or remove members, change the leader or change the repository URL. Teams cannot be deleted or deactivated.

## UC-12 Manage course settings

| Field | Detail     |
| ----- | ---------- |
| Actor | Instructor |

**Main flow:** Instructor opens the settings, edits any of: registration code, allowed sections, team name pattern, minimum team size, maximum team size, exclude merge commits, exclude bot commits; and saves. System validates and saves. Defaults for the team sizes are 2 and 6.

**Exceptions:** invalid pattern (not a valid regular expression), or team sizes that do not satisfy `1 <= minimum <= maximum <= 10` → 422 `VALIDATION_ERROR`.

**Postconditions:** changes apply to later actions only. Existing teams keep all their members even if they now exceed the maximum or fall below the minimum; the new limits apply to later add-member and submit actions.

## UC-13 Manage own team

| Field         | Detail                            |
| ------------- | --------------------------------- |
| Actor         | Team Leader                       |
| Preconditions | Logged in as the leader of a team |

**Main flow**
1. Leader opens the team page and chooses **Add member**; enters a student ID; the system looks it up and shows the name and whether the student is free.
2. Leader enters the member's project role (free text, 2–60 characters) and confirms. System adds the member immediately (no acceptance step) and re-attributes commits.
3. Leader can remove a non-leader member; system removes them and re-attributes commits.
4. Leader can transfer leadership to any current member. System makes that member the leader and the previous leader a regular member.
5. Leader can edit the project title at any time.

**Exceptions**
- 1a. Unknown student ID → 404 `STUDENT_NOT_FOUND`.
- 2a. Student already in a team in the course offering → 409 `ALREADY_IN_TEAM`.
- 2b. The team already has the maximum team size → 409 `TEAM_FULL`.
- 3a. Leader tries to remove themself → 409 `CANNOT_REMOVE_LEADER`.
- 4a. The new leader is not a member of the team → 422 `VALIDATION_ERROR` on field `userId`.
- Any action by a non-leader → 403 `NOT_TEAM_LEADER`.
- Any attempt to change the repository URL → 409 `REPOSITORY_LOCKED`; the URL is permanent.

**Postconditions:** the team always has exactly one leader.

## UC-14 Edit profile and change password

**Main flow:** the user opens Profile; edits the full name or GitHub username (students), or enters the current and a new password; the system validates the same field rules as registration and saves. A changed GitHub username triggers re-attribution of the team's commits.

**Exceptions:** wrong current password → 422 `VALIDATION_ERROR` (field `currentPassword`); username taken → 409 `GITHUB_USERNAME_TAKEN`.

## UC-15 Create instructor account

**Main flow:** an Instructor opens "Instructors", enters full name, email and password; the system validates the fields (student ID and GitHub username are not required), and creates a user with role `instructor`. The first Instructor account is created by the seed script; instructors cannot be created by public registration.

**Exceptions:** duplicate email → 409 `EMAIL_TAKEN`; validation failure → 422 `VALIDATION_ERROR`.

## UC-16 Set a new password for a student

| Field   | Detail                                                   |
| ------- | -------------------------------------------------------- |
| Actor   | Instructor                                               |
| Trigger | A student cannot log in because they forgot the password |

**Main flow**
1. Instructor opens "Students", enters the student's 11-digit student ID and a new password.
2. System checks that the student exists and that the new password follows the password rule, then stores the new bcrypt hash.
3. Instructor tells the student the new password outside the system. The student logs in and can change it (UC-14).

**Exceptions**
- 2a. Unknown student ID → 404 `STUDENT_NOT_FOUND`.
- 2b. Password rule not met → 422 `VALIDATION_ERROR`.
- Non-Instructor → 403 `FORBIDDEN`.

**Postconditions:** the student's password hash is replaced. Instructor passwords cannot be reset this way.