# CourseLens

![Status](https://img.shields.io/badge/status-in%20active%20development-orange)
![Stage](https://img.shields.io/badge/stage-documentation%20complete%2C%20implementation%20pending-blue)
![Branch](https://img.shields.io/badge/main%20branch-documentation%20%26%20structure-lightgrey)
![Course](https://img.shields.io/badge/course-CSE%204104-green)
![License](https://img.shields.io/badge/license-not%20specified-lightgrey)

> CourseLens is a planned academic project-management system that combines coursework submissions, GitHub contribution evidence, rubric-based assessment, and AI-assisted instructor feedback.

**CSE 4104 – Software Development III** | Northern University of Business and Technology, Khulna
Department of Computer Science and Engineering

| Item                  | Details                                                                        |
| --------------------- | ------------------------------------------------------------------------------ |
| Project title         | CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System |
| Repository status     | In active development (planning and documentation phase)                       |
| Repository visibility | Public *(update if this changes)*                                              |
| Course context        | Semester project for CSE 4104, Section 7C, Team CSE4104-7C-T03                 |
| Main branch           | Contains the repository structure and documentation; no application code yet   |

---

## Table of Contents

- [CourseLens](#courselens)
  - [Table of Contents](#table-of-contents)
  - [Project Overview](#project-overview)
  - [Problem Statement](#problem-statement)
  - [User Roles](#user-roles)
  - [Planned Features](#planned-features)
    - [Proposed core features](#proposed-core-features)
    - [Future scope (only if time allows)](#future-scope-only-if-time-allows)
    - [Implemented features](#implemented-features)
  - [Implementation Status](#implementation-status)
    - [Directory placeholders disclaimer](#directory-placeholders-disclaimer)
  - [Detailed Requirements Documentation](#detailed-requirements-documentation)
  - [Documentation Navigation](#documentation-navigation)
  - [Technology Stack](#technology-stack)
    - [Planned frontend](#planned-frontend)
    - [Planned backend](#planned-backend)
    - [Planned database](#planned-database)
    - [Planned integrations](#planned-integrations)
    - [Development and design tools](#development-and-design-tools)
  - [Repository Structure](#repository-structure)
  - [Branching Strategy](#branching-strategy)
    - [Existing branches](#existing-branches)
    - [Workflow before implementation begins](#workflow-before-implementation-begins)
  - [Getting Started](#getting-started)
  - [Environment Variables](#environment-variables)
  - [Testing](#testing)
  - [Planned Deployment](#planned-deployment)
  - [Team](#team)
  - [Project Timeline and Status](#project-timeline-and-status)
    - [Development plan (planned schedule, not completed work)](#development-plan-planned-schedule-not-completed-work)
    - [Progress summary](#progress-summary)
  - [License](#license)

---

## Project Overview

**CourseLens** brings coursework submissions, GitHub contribution data, and AI-assisted feedback into one workflow. The planned process is:

1. The instructor posts weekly milestones.
2. Teams submit their work with a GitHub repository link and a live deployment URL.
3. The system validates each submission and reads the GitHub commit history.
4. An AI assistant (Google Gemini) produces a progress summary and a draft rubric-based review.
5. The instructor reviews everything and makes the final grading decision.

**Key design ideas**

- **Human in the loop:** the AI only suggests. The instructor approves or changes every grade.
- **Evidence-based:** each AI finding should link back to the commits that support it.
- **Privacy-aware:** only commit metadata (message, date, size of change) is planned to be sent to the AI, and student names are replaced with anonymous labels such as "Member A".
- **Reusable:** courses, rubrics, milestones, and naming rules are intended to be configurable for other sections and semesters.

---

## Problem Statement

In team-based project courses, an instructor has to follow many teams across many weekly deliverables. The work is spread over Google Classroom, GitHub repositories, live deployment links, and chat messages. Checking all of this by hand is slow, so problems such as inactive members (free-riding) and last-minute bulk pushes are often found only at the final viva. Students also receive feedback late and have little visibility into their own progress.

---

## User Roles

| Role        | Description                                                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Instructor  | Creates milestones, reviews submissions, AI reports, and contribution data, and approves or edits final grades and feedback. |
| Team Leader | Registers the team, submits weekly deliverables, and views team progress and approved feedback.                              |
| Student     | Views their own contribution summary and approved feedback.                                                                  |

Detailed permissions are documented in [`04-roles-and-permissions.md`](documentation/src/04-roles-and-permissions.md).

---

## Planned Features

> ⚠️ **None of the features below are implemented yet.** They describe the proposed scope of the project.

### Proposed core features

- Authentication and role-based access for Instructor, Team Leader, and Student
- Team registration with the naming rule `CSE4104-[Section]-T[TeamNumber]`
- Weekly milestone creation with deadlines and guidelines
- Submission form with GitHub repository link and live deployment URL
- Automatic checks for team name, file names, required fields, and working links
- GitHub API integration for commit history and per-member contribution metrics (commits per member, active days, change frequency)
- AI review engine (Gemini) for progress summaries, risk flags, and draft rubric-based feedback
- Instructor dashboard with contribution charts, AI report, and approve/edit of grades and feedback
- Student and team view of their own progress and approved feedback

### Future scope (only if time allows)

- AI-suggested viva questions for each student based on their commits
- Email or in-app notifications for deadlines and new feedback
- Weekly report export as PDF

### Implemented features

None yet. This section will be updated as features are completed.

---

## Implementation Status

The repository is currently ahead in planning and documentation compared with implementation.

| Area                       | Status      |
| -------------------------- | ----------- |
| Repository structure       | Complete    |
| Requirements documentation | Complete    |
| Frontend implementation    | Not started |
| Backend implementation     | Not started |
| Database implementation    | Not started |
| Gemini integration         | Not started |
| GitHub integration         | Not started |
| Deployment                 | Not started |

### Directory placeholders disclaimer

Many `frontend/` and `backend/` directories currently contain **planned structure only and no source files**. Please do not assume that any part of the application is already implemented because a folder exists.

---

## Detailed Requirements Documentation

The detailed requirements live in `documentation/src/`. Start with the index:

📑 **[`00-index-and-conventions.md`](documentation/src/00-index-and-conventions.md)**: entry point, naming conventions, and how the documents relate to each other.

| File                                                                                       | Purpose                                                                                                      |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| [`01-business-rules.md`](documentation/src/01-business-rules.md)                           | The rules the system must enforce, such as team naming, submission rules, and grading and approval policies. |
| [`02-functional-requirements.md`](documentation/src/02-functional-requirements.md)         | What the system must do: features and behaviors, grouped by module.                                          |
| [`03-non-functional-requirements.md`](documentation/src/03-non-functional-requirements.md) | Quality attributes such as security, privacy, performance, usability, and reliability.                       |
| [`04-roles-and-permissions.md`](documentation/src/04-roles-and-permissions.md)             | Who can do what: the permission model for Instructor, Team Leader, and Student.                              |
| [`05-use-cases.md`](documentation/src/05-use-cases.md)                                     | Step-by-step user interactions with the system.                                                              |
| [`06-data-model.md`](documentation/src/06-data-model.md)                                   | Entities, attributes, and relationships that the database design is based on.                                |

---

## Documentation Navigation

| Folder                                               | Contents                                    | State                                 |
| ---------------------------------------------------- | ------------------------------------------- | ------------------------------------- |
| [`documentation/src/`](documentation/src/)           | Detailed requirements documents (see above) | Complete                              |
| [`documentation/proposal/`](documentation/proposal/) | Project proposal                            | See folder for current contents       |
| [`documentation/srs/`](documentation/srs/)           | Software Requirements Specification         | See folder for current contents       |
| [`documentation/diagrams/`](documentation/diagrams/) | Use case, ER, and architecture diagrams     | See folder for current contents       |
| [`documentation/reports/`](documentation/reports/)   | Weekly and final reports                    | Placeholder, no reports yet           |
| [`testing/`](testing/)                               | Test cases and test reports                 | Placeholder, no tests yet             |
| [`screenshots/`](screenshots/)                       | UI and progress screenshots                 | Placeholder, contains no screenshots  |
| [`presentation/`](presentation/)                     | Slides and demo material                    | Placeholder, contains no presentation |

---

## Technology Stack

> These are **planned** technologies. Items marked *tentative* have not been decided.

### Planned frontend

- React.js
- Tailwind CSS
- Charting library: *tentative* (Recharts or Chart.js, not yet selected)

### Planned backend

- Node.js
- Express.js
- JWT and bcrypt for authentication

### Planned database

- PostgreSQL
- Hosting provider: *tentative* (Supabase is under consideration but is not a confirmed dependency until it is configured)

### Planned integrations

- Google Gemini API (AI summaries and draft feedback)
- GitHub REST API (commit history and contribution metrics)

### Development and design tools

These are used during development and are not runtime technologies.

- Git and GitHub (version control)
- Figma and Google Stitch AI (UI design)
- Postman (API testing)

---

## Repository Structure

```
cse4104-7c-t03-ai-team-tracking-system/
├── frontend/                 # React + Tailwind client (planned; no manifest yet)
│   ├── public/
│   └── src/                  # Planned structure; implementation pending
│       ├── assets/           # Images, icons, fonts
│       ├── components/       # Reusable UI components
│       ├── context/          # Auth and global state
│       ├── hooks/            # Custom React hooks
│       ├── pages/            # Login, dashboards, submission pages
│       ├── services/         # API calls to the backend
│       └── utils/            # Helper functions
├── backend/                  # Node.js + Express API (planned; no manifest yet)
│   └── src/                  # Planned structure; implementation pending
│       ├── config/           # Environment, database, and app config
│       ├── controllers/      # Request handlers
│       ├── middleware/       # Auth, role checks, validation
│       ├── models/           # Database models and queries
│       ├── routes/           # API route definitions
│       ├── services/
│       │   ├── github/       # GitHub REST API integration
│       │   └── ai/           # Gemini integration and prompts
│       └── utils/            # Helper functions
├── documentation/
│   ├── src/                  # Detailed requirements documentation
│   │   ├── 00-index-and-conventions.md
│   │   ├── 01-business-rules.md
│   │   ├── 02-functional-requirements.md
│   │   ├── 03-non-functional-requirements.md
│   │   ├── 04-roles-and-permissions.md
│   │   ├── 05-use-cases.md
│   │   └── 06-data-model.md
│   ├── proposal/             # Project proposal
│   ├── srs/                  # Software Requirements Specification
│   ├── diagrams/             # Use case, ER, architecture diagrams
│   └── reports/              # Weekly and final reports (no reports yet)
├── screenshots/              # Currently contains no screenshots
├── presentation/             # Currently contains no presentation
├── testing/
│   ├── test-cases/           # Placeholder; no test cases yet
│   └── reports/              # Placeholder; no test reports yet
├── .gitignore
└── README.md
```

---

## Branching Strategy

### Existing branches

| Branch       | Purpose                                                                      |
| ------------ | ---------------------------------------------------------------------------- |
| `main`       | Stable branch containing the repository structure and reviewed documentation |
| `docs`       | Documentation work (requirements, diagrams, reports)                         |
| `shahajalal` | Personal working branch of Md Shahajalal Mahmud                              |

### Workflow before implementation begins

1. Do not commit directly to `main`.
2. Create or use your own branch (or `docs` for documentation changes).
3. Commit small, focused changes with clear messages.
4. Push your branch to GitHub at least once a week.
5. Open a pull request into `main` and request a review from a teammate.
6. Merge only after the review is approved.

Additional feature branches (for example `frontend`, `backend`, `ai-integration`) may be created once implementation starts. They do not exist yet.

---

## Getting Started

Development setup will be available after the frontend and backend applications receive their manifests and entry-point files.

---

## Environment Variables

Not applicable yet. A `.env.example` file will be added when the backend is implemented. The backend is expected to need configuration for the database connection, JWT secret, Gemini API key, and GitHub API token. **Never commit real secrets to the repository.**

---

## Testing

No automated or manual test results exist yet. Test cases and reports will be added under `testing/` during the testing phase (Week 10 of the plan). API testing with Postman is planned.

---

## Planned Deployment

> No deployment has taken place and there are no live links yet.

The following hosting options are **proposed**, not confirmed:

- Frontend: Netlify
- Backend: Render

Deployment details will be documented here after the application is deployed.

---

## Team

**Team name:** CSE4104-7C-T03 | **Section:** 7C | **Course:** CSE 4104 – Software Development III

| Contributor          | Role                              | GitHub                                                  |
| -------------------- | --------------------------------- | ------------------------------------------------------- |
| Md Shahajalal Mahmud | Team Leader & AI Integration Lead | [shahjalal-mahmud](https://github.com/shahjalal-mahmud) |
| Preota Saha          | Frontend Developer                | [Preota-Saha](https://github.com/Preota-Saha)           |
| Md Munna Sardar      | Database Manager                  | [Mdmunnasardar](https://github.com/Mdmunnasardar)       |
| Hazera Islam Mim     | Backend Developer                 | [hazeramim](https://github.com/hazeramim)               |

---

## Project Timeline and Status

### Development plan (planned schedule, not completed work)

| Week | Planned work                                          | Main owner        |
| ---- | ----------------------------------------------------- | ----------------- |
| 2    | Proposal, GitHub repository, README, team sheet       | Everyone          |
| 3    | SRS document and first diagrams                       | Leader, Backend   |
| 4    | Architecture, ER diagram, API list                    | Database, Backend |
| 5    | Figma wireframes (login, dashboards, submission page) | Frontend          |
| 6    | Backend: authentication, database, main APIs          | Backend, Database |
| 7    | Frontend pages and API connection                     | Frontend          |
| 8    | GitHub API and Gemini integration                     | Leader (AI Lead)  |
| 9    | Feature completion, dashboard charts, major bug fixes | Everyone          |
| 10   | Testing: functional, API, UI, and AI response checks  | Everyone          |
| 11   | Deployment (Netlify and Render)                       | Leader, Backend   |
| 12   | Final report, user manual, API documentation          | Everyone          |
| 13   | Presentation slides and demo practice                 | Everyone          |
| 14   | Final presentation and viva                           | Everyone          |

### Progress summary

- **Done:** proposal submitted, repository created, project structure added, and detailed requirements documentation written (business rules, functional and non-functional requirements, roles and permissions, use cases, data model, architecture and API planning).
- **Next:** Figma wireframes, then backend and frontend implementation.
- **Not started:** all application code, database, AI and GitHub integrations, testing, and deployment.

---

## License

No license is currently specified.
A suitable course or open-source license will be added before public distribution.