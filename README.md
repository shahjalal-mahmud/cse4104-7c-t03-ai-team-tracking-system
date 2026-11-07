# CourseLens

**AI-Powered Academic Coursework & Team Contribution Tracking System**

CSE 4104 – Software Development III | Northern University of Business and Technology, Khulna
Department of Computer Science and Engineering

---

## Project Title

**CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System**

---

## Team Information

| Field       | Details                             |
| ----------- | ----------------------------------- |
| Team Name   | CSE4104-7C-T03                      |
| Section     | 7C                                  |
| Course      | CSE 4104 – Software Development III |
| Team Leader | Md Shahajalal Mahmud                |

| No. | Name                 | Student ID  | Role                              | GitHub Profile                                          |
| --- | -------------------- | ----------- | --------------------------------- | ------------------------------------------------------- |
| 1   | Md Shahajalal Mahmud | 11230321539 | Team Leader & AI Integration Lead | [shahjalal-mahmud](https://github.com/shahjalal-mahmud) |
| 2   | Preota Saha          | 11230321538 | Frontend Developer                | [Preota-Saha](https://github.com/Preota-Saha)           |
| 3   | Md Munna Sardar      | 11230321540 | Database Manager                  | [Mdmunnasardar](https://github.com/Mdmunnasardar)       |
| 4   | Hazera Islam Mim     | 11230321542 | Backend Developer                 | [hazeramim](https://github.com/hazeramim)               |

---

## Project Description

In team-based project courses, an instructor has to follow many teams across many weekly deliverables. The work is spread over Google Classroom, GitHub repositories, live deployment links and chat messages. Checking all of this by hand is slow, so problems such as inactive members (free-riding) and last-minute bulk pushes are often found only at the final viva. Students also get feedback late and have little view of their own progress.

**CourseLens** brings submissions, GitHub contribution data and AI feedback into one workflow. The instructor posts weekly milestones, teams submit their work, the system checks each submission and reads the GitHub commit history, and an AI assistant (Google Gemini) writes a progress summary and a draft rubric-based review. The instructor reviews everything and makes the final grading decision.

**Key ideas**

- **Human in the loop:** the AI only suggests. The instructor approves or changes every grade.
- **Evidence-based:** each AI finding links back to the commits that support it.
- **Privacy-aware:** only commit metadata (message, date, size of change) is sent to the AI, and student names are replaced with anonymous labels such as "Member A".
- **Reusable:** courses, rubrics, milestones and naming rules are configurable for other sections and semesters.

---

## Proposed Features

**Core features**

- Authentication and role-based access for Instructor, Team Leader and Student
- Team registration with the official naming rule `CSE4104-[Section]-T[TeamNumber]`
- Weekly milestone creation with deadlines and guidelines
- Submission form with GitHub repository link and live deployment URL
- Automatic checks for team name, file names, required fields and working links
- GitHub API integration for commit history and per-member contribution metrics (commits per member, active days, change frequency)
- AI review engine (Gemini) for progress summaries, risk flags and draft rubric-based feedback
- Instructor dashboard with contribution charts, AI report and one-click approve or edit of grades and feedback
- Student and team view of their own progress and approved feedback

**Extra features (if time allows)**

- AI-suggested viva questions for each student based on their commits
- Email or in-app notifications for deadlines and new feedback
- Weekly report export as PDF

---

## Technology Stack

| Layer           | Technology                           |
| --------------- | ------------------------------------ |
| Frontend        | React.js, Tailwind CSS               |
| Backend         | Node.js, Express.js                  |
| Database        | PostgreSQL (hosted on Supabase)      |
| Authentication  | JWT, bcrypt                          |
| AI Integration  | Google Gemini API                    |
| External API    | GitHub REST API                      |
| Charts          | Recharts (or Chart.js)               |
| Design          | Figma, Google Stitch AI              |
| API Testing     | Postman                              |
| Version Control | Git and GitHub                       |
| Deployment      | Netlify (frontend), Render (backend) |

---

## Project Structure

```
cse4104-7c-t03-ai-team-tracking-system/
├── frontend/                 # React + Tailwind client
│   ├── public/
│   └── src/
│       ├── assets/           # Images, icons, fonts
│       ├── components/       # Reusable UI components
│       ├── context/          # Auth and global state
│       ├── hooks/            # Custom React hooks
│       ├── pages/            # Login, dashboards, submission pages
│       ├── services/         # API calls to the backend
│       └── utils/            # Helper functions
├── backend/                  # Node.js + Express API
│   └── src/
│       ├── config/           # Environment, database and app config
│       ├── controllers/      # Request handlers
│       ├── middleware/       # Auth, role checks, validation
│       ├── models/           # Database models and queries
│       ├── routes/           # API route definitions
│       ├── services/
│       │   ├── github/       # GitHub REST API integration
│       │   └── ai/           # Gemini integration and prompts
│       └── utils/            # Helper functions
├── documentation/
│   ├── proposal/             # Project proposal
│   ├── srs/                  # Software Requirements Specification
│   ├── diagrams/             # Use case, ER, architecture diagrams
│   └── reports/              # Weekly and final reports
├── screenshots/              # UI and progress screenshots
├── presentation/             # Slides and demo material
├── testing/
│   ├── test-cases/           # Test case documents
│   └── reports/              # Test result reports
├── .gitignore
└── README.md
```

---

## Branches

| Branch           | Purpose                       |
| ---------------- | ----------------------------- |
| `main`           | Stable, submission-ready code |
| `development`    | Merged work in progress       |
| `frontend`       | UI work                       |
| `backend`        | API and database work         |
| `ai-integration` | Gemini and GitHub API work    |

Every member pushes to GitHub every week with clear commit messages, using separate branches.

---

## Development Plan

| Week | Work                                                  | Main Owner        |
| ---- | ----------------------------------------------------- | ----------------- |
| 2    | Proposal, GitHub repository, README, team sheet       | Everyone          |
| 3    | SRS document and first diagrams                       | Leader, Backend   |
| 4    | Architecture, ER diagram, API list                    | Database, Backend |
| 5    | Figma wireframes (login, dashboards, submission page) | Frontend          |
| 6    | Backend: authentication, database, main APIs          | Backend, Database |
| 7    | Frontend pages and API connection                     | Frontend          |
| 8    | GitHub API and Gemini integration                     | Leader (AI Lead)  |
| 9    | Feature completion, dashboard charts, major bug fixes | Everyone          |
| 10   | Testing: functional, API, UI and AI response checks   | Everyone          |
| 11   | Deployment (Netlify and Render)                       | Leader, Backend   |
| 12   | Final report, user manual, API documentation          | Everyone          |
| 13   | Presentation slides and demo practice                 | Everyone          |
| 14   | Final presentation and viva                           | Everyone          |

---

## Installation

The project is in the planning stage. Setup steps will be added after development starts.

## Deployment Link

Will be added in Week 11.

## Status

**Week 01** –
**Week 02** – Proposal submitted, repository created and initial project structure added.
