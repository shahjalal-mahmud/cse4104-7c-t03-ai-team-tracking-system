# 🔍 CourseLens: AI-Powered Academic Coursework & Team Contribution Tracking System

[![Course: CSE 4104](<https://img.shields.io/badge/Course-CSE%204104%20(Software%20Development%20III)-blue?style=flat-square>)]()
[![Team: CSE4104-7C-T03](https://img.shields.io/badge/Team-CSE4104--7C--T03-purple?style=flat-square)]()
[![Status: Initial Proposal](https://img.shields.io/badge/Status-Week%2001%20Submission-orange?style=flat-square)]()

CourseLens is an automated academic management and AI-audit platform designed to streamline project-based courses. It bridges the gap between scattered student submissions, GitHub repository histories, and instructor grading workflows by providing real-time contribution metrics and AI-driven rubric feedback.

---

## 👥 Team Details (Team 03, Section 7C)

| Role                                  | Name                 | Student ID    | GitHub / Contact                   |
| :------------------------------------ | :------------------- | :------------ | :--------------------------------- |
| **Team Leader & AI Integration Lead** | Md Shahajalal Mahmud | `11230321539` | [@shahajalal](https://github.com/) |
| **Frontend Developer**                | Preota Saha          | `11230321538` | —                                  |
| **Database Manager**                  | Md Munna Sardar      | `11230321540` | —                                  |
| **Backend Developer**                 | Hazera Islam Mim     | `11230321542` | —                                  |

---

## 🚀 The Problem We Are Solving

In multi-week, team-based software engineering courses:

- **Scattered Submissions:** Deliverables are fragmented across learning management systems, GitHub, deployment links, and chat applications.
- **The "Free-Rider" Problem:** Inactive team members and last-minute bulk code pushes often remain hidden until the final viva.
- **Manual Overhead:** Instructors spend excessive time checking naming conventions, verifying live links, reviewing git commit logs, and writing repetitive feedback across 14 weeks.

---

## 🛠️ Key Features

- **Role-Based Access Control (RBAC):** Dedicated portals for Students, Teams, and Instructors with configurable team naming conventions.
- **Weekly Milestone Engine:** Structured submission pipelines enforcing link validation and required file structures.
- **GitHub API Integration:** Automatically fetches commit frequencies, lines changed, and per-student contributions.
- **AI Audit & Review Engine (Google Gemini API):** Parses commit logs and assignment rubrics to flag inactive members, summarize progress, and draft preliminary rubric-based feedback.
- **Human-in-the-Loop Grading:** Instructors retain full authority to review, adjust, and approve AI-generated feedback and grades with a single click.
- **Privacy-Aware AI Pipeline:** Commit metadata is sent securely to the AI with student identifiers replaced by anonymous tokens.

---

## 🏗️ Initial Technology Stack

| Layer              | Technology Choice                                              |
| :----------------- | :------------------------------------------------------------- |
| **Frontend**       | React.js, Tailwind CSS                                         |
| **Backend**        | Node.js, Express.js                                            |
| **Database**       | PostgreSQL                                                     |
| **AI Integration** | Google Gemini API                                              |
| **DevOps & Tools** | GitHub (API & Version Control), Figma, Postman, Vercel, Render |

---

## 📂 Project Structure (Planned)

```text
courselens/
├── client/                 # React.js frontend application
├── server/                 # Node.js & Express.js backend API
├── database/               # PostgreSQL schemas and migration scripts
└── docs/                   # Course submissions, diagrams, and project documentation
```
