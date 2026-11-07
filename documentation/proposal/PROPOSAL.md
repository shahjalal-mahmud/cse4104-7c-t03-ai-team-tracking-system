# CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System

**Project Proposal | CSE 4104 – Software Development III | Week 02**

| Field       | Details                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------- |
| Team Name   | CSE4104-7C-T03                                                                           |
| Section     | 7C                                                                                       |
| Team Leader | Md Shahajalal Mahmud (11230321539)                                                       |
| Members     | Preota Saha (11230321538), Md Munna Sardar (11230321540), Hazera Islam Mim (11230321542) |

---

## 1. Project Title

AI-Powered Team Contribution & Academic Tracking System

---

## 2. Problem Statement

**What problem exists.** In team-based project courses, an instructor has to follow many teams across many weekly deliverables. The work is spread over different places: Google Classroom, GitHub repositories, live deployment links and chat messages. To check one team, the instructor must open the file, check the file name, open the repository, read the commit history and test the live link. This takes a lot of time and is done mostly by hand.

**Why it matters.** Because the checking is slow, some problems are found very late. A member who does no work (free-riding), or a team that pushes all its code in the last days, is often discovered only at the final viva. Feedback to students also comes late and can be different from team to team. Students have very little view of their own progress during the semester.

**Why existing solutions are not enough.** Current tools solve only one part of the problem:

- **Google Classroom** handles assignments and grades, but it does not look at GitHub or at each member's contribution.
- **GitHub Classroom** is built around code and automatic tests. It is not suitable for documents like proposals or SRS, and it gives limited feedback.
- **gitinspector** shows commit statistics, but it is only a command-line report. It has no submissions, roles or feedback, and raw line counts are easy to game.
- **CATME** collects peer ratings only. It is not connected to real repository activity.

**Why AI is relevant.** Counters and rules can count commits, but they cannot understand what the work was. Commit messages are free text. An AI model can read them and summarise what each member actually worked on, apply the same rubric to every team, and point out risks such as inactivity or a big last-minute push. The AI will only suggest. The instructor always makes the final decision.

---

## 3. Objectives

- Build one web system where the instructor can manage weekly milestones, submissions and grades.
- Check submissions automatically (team name format, file names, required fields, working links).
- Collect GitHub commit data and calculate a simple contribution summary for each student.
- Use the Gemini API to write a short weekly progress summary and a draft rubric-based feedback for each team.
- Flag possible problems such as inactive members and last-minute bulk pushes, with links to the commits that support each finding.
- Give the instructor a dashboard to review the AI report and approve or change the suggested grade.
- Give students a clear view of their own team's progress.

---

## 4. Proposed Features

### Core features

- Authentication and role-based access for Instructor, Team Leader and Student.
- Team registration that checks the official naming rule: `CSE4104-[Section]-T[TeamNumber]`.
- Weekly milestone creation by the instructor, with deadline and guidelines.
- Submission form with GitHub repository link and live deployment URL, plus automatic checks of file names, required fields and link status.
- GitHub API integration to fetch commit history and calculate commits per member, active days and change frequency.
- AI review engine (Gemini) that reads commit logs and the guidelines, then produces a progress summary, risk flags and draft feedback.
- Instructor dashboard with contribution charts, the AI report and one-click approve or edit for grades and feedback.
- Student and team view to see their own progress and the approved feedback.

### Extra features

- AI-suggested viva questions for each student, based on their own commits.
- Email or in-app notifications for deadlines and new feedback.
- Export of a weekly report as PDF.

### Privacy and trust rules

- Only commit metadata (message, date, size of change) is sent to the AI. Student names are replaced with anonymous labels like "Member A".
- Every AI finding shows the commits it is based on, so the instructor can verify it quickly.
- The AI never grades by itself. A human approval is required.

---

## 5. Technology Stack

| Layer           | Technology                           |
| --------------- | ------------------------------------ |
| Frontend        | React.js with Tailwind CSS           |
| Backend         | Node.js with Express.js              |
| Database        | PostgreSQL (hosted on Supabase)      |
| Authentication  | JWT with bcrypt password hashing     |
| AI Integration  | Google Gemini API                    |
| External API    | GitHub REST API                      |
| Charts          | Recharts (or Chart.js)               |
| Design          | Figma, Google Stitch AI              |
| API Testing     | Postman                              |
| Version Control | Git and GitHub                       |
| Deployment      | Netlify (frontend), Render (backend) |

This is the plan for now. We may adjust small parts as we go through the design weeks.

---

## 6. Expected Outcome

### What users will gain

- Instructors will see submissions, commit activity, AI summaries and grades in one place instead of checking four or five different places.
- Students will know early how their team is doing and will get faster and more consistent feedback.
- Departments can reuse the system for other sections, courses and semesters, because milestones, rubrics and naming rules are configurable.

### How it improves the current workflow

- Manual checking of file names and links is replaced by automatic checks.
- Free-riding and last-minute pushes can be seen during the semester, not only at the viva.
- Feedback drafting is faster, and the same rubric is applied to every team.

### Expected benefits of AI integration

- Understands what the commit messages mean, not just how many there are.
- Saves the instructor's time while keeping the final decision with the instructor.
- Gives evidence-based findings that can be checked.

---

## 7. Initial Project Planning

| Week | Plan                                                  | Main Owner        |
| ---- | ----------------------------------------------------- | ----------------- |
| 2    | Proposal, GitHub repository, README, team sheet       | Everyone          |
| 3    | SRS document and first diagrams                       | Leader, Backend   |
| 4    | Architecture, ER diagram, API list                    | Database, Backend |
| 5    | Figma wireframes (login, dashboards, submission page) | Frontend          |
| 6    | Backend: authentication, database, main APIs          | Backend, Database |
| 7    | Frontend pages and API connection                     | Frontend          |
| 8    | GitHub API and Gemini integration                     | Leader (AI Lead)  |
| 9    | Complete features, dashboard charts, fix major bugs   | Everyone          |
| 10   | Testing: functional, API, UI and AI response checks   | Everyone          |
| 11   | Deployment on Vercel and Render                       | Leader, Backend   |
| 12   | Final report, user manual, API documentation          | Everyone          |
| 13   | Presentation slides and demo practice                 | Everyone          |
| 14   | Final presentation and viva                           | Everyone          |

### Team roles

- **Md Shahajalal Mahmud** – Team Leader and AI Integration Lead: coordination, weekly reports, Gemini prompts and AI workflow.
- **Preota Saha** – Frontend Developer: React pages, dashboards, charts.
- **Md Munna Sardar** – Database Manager: database design, queries, data validation.
- **Hazera Islam Mim** – Backend Developer: APIs, authentication, GitHub API connection.

### Possible risks and how we will handle them

- **GitHub API rate limits:** we will save fetched commit data in our database and refresh it only when needed.
- **AI output is not always correct:** we will use clear prompts, structured output, and require instructor approval.
- **Scope is too big:** we will finish the core features first and treat the extra features as optional.
- **Free hosting is slow:** we will keep backup screenshots and a demo video for the presentation.

To keep development continuous, every member will push to GitHub every week with clear commit messages, using separate branches (`main`, `development`, `frontend`, `backend`, `ai-integration`).
