# CourseLens

**AI-Powered Academic Coursework & Team Contribution Tracking System**

CSE 4104 – Software Development III | Northern University of Business and Technology, Khulna
Department of Computer Science and Engineering

---

## Team Information

| Field | Details |
|---|---|
| Team Name | CSE4104-7C-T03 |
| Section | 7C |
| Project Title | CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System |
| Team Leader | Md Shahajalal Mahmud |



| No. | Name                 | Student ID  | Role                              | GitHub Profile URL                                                             |
| --- | -------------------- | ----------- | --------------------------------- | ------------------------------------------------------------------------------ |
| 1   | Md Shahajalal Mahmud | 11230321539 | Team Leader & AI Integration Lead | [shahjalal-mahmud](https://github.com/shahjalal-mahmud?utm_source=chatgpt.com) |
| 2   | Preota Saha          | 11230321538 | Frontend Developer                | [Preota-Saha](https://github.com/Preota-Saha?utm_source=chatgpt.com)           |
| 3   | Md Munna Sardar      | 11230321540 | Database Manager                  | [Mdmunnasardar](https://github.com/Mdmunnasardar?utm_source=chatgpt.com)       |
| 4   | Hazera Islam Mim     | 11230321542 | Backend Developer                 | [hazeramim](https://github.com/hazeramim?utm_source=chatgpt.com)               |

---

## Project Description

In team-based project courses, an instructor has to follow many teams across many weekly deliverables. The work is spread over Google Classroom, GitHub repositories, live links and chat messages. Checking all of this by hand is slow, so inactive members and last-minute bulk pushes are often found only at the final viva.

**CourseLens** brings submissions, GitHub contribution data and AI feedback into one workflow. The instructor posts weekly milestones, teams submit their work, the system checks the submission and reads the GitHub commit history, and an AI assistant writes a progress summary and a draft review. The instructor reviews everything and makes the final grading decision.

**Key ideas**

- **Human in the loop:** the AI only suggests. The instructor approves or changes every grade.
- **Evidence-based:** each AI finding links back to the commits that support it.
- **Privacy-aware:** only commit metadata is sent to the AI, and student names are replaced with anonymous labels.
- **Reusable:** courses, rubrics, milestones and naming rules are configurable.

---

## Proposed Features

**Core features**

- Authentication and role-based access (Instructor, Team Leader, Student)
- Team registration with the official naming rule `CSE4104-[Section]-T[TeamNumber]`
- Weekly milestone creation with deadlines and guidelines
- Submission form with GitHub repository link and live deployment URL
- Automatic checks for team name, file names, required fields and working links
- GitHub API integration for commit history and per-member contribution metrics
- AI review engine (Gemini) for progress summaries, risk flags and draft rubric feedback
- Instructor dashboard with contribution charts, AI report and one-click grade approval
- Student and team progress view

**Extra features (if time allows)**

- AI-suggested viva questions for each student
- Deadline and feedback notifications
- Weekly report export as PDF

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React.js, Tailwind CSS |
| Backend | Node.js, Express.js |
| Database | PostgreSQL (Supabase) |
| Authentication | JWT, bcrypt |
| AI Integration | Google Gemini API |
| External API | GitHub REST API |
| Charts | Recharts |
| Design | Figma |
| API Testing | Postman |
| Deployment | Netlify (frontend), Render (backend) |

---

## Planned Project Structure

```
cse4104-7c-t03-courselens/
├── frontend/         # React app
├── backend/          # Express API
├── documentation/    # Proposal, SRS, diagrams, reports
├── screenshots/      # UI and progress screenshots
├── presentation/     # Slides
├── testing/          # Test cases and reports
└── README.md
```

---

## Branches

- `main` – stable code
- `development` – merged work in progress
- `frontend` – UI work
- `backend` – API and database work
- `ai-integration` – Gemini and GitHub API work

---

## Development Plan

| Week | Work |
|---|---|
| 2 | Proposal and repository setup |
| 3 | SRS |
| 4 | System design |
| 5 | UI/UX design |
| 6 | Backend development |
| 7 | Frontend development |
| 8 | AI integration |
| 9 | Feature completion |
| 10 | Testing and debugging |
| 11 | Deployment |
| 12 | Documentation |
| 13 | Presentation preparation |
| 14 | Final presentation and viva |

---

## Installation

The project is in the planning stage. Setup steps will be added after development starts.

## Deployment Link

Will be added in Week 11.

## Status

Week 02 – Proposal submitted.
