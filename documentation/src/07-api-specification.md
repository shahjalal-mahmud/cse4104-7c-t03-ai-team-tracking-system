# 07 – API Specification

## 1. General conventions

### 1.1 Base path and format

- All paths below are relative to the base path `/api` (for example `POST /api/auth/login`).
- Requests and responses are JSON (`Content-Type: application/json`), except submission creation and dry-run check (`multipart/form-data`) and the CSV export (`text/csv`).
- JSON keys are `camelCase`. Timestamps are ISO 8601 UTC strings ending in `Z`. IDs are UUID v4 strings.
- JSON body limit: 100 KB. File upload limit: 10 MB per file.
- Unknown fields in a request body are rejected with `VALIDATION_ERROR`.

### 1.2 Authentication

- `POST /auth/register`, `POST /auth/login` and `GET /health` are public. Every other route requires `Authorization: Bearer <token>`.
- Token: JWT HS256, expires after 8 hours. Claims: `sub` (user id), `role`, `iat`, `exp`.
- The **Auth** line of each endpoint lists who may call it:
    - **Any user**: any logged-in user.
    - **Instructor**: role `instructor`.
    - **Student**: role `student`.
    - **Member**: a student who belongs to the team, or an Instructor (read access only).
    - **Leader**: the student who is the leader of the team. Instructors never pass.
    - **Leader or Instructor**: the team's leader, or any Instructor (used only for manual sync).

### 1.3 Response envelope

Success:

```json
{ "success": true, "data": { }, "meta": { } }
```

`meta` appears only on paginated lists. Failure:

```json
{
  "success": false,
  "error": { "code": "TEAM_NAME_TAKEN", "message": "Team CSE4104-7C-T03 already exists.", "details": { } }
}
```

`details` is optional. For `VALIDATION_ERROR` it is `{ "fields": { "email": "Email is not valid." } }`. Every endpoint returns this envelope. Deletes return `data: null`.

### 1.4 Pagination

List endpoints accept `page` (default 1) and `pageSize` (default 20, maximum 100) and return:

```json
"meta": { "page": 1, "pageSize": 20, "total": 57, "totalPages": 3 }
```

### 1.5 HTTP status codes

| Status | Use                                                    |
| ------ | ------------------------------------------------------ |
| 200    | Success                                                |
| 201    | Resource created                                       |
| 202    | Background job accepted (sync, AI review)              |
| 401    | Missing, invalid or expired token; wrong credentials   |
| 403    | Authenticated but not allowed                          |
| 404    | Resource not found                                     |
| 409    | State conflict (duplicate, locked, already exists)     |
| 413    | JSON request body too large                            |
| 422    | Validation failure                                     |
| 429    | Cooldown or AI hourly limit                            |
| 500    | Unexpected server error (`INTERNAL_ERROR`, no details) |
| 502    | Storage or other upstream failure                      |

### 1.6 Phase column

In the endpoint summary, **Ph** is the delivery phase (1 = Must, 2 = Should). Phase 2 endpoints and Phase 2 parts of an endpoint are not built until Phase 1 is complete.

## 2. Error codes

| Code                        | HTTP | Meaning                                                                                                                                                                 |
| --------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `UNAUTHENTICATED`           | 401  | Missing, invalid or expired token                                                                                                                                       |
| `INVALID_CREDENTIALS`       | 401  | Wrong email or password                                                                                                                                                 |
| `FORBIDDEN`                 | 403  | Role not allowed for this action                                                                                                                                        |
| `NOT_TEAM_MEMBER`           | 403  | Student is not a member of the team                                                                                                                                     |
| `NOT_TEAM_LEADER`           | 403  | Action needs the Team Leader                                                                                                                                            |
| `NOT_FOUND`                 | 404  | Resource does not exist                                                                                                                                                 |
| `STUDENT_NOT_FOUND`         | 404  | Unknown student ID                                                                                                                                                      |
| `VALIDATION_ERROR`          | 422  | Invalid or missing input (including a section that is not allowed and a leadership target who is not a member); also the generic failed-validation code of a submission |
| `EMAIL_TAKEN`               | 409  | Email already registered                                                                                                                                                |
| `STUDENT_ID_TAKEN`          | 409  | Student ID already registered                                                                                                                                           |
| `GITHUB_USERNAME_TAKEN`     | 409  | GitHub username already registered                                                                                                                                      |
| `INVALID_REGISTRATION_CODE` | 422  | Wrong or missing registration code                                                                                                                                      |
| `INVALID_TEAM_NAME`         | 422  | The generated team name does not match the name pattern                                                                                                                 |
| `TEAM_NAME_TAKEN`           | 409  | Team name already exists in the offering                                                                                                                                |
| `TEAM_LIMIT_REACHED`        | 409  | The section already has 99 teams                                                                                                                                        |
| `ALREADY_IN_TEAM`           | 409  | Student already belongs to a team in the offering                                                                                                                       |
| `TEAM_FULL`                 | 409  | Team already has the maximum team size                                                                                                                                  |
| `TEAM_TOO_SMALL`            | 409  | Team has fewer members than the minimum team size                                                                                                                       |
| `CANNOT_REMOVE_LEADER`      | 409  | The leader cannot be removed                                                                                                                                            |
| `INVALID_REPOSITORY_URL`    | 422  | Not a valid `github.com` repository URL                                                                                                                                 |
| `REPOSITORY_UNREACHABLE`    | 422  | Repository not found, not reachable or not public (team creation and submission validation)                                                                             |
| `REPOSITORY_IN_USE`         | 409  | Another team in the offering already uses this repository                                                                                                               |
| `REPOSITORY_LOCKED`         | 409  | The repository URL can never be changed                                                                                                                                 |
| `MILESTONE_NOT_PUBLISHED`   | 409  | Milestone is not published                                                                                                                                              |
| `MILESTONE_LOCKED`          | 409  | Milestone has submissions; the change is not allowed                                                                                                                    |
| `MILESTONE_WEEK_TAKEN`      | 409  | Week number already used in the offering                                                                                                                                |
| `RUBRIC_TOTAL_MISMATCH`     | 422  | Sum of criterion points differs from max marks                                                                                                                          |
| `SUBMISSION_EXISTS`         | 409  | The team already submitted this milestone (also returned when a parallel submit loses the race)                                                                         |
| `FILENAME_MISMATCH`         | 422  | File name differs from the expected name                                                                                                                                |
| `INVALID_FILE_TYPE`         | 422  | Extension not `pdf`/`docx`, not equal to the pattern's extension, or real content type does not match                                                                   |
| `FILE_TOO_LARGE`            | 422  | File is larger than 10 MB (also returned by the upload middleware for an oversized file)                                                                                |
| `LIVE_URL_UNREACHABLE`      | 422  | Phase 2: live URL not reachable or not allowed                                                                                                                          |
| `GITHUB_RATE_LIMITED`       | 422  | GitHub rate limit hit during team creation or validation (stored on a sync run when it happens in a sync)                                                               |
| `GITHUB_REPO_NOT_FOUND`     | –    | Stored on a sync run: repository missing or private                                                                                                                     |
| `SYNC_COOLDOWN`             | 429  | Last sync finished less than 5 minutes ago                                                                                                                              |
| `SYNC_IN_PROGRESS`          | 409  | A sync is already queued or running for the team                                                                                                                        |
| `NO_COMMIT_DATA`            | 422  | No counted commit in the period                                                                                                                                         |
| `AI_REVIEW_IN_PROGRESS`     | 409  | A review is already pending for the submission                                                                                                                          |
| `RATE_LIMITED`              | 429  | More than 10 AI review requests by the Instructor in one hour                                                                                                           |
| `AI_UNAVAILABLE`            | –    | Stored on an AI report: Gemini failed after retries                                                                                                                     |
| `AI_INVALID_OUTPUT`         | –    | Stored on an AI report: output invalid twice                                                                                                                            |
| `INTERRUPTED`               | –    | Stored on a sync run or AI report that was unfinished when the server started                                                                                           |
| `GRADE_INCOMPLETE`          | 422  | A criterion lacks a score or feedback of at least 10 characters                                                                                                         |
| `GRADE_PUBLISHED`           | 409  | A published grade cannot be edited                                                                                                                                      |
| `INVALID_GRADE_TRANSITION`  | 409  | Transition not allowed from the current grade status                                                                                                                    |
| `PAYLOAD_TOO_LARGE`         | 413  | JSON body larger than 100 KB                                                                                                                                            |
| `UPSTREAM_ERROR`            | 502  | Storage upload or another upstream call failed                                                                                                                          |
| `INTERNAL_ERROR`            | 500  | Unexpected error                                                                                                                                                        |

## 3. Endpoint summary

| Method | Path                                                       | Auth                 | Ph  | Purpose                                                                 |
| ------ | ---------------------------------------------------------- | -------------------- | --- | ----------------------------------------------------------------------- |
| GET    | `/health`                                                  | Public               | 1   | Health check                                                            |
| POST   | `/auth/register`                                           | Public               | 1   | Register a student                                                      |
| POST   | `/auth/login`                                              | Public               | 1   | Log in                                                                  |
| GET    | `/auth/me`                                                 | Any user             | 1   | Current user                                                            |
| POST   | `/auth/change-password`                                    | Any user             | 2   | Change own password                                                     |
| PATCH  | `/users/me`                                                | Any user             | 2   | Edit own profile                                                        |
| POST   | `/instructors`                                             | Instructor           | 2   | Create an Instructor                                                    |
| POST   | `/students/:studentId/reset-password`                      | Instructor           | 1   | Set a new password for a student                                        |
| GET    | `/course`                                                  | Any user             | 1   | Read course settings                                                    |
| PATCH  | `/course`                                                  | Instructor           | 2   | Edit course settings                                                    |
| POST   | `/teams`                                                   | Student              | 1   | Create a team                                                           |
| GET    | `/teams`                                                   | Instructor           | 1   | List teams                                                              |
| GET    | `/teams/me`                                                | Student              | 1   | Own team (or null)                                                      |
| GET    | `/teams/:teamId`                                           | Member               | 1   | Team details                                                            |
| PATCH  | `/teams/:teamId`                                           | Leader               | 1   | Edit project title                                                      |
| POST   | `/teams/:teamId/members`                                   | Leader               | 1   | Add a member                                                            |
| DELETE | `/teams/:teamId/members/:userId`                           | Leader               | 1   | Remove a non-leader member                                              |
| POST   | `/teams/:teamId/leader`                                    | Leader               | 1   | Transfer leadership                                                     |
| GET    | `/teams/:teamId/students/lookup`                           | Leader               | 1   | Look up a student by ID                                                 |
| POST   | `/milestones`                                              | Instructor           | 1   | Create a milestone                                                      |
| GET    | `/milestones`                                              | Any user             | 1   | List milestones                                                         |
| GET    | `/milestones/:milestoneId`                                 | Any user             | 1   | Milestone detail                                                        |
| PATCH  | `/milestones/:milestoneId`                                 | Instructor           | 1   | Edit a milestone                                                        |
| DELETE | `/milestones/:milestoneId`                                 | Instructor           | 1   | Delete a milestone                                                      |
| PUT    | `/milestones/:milestoneId/documents`                       | Instructor           | 1   | Replace document requirements                                           |
| PUT    | `/milestones/:milestoneId/rubric`                          | Instructor           | 1   | Replace rubric criteria                                                 |
| POST   | `/milestones/:milestoneId/publish`                         | Instructor           | 1   | Publish                                                                 |
| POST   | `/milestones/:milestoneId/unpublish`                       | Instructor           | 1   | Unpublish                                                               |
| GET    | `/milestones/:milestoneId/dashboard`                       | Instructor           | 1   | Teams and their status for a milestone (search and filters are Phase 2) |
| GET    | `/milestones/:milestoneId/grades/export`                   | Instructor           | 2   | Grades as CSV                                                           |
| POST   | `/teams/:teamId/milestones/:milestoneId/submissions`       | Leader               | 1   | Submit a milestone                                                      |
| POST   | `/teams/:teamId/milestones/:milestoneId/submissions/check` | Leader               | 2   | Dry-run check, stores nothing                                           |
| GET    | `/teams/:teamId/submissions`                               | Member               | 1   | List a team's submissions                                               |
| GET    | `/submissions/:submissionId`                               | Member               | 1   | Submission detail                                                       |
| GET    | `/submissions/:submissionId/files/:fileId/download`        | Member               | 1   | Signed download URL                                                     |
| POST   | `/teams/:teamId/github/sync`                               | Leader or Instructor | 1   | Start a manual sync                                                     |
| GET    | `/teams/:teamId/github/sync-runs/latest`                   | Member               | 1   | Latest sync run                                                         |
| GET    | `/teams/:teamId/commits`                                   | Member               | 1   | Stored commits                                                          |
| GET    | `/teams/:teamId/metrics`                                   | Member               | 1   | Contribution metrics                                                    |
| POST   | `/submissions/:submissionId/ai-reviews`                    | Instructor           | 1   | Request an AI review                                                    |
| GET    | `/submissions/:submissionId/ai-reviews`                    | Instructor           | 1   | List AI reviews                                                         |
| GET    | `/ai-reviews/:reviewId`                                    | Instructor           | 1   | One AI review                                                           |
| POST   | `/ai-reviews/:reviewId/dismiss`                            | Instructor           | 1   | Dismiss an AI review                                                    |
| GET    | `/submissions/:submissionId/grade`                         | Member               | 1   | Read the grade                                                          |
| PUT    | `/submissions/:submissionId/grade`                         | Instructor           | 1   | Save a draft grade                                                      |
| POST   | `/submissions/:submissionId/grade/approve`                 | Instructor           | 1   | Approve the grade                                                       |
| POST   | `/submissions/:submissionId/grade/publish`                 | Instructor           | 1   | Publish the grade                                                       |
| POST   | `/submissions/:submissionId/grade/unpublish`               | Instructor           | 1   | Unpublish the grade                                                     |

## 4. Shared objects

**User**

```json
{
  "id": "uuid", "fullName": "Rafi Ahmed", "email": "rafi@example.com", "role": "student",
  "studentId": "20230104001", "githubUsername": "rafi-dev",
  "lastLoginAt": "2026-11-09T05:00:00Z", "createdAt": "2026-09-02T08:00:00Z"
}
```

For an Instructor `studentId` and `githubUsername` are `null`. `email` is returned only to the user themself and to Instructors.

**Team**

```json
{
  "id": "uuid", "name": "CSE4104-7C-T03", "section": "7C", "teamNumber": 3,
  "projectTitle": "CourseLens", "repositoryUrl": "https://github.com/org/repo",
  "repoOwner": "org", "repoName": "repo", "createdAt": "2026-09-10T08:00:00Z",
  "memberCount": 3,
  "members": [
    { "userId": "uuid", "fullName": "Rafi Ahmed", "studentId": "20230104001",
      "githubUsername": "rafi-dev", "projectRole": "Team Leader", "isLeader": true }
  ]
}
```

Members include `email` only for Instructors.

**Milestone**

```json
{
  "id": "uuid", "weekNumber": 3, "title": "Proposal", "description": "…", "guidelines": "…",
  "deadline": "2026-10-01T17:59:00Z", "maxMarks": 20,
  "requireLiveUrl": false, "status": "published", "publishedAt": "2026-09-20T05:00:00Z",
  "hasSubmissions": false,
  "documents": [
    { "id": "uuid", "key": "proposal", "label": "Project proposal",
      "filenamePattern": "{TEAM}_Proposal.pdf", "isRequired": true, "sortOrder": 0 }
  ],
  "rubric": [
    { "id": "uuid", "title": "Problem statement", "description": "…", "maxPoints": 10, "sortOrder": 0 }
  ],
  "createdAt": "2026-09-15T08:00:00Z"
}
```

`hasSubmissions` is returned only to Instructors. In list responses `documents` and `rubric` are omitted.

**Submission**

```json
{
  "id": "uuid", "teamId": "uuid", "milestoneId": "uuid", "status": "checked",
  "liveUrl": null, "notes": "…",
  "submittedBy": { "id": "uuid", "fullName": "Rafi Ahmed" },
  "submittedAt": "2026-10-01T15:00:00Z", "isLate": false,
  "files": [
    { "id": "uuid", "requirementId": "uuid", "requirementKey": "proposal",
      "originalFilename": "CSE4104-7C-T03_Proposal.pdf", "sizeBytes": 183422, "mimeType": "application/pdf" }
  ],
  "validation": {
    "checks": [
      { "checkType": "required_fields", "status": "passed", "message": "All required fields are present.", "details": {} },
      { "checkType": "live_url", "status": "skipped", "message": "Live URL not required.", "details": {} }
    ]
  },
  "aiReviews": [ { "id": "uuid", "status": "completed", "createdAt": "…", "dismissed": false } ]
}
```

**Status values.** For an Instructor, `status` is the stored value: `checked`, `ai_reviewed`, `graded` or `published`. For a student or Team Leader, `status` is the **student status**: `submitted` (stored `checked`, `ai_reviewed` or `graded`) or `published` (stored `published`). `aiReviews` is returned only to Instructors and is omitted for students. Storage paths are never returned. The repository is the team's repository and is not part of the submission.

**Grade**

```json
{
  "id": "uuid", "submissionId": "uuid", "status": "draft", "totalScore": 17.5, "maxMarks": 20,
  "feedback": "Good work overall.",
  "scores": [
    { "criterionId": "uuid", "title": "Problem statement", "maxPoints": 10,
      "score": 8.5, "comment": "Clear and focused." }
  ],
  "approvedAt": null, "publishedAt": null, "updatedAt": "…"
}
```

For students only a `published` grade is returned.

**Sync run**

```json
{
  "id": "uuid", "teamId": "uuid", "trigger": "manual", "status": "completed",
  "repoFullName": "org/repo", "startedAt": "…", "finishedAt": "…",
  "commitsFound": 120, "commitsNew": 14, "truncated": false,
  "errorCode": null, "errorMessage": null, "createdAt": "…"
}
```

**Commit**

```json
{
  "id": "uuid", "sha": "40 hex characters", "authorLogin": "rafi-dev", "authorName": "Rafi Ahmed",
  "authoredAt": "…", "message": "feat: add login", "additions": 40, "deletions": 2, "filesChanged": 3,
  "isMerge": false, "isBot": false, "counted": true, "branches": ["main"],
  "htmlUrl": "https://github.com/org/repo/commit/…",
  "member": { "userId": "uuid", "fullName": "Rafi Ahmed" }
}
```

`member` is `null` for "Unassigned" commits. `authorEmail` is added for Instructors only.

## 5. Endpoints

### 5.1 Health and authentication

#### `GET /health`
Auth: Public. Response 200: `{ "success": true, "data": { "status": "ok" } }`.

#### `POST /auth/register`
Auth: Public. Creates a user with role `student` and logs them in.

Body:

| Field              | Type   | Rules                                                                      |
| ------------------ | ------ | -------------------------------------------------------------------------- |
| `fullName`         | string | 2–100 characters                                                           |
| `studentId`        | string | Exactly 11 digits                                                          |
| `email`            | string | Valid, at most 254 characters                                              |
| `githubUsername`   | string | 1–39 characters, letters, digits, hyphens, not starting with a hyphen      |
| `password`         | string | 8–72 characters, at least one letter and one digit                         |
| `registrationCode` | string | Required when the offering has a registration code (exact, case-sensitive) |

Response 201: `{ "token": "…", "user": User }`.
Errors: `VALIDATION_ERROR`, `INVALID_REGISTRATION_CODE`, `EMAIL_TAKEN`, `STUDENT_ID_TAKEN`, `GITHUB_USERNAME_TAKEN`.

#### `POST /auth/login`
Auth: Public. Body: `{ "email", "password" }`. Updates `lastLoginAt`.
Response 200: `{ "token": "…", "user": User }`.
Errors: `INVALID_CREDENTIALS` (same message for unknown email and wrong password), `VALIDATION_ERROR`.

#### `GET /auth/me`
Auth: Any user. Response 200: `User`. Error: `UNAUTHENTICATED`.

#### `POST /auth/change-password` (Phase 2)
Auth: Any user. Body: `{ "currentPassword", "newPassword" }` (new password follows the password rule).
Response 200: `data: null`.
Errors: `VALIDATION_ERROR` (wrong current password is reported on field `currentPassword`).

#### `PATCH /users/me` (Phase 2)
Auth: Any user. Body (at least one field): `fullName`, `githubUsername` (students only). A changed `githubUsername` triggers re-attribution of the team's commits (no GitHub call).
Response 200: `User`. Errors: `VALIDATION_ERROR`, `GITHUB_USERNAME_TAKEN`, `FORBIDDEN` (Instructor sending `githubUsername`).

#### `POST /instructors` (Phase 2)
Auth: Instructor. Body: `{ "fullName", "email", "password" }`. Response 201: `User` with role `instructor`.
Errors: `VALIDATION_ERROR`, `EMAIL_TAKEN`.

#### `POST /students/:studentId/reset-password`
Auth: Instructor. `:studentId` is the 11-digit student ID. Body: `{ "newPassword" }` (follows the password rule). Replaces the student's password hash. Works only for student accounts.
Response 200: `data: null`. Errors: `STUDENT_NOT_FOUND`, `VALIDATION_ERROR`, `FORBIDDEN`.

### 5.2 Course settings

#### `GET /course`
Auth: Any user. Response 200:

```json
{
  "id": "uuid", "courseCode": "CSE4104", "title": "Software Development III",
  "semesterLabel": "Fall 2026", "startDate": "2026-09-01", "endDate": "2027-01-31",
  "allowedSections": ["7A", "7B", "7C"], "teamNamePattern": "^CSE4104-[0-9][A-Z]-T[0-9]{2}$",
  "minTeamSize": 2, "maxTeamSize": 6, "excludeMergeCommits": true, "excludeBotCommits": true,
  "registrationCode": "abc123"
}
```

`registrationCode` is returned only to Instructors.

#### `PATCH /course` (Phase 2)
Auth: Instructor. Body (at least one field): `registrationCode` (string or `null`), `allowedSections` (non-empty array of strings), `teamNamePattern` (valid regular expression), `minTeamSize`, `maxTeamSize` (`1 <= minTeamSize <= maxTeamSize <= 10`), `excludeMergeCommits`, `excludeBotCommits`.
Response 200: updated course. Errors: `VALIDATION_ERROR`.
Changing team sizes never removes members from existing teams.

### 5.3 Teams

#### `POST /teams`
Auth: Student. Body: `{ "section", "projectTitle", "repositoryUrl" }`.

| Field           | Rules                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------ |
| `section`       | One of the offering's allowed sections (otherwise `VALIDATION_ERROR` on field `section`)         |
| `projectTitle`  | 3–150 characters                                                                                 |
| `repositoryUrl` | `github.com` URL matching `^https://github\.com/([A-Za-z0-9_.-]+)/([A-Za-z0-9_.-]+?)(\.git)?/?$` |

Processing order: validate the input; ask GitHub for the repository (must exist and not be private); check that no other team in the offering uses it; then, in one transaction, lock the offering row, assign the team number, build the name, check it against the name pattern, and create the team (canonical repository URL, owner and name from GitHub) and the leader's membership with project role `Team Leader`.
Response 201: `Team`.
Errors: `VALIDATION_ERROR`, `INVALID_REPOSITORY_URL`, `REPOSITORY_UNREACHABLE`, `REPOSITORY_IN_USE`, `GITHUB_RATE_LIMITED`, `ALREADY_IN_TEAM`, `INVALID_TEAM_NAME`, `TEAM_NAME_TAKEN`, `TEAM_LIMIT_REACHED`, `FORBIDDEN`.

#### `GET /teams`
Auth: Instructor. Query: `search` (team name), `section`, `page`, `pageSize`. Response 200: list of `Team` (without `members`, with `memberCount` and `leader: { userId, fullName }`) with `meta`.

#### `GET /teams/me`
Auth: Student. Response 200: `Team` plus `myMembership: { "isLeader": true, "projectRole": "Team Leader" }`, or `data: null` when the student has no team.

#### `GET /teams/:teamId`
Auth: Member. Response 200: `Team`. Errors: `NOT_FOUND`, `NOT_TEAM_MEMBER`.

#### `PATCH /teams/:teamId`
Auth: Leader. Body: `{ "projectTitle" }`. Response 200: `Team`.
A body containing `repositoryUrl` (or any other field) is rejected: `REPOSITORY_LOCKED` for `repositoryUrl`, `VALIDATION_ERROR` otherwise.
Errors: `NOT_TEAM_LEADER`, `REPOSITORY_LOCKED`, `VALIDATION_ERROR`.

#### `POST /teams/:teamId/members`
Auth: Leader. Body: `{ "studentId", "projectRole" }` (`projectRole` 2–60 characters). The member is added immediately and commit attribution is recomputed.
Response 201: `Team`.
Errors: `NOT_TEAM_LEADER`, `STUDENT_NOT_FOUND`, `ALREADY_IN_TEAM`, `TEAM_FULL`, `VALIDATION_ERROR`.

#### `DELETE /teams/:teamId/members/:userId`
Auth: Leader. Removes a non-leader member and recomputes commit attribution. Response 200: `data: null`.
Errors: `NOT_TEAM_LEADER`, `CANNOT_REMOVE_LEADER`, `NOT_FOUND`.

#### `POST /teams/:teamId/leader`
Auth: Leader. Body: `{ "userId" }` (an existing member). The previous leader becomes a regular member; exactly one leader exists afterwards.
Response 200: `Team`. Errors: `NOT_TEAM_LEADER`, `VALIDATION_ERROR` (field `userId`: the target is not a member of the team).

#### `GET /teams/:teamId/students/lookup`
Auth: Leader. Query: `studentId` (11 digits). Response 200: `{ "id", "fullName", "studentId", "hasTeam" }`.
Errors: `STUDENT_NOT_FOUND`, `NOT_TEAM_LEADER`, `VALIDATION_ERROR`.

### 5.4 Milestones

#### `POST /milestones`
Auth: Instructor. Body: `weekNumber` (1–14), `title` (3–150), `description` (≤ 5,000, optional), `guidelines` (≤ 10,000, optional), `deadline` (ISO UTC), `maxMarks` (> 0, 2 decimals), `requireLiveUrl` (default false).
Response 201: `Milestone` with status `draft`. Errors: `VALIDATION_ERROR`, `MILESTONE_WEEK_TAKEN`.

#### `GET /milestones`
Auth: Any user. Instructors get all milestones; students get published milestones only. Ordered by week number. For a student with a team each item also has `mySubmission: { "id", "status", "isLate" }` or `null`, where `status` is the student status.
Response 200: list (no pagination, at most 14 items).

#### `GET /milestones/:milestoneId`
Auth: Any user. Response 200: `Milestone` with `documents` and `rubric`. A student requesting a draft milestone gets `NOT_FOUND`.

#### `PATCH /milestones/:milestoneId`
Auth: Instructor. Body: any of the fields of create. Once any submission exists only `title`, `description`, `guidelines` and `deadline` may be changed; other fields give `MILESTONE_LOCKED`. When the deadline of a published milestone is changed, the deadline order across published milestones must still hold (`VALIDATION_ERROR`).
Response 200: `Milestone`. Errors: `VALIDATION_ERROR`, `MILESTONE_LOCKED`, `MILESTONE_WEEK_TAKEN`, `NOT_FOUND`.

#### `DELETE /milestones/:milestoneId`
Auth: Instructor. Allowed only if the milestone is `draft` or has no submissions. Cascades to its documents and rubric.
Response 200: `data: null`. Errors: `MILESTONE_LOCKED`, `NOT_FOUND`.

#### `PUT /milestones/:milestoneId/documents`
Auth: Instructor. Replaces the whole list. Body: `{ "documents": [ { "key", "label", "filenamePattern", "isRequired" } ] }`.

| Field             | Rules                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `key`             | Lowercase letter first, then lowercase letters, digits or `_`, 2–30 characters, unique in the list                           |
| `label`           | 2–80 characters                                                                                                              |
| `filenamePattern` | 3–120 characters of letters, digits, `_`, `{`, `}`, `.`, `-`; only the placeholder `{TEAM}`; must end with `.pdf` or `.docx` |
| `isRequired`      | Boolean, default true                                                                                                        |

Response 200: `Milestone`. Errors: `VALIDATION_ERROR`, `MILESTONE_LOCKED`.

#### `PUT /milestones/:milestoneId/rubric`
Auth: Instructor. Replaces the whole list. Body: `{ "criteria": [ { "title", "description", "maxPoints" } ] }` (`title` 2–100, `description` ≤ 1,000, `maxPoints` > 0 with 2 decimals).
Response 200: `Milestone` plus `rubricTotal` (sum of points). Errors: `VALIDATION_ERROR`, `MILESTONE_LOCKED`.

#### `POST /milestones/:milestoneId/publish`
Auth: Instructor. No body. Requires at least one rubric criterion, a criteria total equal to `maxMarks`, a deadline in the future, and a deadline later than every published milestone with a lower week number and earlier than every published milestone with a higher week number.
Response 200: `Milestone`. Errors: `RUBRIC_TOTAL_MISMATCH` (details: `{ "rubricTotal", "maxMarks" }`), `VALIDATION_ERROR` (no criteria, deadline in the past, or deadline order broken).

#### `POST /milestones/:milestoneId/unpublish`
Auth: Instructor. Allowed only while the milestone has no submissions. Response 200: `Milestone`. Error: `MILESTONE_LOCKED`.

#### `GET /milestones/:milestoneId/dashboard`
Auth: Instructor. Lists all teams for the milestone. Query: `page`, `pageSize`; Phase 2 adds `search` (team name), `section`, `submissionStatus` (`not_submitted`, `checked`, `ai_reviewed`, `graded`, `published`), `gradeStatus` (`none`, `draft`, `approved`, `published`).
Response 200 item:

```json
{
  "teamId": "uuid", "teamName": "CSE4104-7C-T03", "section": "7C",
  "leader": { "userId": "uuid", "fullName": "Rafi Ahmed" },
  "submissionId": "uuid", "submissionStatus": "checked", "isLate": false,
  "gradeStatus": "draft", "lastSyncAt": "2026-11-08T10:00:00Z"
}
```

For a team without a submission: `submissionId` is `null`, `submissionStatus` is `not_submitted`, `isLate` is `null`, and `gradeStatus` is `none`.

#### `GET /milestones/:milestoneId/grades/export` (Phase 2)
Auth: Instructor. Response 200 `text/csv` with header `team name,section,status,late,total score,max marks,grade status`. One row per team; teams without a submission have status `not_submitted`.

### 5.5 Submissions

#### `POST /teams/:teamId/milestones/:milestoneId/submissions`
Auth: Leader. Content type `multipart/form-data`.

| Part                       | Rules                                                                                                                |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `liveUrl`                  | Required when the milestone requires it; `http` or `https` URL, at most 2,000 characters                             |
| `notes`                    | Optional, at most 2,000 characters                                                                                   |
| one file part per document | The part name is the requirement `key`; the file name must equal the pattern with `{TEAM}` replaced by the team name |

The repository is always the team's repository; it is not sent.

Order of processing:

1. `receivedAt` is captured as the very first step. The upload middleware limits each file to 10 MB; a larger file is rejected with `FILE_TOO_LARGE` (422) before validation, without a per-check report.
2. Guards: Leader, milestone `published` (`MILESTONE_NOT_PUBLISHED`), no existing submission (`SUBMISSION_EXISTS`), team size at least the minimum (`TEAM_TOO_SMALL`).
3. Validation runs **before anything is stored** (all checks are blocking, see 5.5.1).
4. If every check passes, the files are uploaded to private storage. If an upload fails, uploaded objects are deleted and the response is `UPSTREAM_ERROR` (502).
5. One database transaction inserts the submission (`status = checked`, `submittedAt = receivedAt`, `isLate` set once from `receivedAt` versus the deadline), the file rows and the validation results. If it fails, the uploaded objects are deleted. A unique-constraint violation (SQLSTATE `23505`) on team and milestone is returned as `SUBMISSION_EXISTS`.
6. A GitHub sync (trigger `submission`, no cooldown) is started in the background.

Response 201: `Submission`.
Errors: `NOT_TEAM_LEADER`, `MILESTONE_NOT_PUBLISHED`, `SUBMISSION_EXISTS`, `TEAM_TOO_SMALL`, `FILE_TOO_LARGE`, `VALIDATION_ERROR` and the specific codes below, `UPSTREAM_ERROR`.

A late submission is accepted and marked late; there is no cut-off and no automatic penalty. A submission cannot be replaced or edited.

##### 5.5.1 Validation checks

Checks run in this order and every check reports its own result:

| Order | `checkType`             | Code on failure                                                      |
| ----- | ----------------------- | -------------------------------------------------------------------- |
| 1     | `required_fields`       | `VALIDATION_ERROR`                                                   |
| 2     | `team_name_format`      | `VALIDATION_ERROR`                                                   |
| 3     | `file_names`            | `FILENAME_MISMATCH`                                                  |
| 4     | `file_types`            | `INVALID_FILE_TYPE`                                                  |
| 5     | `file_sizes`            | `FILE_TOO_LARGE`                                                     |
| 6     | `repository_accessible` | `REPOSITORY_UNREACHABLE` or `GITHUB_RATE_LIMITED`                    |
| 7     | `live_url`              | `VALIDATION_ERROR` (bad format); Phase 2 also `LIVE_URL_UNREACHABLE` |

A check that does not apply (live URL not required) is `skipped`; a check whose input is missing because an earlier check failed is also `skipped`. `repository_accessible` passes only when GitHub returns the team's repository and it is not private. In Phase 1 the `live_url` check only confirms the format and never makes a request to the submitted URL. If any check fails, the request returns HTTP 422 with the code of the **first failed check** and the full report in `details`; nothing is stored:

```json
{
  "success": false,
  "error": {
    "code": "FILENAME_MISMATCH",
    "message": "Submission failed validation. Nothing was stored.",
    "details": {
      "checks": [
        { "checkType": "required_fields", "status": "passed", "message": "All required fields are present.", "details": {} },
        { "checkType": "file_names", "status": "failed",
          "message": "Expected CSE4104-7C-T03_Proposal.pdf but received proposal.pdf.",
          "details": { "requirementKey": "proposal", "expected": "CSE4104-7C-T03_Proposal.pdf", "received": "proposal.pdf" } }
      ]
    }
  }
}
```

If GitHub cannot be reached at that moment, the check fails and the leader may try again later. Phase 2 live URL reachability follows redirects, accepts only `http` and `https` on ports 80 and 443, refuses private, loopback and link-local addresses (including after each redirect, message `URL is not allowed.`), and stores final status, redirect count and response time in `details`.

#### `POST /teams/:teamId/milestones/:milestoneId/submissions/check` (Phase 2)
Auth: Leader. Content type `multipart/form-data`, same parts as the submit endpoint. Runs the same guards and the same validation, but uploads nothing, stores nothing and starts no sync.
Response 200 (always, also when checks fail): `{ "passed": true, "checks": [ { "checkType", "status", "message", "details" } ] }`.
Errors: `NOT_TEAM_LEADER`, `MILESTONE_NOT_PUBLISHED`, `SUBMISSION_EXISTS`, `TEAM_TOO_SMALL`, `FILE_TOO_LARGE`.

#### `GET /teams/:teamId/submissions`
Auth: Member. Response 200: list (not paginated, at most 14) of `Submission` summaries with `milestone: { "id", "weekNumber", "title", "deadline" }` and without `files` and `validation`. `status` follows the status rule of Section 4.

#### `GET /submissions/:submissionId`
Auth: Member. Response 200: `Submission`. Errors: `NOT_FOUND`, `NOT_TEAM_MEMBER`.

#### `GET /submissions/:submissionId/files/:fileId/download`
Auth: Member. Response 200: `{ "url": "signed url", "expiresInSeconds": 60 }`. The storage path is never returned.
Errors: `NOT_FOUND`, `NOT_TEAM_MEMBER`.

### 5.6 GitHub sync, commits and metrics

#### `POST /teams/:teamId/github/sync`
Auth: Leader or Instructor (other students get `NOT_TEAM_LEADER`). No body.
The sync is refused with `SYNC_COOLDOWN` (429, details `{ "retryAfterSeconds" }`) if the team's last sync finished less than 5 minutes ago, and with `SYNC_IN_PROGRESS` if a run is queued or running.
Response 202: `Sync run` with status `queued`. The UI polls the latest run.
Errors: `SYNC_COOLDOWN`, `SYNC_IN_PROGRESS`, `NOT_TEAM_LEADER`, `NOT_FOUND`.

The background worker lists the branches, lists commits per branch (only commits since the offering start date), keeps the newest 2,000 distinct commits across all branches (`truncated = true` if more exist), fetches details only for SHAs not yet stored, stores commits in batches, and attributes commits to members in this order: GitHub author login equals the member's GitHub username; author email equals the member's registered email; author email is a GitHub noreply address containing the member's GitHub username. Matching is case-insensitive. Other commits are Unassigned. Failures are stored on the run (`GITHUB_REPO_NOT_FOUND`, `GITHUB_RATE_LIMITED`, `INTERRUPTED`); commits stored before a failure stay, and stored commits are never deleted.

#### `GET /teams/:teamId/github/sync-runs/latest`
Auth: Member. Response 200: `Sync run`, or `data: null` if the team has never synced.

#### `GET /teams/:teamId/commits`
Auth: Member. Query: `milestoneId` (limits to that milestone's period), `memberId`, `unassigned` (`true`), `counted` (`true` or `false`), `page`, `pageSize`. Newest first.
Response 200: list of `Commit` with `meta`.

#### `GET /teams/:teamId/metrics`
Auth: Member. Query: `milestoneId` (optional). Without it the period is the whole offering (start date 00:00 Asia/Dhaka until now).

The period for a milestone starts at the deadline of the published milestone with the next lower week number (otherwise the offering start date at 00:00 Asia/Dhaka) and ends at the milestone deadline, or at the team's submission time if the submission exists and is later. Only counted commits are used (merge and bot commits are excluded according to the course settings). `sharePercent` is a member's counted commits divided by all counted commits of the team, including Unassigned ones. Days and weeks are calculated in Asia/Dhaka, weeks start on Monday.

Response 200:

```json
{
  "period": { "start": "…", "end": "…", "milestoneId": "uuid" },
  "excluded": { "mergeCommits": 4, "botCommits": 0 },
  "team": {
    "countedCommits": 52, "activeDays": 14, "additions": 2300, "deletions": 410,
    "filesChanged": 96, "unassignedCommits": 3,
    "firstCommitAt": "…", "lastCommitAt": "…"
  },
  "members": [
    { "userId": "uuid", "fullName": "Rafi Ahmed", "commits": 20, "activeDays": 8,
      "additions": 900, "deletions": 120, "filesChanged": 40, "sharePercent": 38.5,
      "firstCommitAt": "…", "lastCommitAt": "…",
      "weekly": [ { "weekStart": "2026-09-28", "commits": 6, "linesChanged": 310 } ] }
  ],
  "timeline": [ { "date": "2026-10-01", "commits": 5 } ]
}
```

Metrics use commit metadata only and are never a definitive measure of individual effort or quality; the UI shows this disclaimer next to every chart.

### 5.7 AI review (Instructor only)

AI endpoints are never available to students or Team Leaders, and their data never appears in responses to them. The exact AI input, commit refs, output schema and prompt are in `09-ai-integration-spec.md`.

#### `POST /submissions/:submissionId/ai-reviews`
Auth: Instructor. No body. Requires at least one counted commit in the period (`NO_COMMIT_DATA`), no pending review for the submission (`AI_REVIEW_IN_PROGRESS`), and at most 10 requests by this Instructor in the last hour (`RATE_LIMITED`).
Creates a new report with status `pending` and returns 202 with the `AI review` object; earlier reports are kept.

Background steps: compute metrics for the period; anonymise and sanitise (member names, usernames and emails become `Member A`, `Member B`, …; unmatched authors become `Unassigned Author 1`, …; the same replacement is applied inside commit messages; source code, file contents, file paths, diffs, repository URLs and team names are never sent); number the newest 400 commits of the period `c1`, `c2`, … and send them with the metrics and the milestone title and guidelines; ask Gemini for JSON only; validate against the fixed output schema (one retry with the error appended; then `AI_INVALID_OUTPUT`); drop flags that do not cite at least one ref present in the sent input (counted in `droppedFlagCount`); map refs back to SHAs; store the result and flags and set the report `completed`.
Errors: `NO_COMMIT_DATA`, `AI_REVIEW_IN_PROGRESS`, `RATE_LIMITED`, `NOT_FOUND`.

#### `GET /submissions/:submissionId/ai-reviews`
Auth: Instructor. Query: `includeDismissed` (default `false`). Response 200: list of `AI review`, newest first.

#### `GET /ai-reviews/:reviewId`
Auth: Instructor. Response 200: `AI review`. Labels are mapped back to real names only here, at view time.

```json
{
  "id": "uuid", "submissionId": "uuid", "status": "completed",
  "model": "gemini-…", "promptVersion": "v1.0",
  "periodStart": "…", "periodEnd": "…",
  "requestedBy": { "id": "uuid", "fullName": "Instructor" },
  "createdAt": "…", "completedAt": "…", "dismissedAt": null,
  "errorCode": null, "errorMessage": null,
  "result": {
    "summary": "…",
    "memberSummaries": [ { "label": "Member A", "member": { "userId": "uuid", "fullName": "Rafi Ahmed" }, "summary": "…" } ],
    "droppedFlagCount": 1
  },
  "flags": [
    { "id": "uuid", "flagType": "low_information_commit_messages", "severity": "medium",
      "member": { "userId": "uuid", "fullName": "Rafi Ahmed" },
      "title": "…", "description": "…",
      "evidenceCommits": [ { "sha": "…", "htmlUrl": "https://github.com/org/repo/commit/…" } ] }
  ],
  "tokensInput": 5200, "tokensOutput": 900
}
```

`member` is `null` for an Unassigned Author label. `flagType` is one of `low_information_commit_messages`, `off_topic_commits`, `repetitive_trivial_commits`; `severity` is `low`, `medium` or `high`. `result` and `flags` are `null` until the report is `completed`. A failed report has `errorCode` (`AI_UNAVAILABLE`, `AI_INVALID_OUTPUT` or `INTERRUPTED`). All AI content is labelled as advisory in the UI.

#### `POST /ai-reviews/:reviewId/dismiss`
Auth: Instructor. The report stays stored, is hidden by default and no longer counts for the `ai_reviewed` status. The submission status is recomputed.
Response 200: `AI review`. Error: `NOT_FOUND`.

### 5.8 Grading

Grade lifecycle: `draft` → `approved` → `published`. Save keeps `draft`; approve moves `draft` → `approved`; publish moves `approved` → `published`; unpublish moves `published` → `approved`; saving an `approved` grade returns it to `draft`; a `published` grade cannot be edited. Every change recomputes the submission status (`published` for a published grade, `graded` for an approved grade, `ai_reviewed` if a completed non-dismissed AI report exists, otherwise `checked`). AI never fills, approves or publishes a grade.

#### `GET /submissions/:submissionId/grade`
Auth: Member. Instructors receive the grade in any status (or `data: null` if none exists). Students and Team Leaders receive it only when it is `published`; otherwise `data: null` with HTTP 200, so the response never reveals whether an unpublished grade exists.
Response 200: `Grade` or `null`.

#### `PUT /submissions/:submissionId/grade`
Auth: Instructor. Saves a draft. Body: `{ "feedback", "scores": [ { "criterionId", "score", "comment" } ] }`.

| Field         | Rules                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------- |
| `feedback`    | At most 5,000 characters                                                                      |
| `criterionId` | Must belong to the milestone's rubric                                                         |
| `score`       | `null` (not scored yet) or a number from 0 to the criterion's `maxPoints`, at most 2 decimals |
| `comment`     | At most 1,000 characters                                                                      |

`totalScore` is the sum of the criterion scores. Creates the grade if it does not exist.
Response 200: `Grade`. Errors: `VALIDATION_ERROR`, `GRADE_PUBLISHED`, `FORBIDDEN`.

#### `POST /submissions/:submissionId/grade/approve`
Auth: Instructor. No body. Requires every criterion to have a score and a comment of at least 10 characters.
Response 200: `Grade` with status `approved`. Errors: `GRADE_INCOMPLETE` (details list the criteria that are incomplete; also returned when no grade has been saved), `INVALID_GRADE_TRANSITION`, `GRADE_PUBLISHED`.

#### `POST /submissions/:submissionId/grade/publish`
Auth: Instructor. No body. Response 200: `Grade` with status `published`; students can now see it.
Errors: `INVALID_GRADE_TRANSITION` (grade is not `approved`).

#### `POST /submissions/:submissionId/grade/unpublish`
Auth: Instructor. No body. Response 200: `Grade` with status `approved`; hidden from students again.
Errors: `INVALID_GRADE_TRANSITION` (grade is not `published`).