# 04 – Roles and Permissions

## 1. Roles

CourseLens has **two database roles** and **one contextual role**.

| Role        | Stored as                                                                       | How it is determined                                     | Created by                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Instructor  | `users.role = 'instructor'`                                                     | JWT claim `role`                                         | Seed script or another Instructor                                                                                                  |
| Student     | `users.role = 'student'`                                                        | JWT claim `role`                                         | Self-registration (always creates role `student`)                                                                                  |
| Team Leader | **Not stored on the user.** `team_members.is_leader = true` for the user's team | Looked up in the database on every request that needs it | Created automatically when a student creates a team, or when the current leader transfers leadership to another member of the team |

A user's role never changes. The words "Team Leader" always mean *a student whose team membership has `is_leader = true`*. Leadership is never read from the token, because it can change. A team has exactly one leader at all times. Instructors cannot become team leaders, cannot change a team's leader, and cannot change team membership.

External actors (no login): **GitHub REST API** and **Google Gemini API**. They are called only by the backend.

## 2. Token

JWT, algorithm HS256, sent as `Authorization: Bearer <token>`.

| Claim        | Value                                |
| ------------ | ------------------------------------ |
| `sub`        | User UUID                            |
| `role`       | `instructor` or `student`            |
| `iat`, `exp` | Issued-at and expiry (8 hours later) |

No other claims. On each authenticated request the backend loads the user row (id, role, full name) and rejects the request with `UNAUTHENTICATED` if the user no longer exists. The loaded user is attached as `req.user = { id, role, fullName }`.

## 3. Guards (middleware)

| Guard                       | Behaviour                                                                                                                                                                                               | Failure                                                       |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `requireAuth`               | Verifies the JWT, loads the user, sets `req.user`                                                                                                                                                       | 401 `UNAUTHENTICATED`                                         |
| `requireRole(...roles)`     | `req.user.role` must be in the list                                                                                                                                                                     | 403 `FORBIDDEN`                                               |
| `loadTeamAccess`            | Reads `:teamId`; loads the team; sets `req.team` and `req.membership` (null for an Instructor). An Instructor always passes (read access only). A student passes only if they are a member of that team | 404 `NOT_FOUND` if the team is missing; 403 `NOT_TEAM_MEMBER` |
| `requireTeamLeader`         | Must run after `loadTeamAccess`. Passes only if `req.membership.isLeader` is true. Instructors do **not** pass it                                                                                       | 403 `NOT_TEAM_LEADER`                                         |
| `requireLeaderOrInstructor` | Must run after `loadTeamAccess`. Passes if `req.user.role` is `instructor`, or if `req.membership.isLeader` is true. Used only by the manual sync route                                                 | 403 `NOT_TEAM_LEADER`                                         |

For routes with `:submissionId`, the controller loads the submission, then reuses the team access logic through `teamAccessService.assertCanViewTeam(user, teamId)` and `assertIsLeader(user, teamId)`.

Every route that changes a team (edit title, add or remove a member, transfer leadership, student lookup, create a submission, dry-run check) uses this chain: `requireAuth` → `requireRole('student')` → `loadTeamAccess` → `requireTeamLeader`. Because Instructors fail this chain, they can never change a team or submit for it.

The manual sync route uses: `requireAuth` → `loadTeamAccess` → `requireLeaderOrInstructor`.

## 4. Permission matrix (actions)

"Own team" = a team the user is a member of. "Any" = any team.

| Action                                                                | Instructor                | Team Leader              | Student (non-leader)     | Visitor |
| --------------------------------------------------------------------- | ------------------------- | ------------------------ | ------------------------ | ------- |
| Register                                                              | –                         | –                        | –                        | Yes     |
| Log in                                                                | Yes                       | Yes                      | Yes                      | Yes     |
| View/edit own profile, change own password                            | Yes                       | Yes                      | Yes                      | No      |
| Create instructor account                                             | Yes                       | No                       | No                       | No      |
| Set a new password for a student                                      | Yes                       | No                       | No                       | No      |
| View course settings (non-secret fields)                              | Yes                       | Yes                      | Yes                      | No      |
| View `registrationCode`                                               | Yes                       | No                       | No                       | No      |
| Edit course settings (including minimum and maximum team size)        | Yes                       | No                       | No                       | No      |
| Create a team (when not in a team)                                    | No                        | –                        | Yes                      | No      |
| View a team and its members                                           | Any (read only)           | Own team                 | Own team                 | No      |
| List all teams                                                        | Yes                       | No                       | No                       | No      |
| Edit team `projectTitle`                                              | No                        | Own team                 | No                       | No      |
| Edit team `repositoryUrl`                                             | No                        | No (permanent)           | No                       | No      |
| Add members and remove non-leader members                             | No                        | Own team                 | No                       | No      |
| Transfer leadership to another member                                 | No                        | Own team                 | No                       | No      |
| Look up a student by ID (for adding members)                          | No                        | Own team                 | No                       | No      |
| Create, edit, delete milestone; edit rubric and document requirements | Yes                       | No                       | No                       | No      |
| Publish / unpublish milestone                                         | Yes                       | No                       | No                       | No      |
| List milestones                                                       | All (draft and published) | Published only           | Published only           | No      |
| Create a submission                                                   | No                        | Own team                 | No                       | No      |
| Run a submission dry-run check                                        | No                        | Own team                 | No                       | No      |
| View a team's submission and validation results                       | Any                       | Own team                 | Own team                 | No      |
| See the stored submission status                                      | Yes                       | No (sees student status) | No (sees student status) | No      |
| Download submitted files                                              | Any                       | Own team                 | Own team                 | No      |
| Start GitHub sync                                                     | Any                       | Own team                 | No                       | No      |
| View commits and contribution metrics                                 | Any                       | Own team                 | Own team                 | No      |
| Request, view, dismiss AI reports; view AI flags                      | Yes                       | No                       | No                       | No      |
| Save, approve, publish, unpublish grade                               | Yes                       | No                       | No                       | No      |
| View draft or approved grade                                          | Yes                       | No                       | No                       | No      |
| View **published** grade and feedback                                 | Any                       | Own team                 | Own team                 | No      |
| View instructor dashboard                                             | Yes                       | No                       | No                       | No      |
| View student dashboard                                                | –                         | Yes                      | Yes                      | No      |
| Export grades CSV                                                     | Yes                       | No                       | No                       | No      |

## 5. Data visibility rules

1. **AI content** (reports, AI flags, label maps) is never returned to students or Team Leaders. Endpoints that return a submission to a student must omit the field `aiReviews`.
2. **Grades**: a student receives `data: null` (HTTP 200) from the grade endpoint unless the grade is `published`. The response never reveals whether an unpublished grade exists.
3. **Submission status**: every response to a student or Team Leader that contains a submission status returns the **student status**: `submitted` when the stored status is `checked`, `ai_reviewed` or `graded`, and `published` when the stored status is `published`. The stored values `ai_reviewed` and `graded` are never sent to students, because they would reveal that an AI review or an unpublished grade exists. Instructors always receive the stored status.
4. **Other users' personal data**: a member sees teammates' `fullName`, `studentId`, `githubUsername` and `projectRole`. Email addresses are visible only to Instructors.
5. **Student lookup** returns only `id`, `fullName`, `studentId` and `hasTeam`, and is available only to Team Leaders (for adding members).
6. **File downloads** return a signed URL (valid for 60 seconds) only after the access check passes. Storage paths are never returned.
7. **Password hashes** and `registrationCode` (for non-Instructors) are never returned.
8. **Team isolation**: a student or Team Leader can access the data of their own team only (`NOT_TEAM_MEMBER` otherwise). An Instructor has read access to all teams.