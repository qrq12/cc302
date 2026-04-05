# 📝 ToDo SaaS — Full-Stack Task Management Application

> A production-grade, multi-user **Flask** task management platform built with professional DevOps practices: feature branching, Docker containerization, automated testing, and CI/CD pipelines via GitHub Actions.

---

## 📚 Table of Contents

1. [Project Overview](#-project-overview)
2. [Architecture](#-architecture)
3. [Features](#-features)
   - [Authentication System](#1-authentication-system)
   - [Task Management](#2-task-management)
   - [Subtask System](#3-subtask-system)
   - [Search, Filters & Sorting](#4-search-filters--sorting)
   - [Statistics Dashboard](#5-statistics-dashboard)
4. [API Reference](#-api-reference)
   - [Auth Endpoints](#auth-endpoints)
   - [Task Endpoints](#task-endpoints)
   - [Subtask Endpoints](#subtask-endpoints)
   - [Stats Endpoint](#stats-endpoint)
5. [Git Workflow](#-git-workflow)
   - [Branch Structure](#branch-structure)
   - [Feature Branches](#feature-branches)
   - [Workflow Rules](#workflow-rules)
6. [Docker & Versioning](#-docker--versioning)
   - [Image Details](#image-details)
   - [Build & Push](#build--push)
7. [CI/CD Pipeline](#-cicd-pipeline)
   - [Continuous Integration](#continuous-integration-ci)
   - [Continuous Delivery](#continuous-delivery-cd)
   - [Secrets Management](#secrets-management)
8. [Testing](#-testing)
   - [Test Coverage](#test-coverage)
   - [Running Tests](#running-tests)
9. [Branch Protection & Quality Gates](#-branch-protection--quality-gates)
10. [Project Structure](#-project-structure)
11. [Local Setup & Running](#-local-setup--running)
    - [Python Setup](#python-setup)
    - [Running with Docker](#running-with-docker)
    - [Environment Variables](#environment-variables)
12. [Data Model](#-data-model)
13. [Key Takeaways & Learning Outcomes](#-key-takeaways--learning-outcomes)
14. [Author](#-author)

---

## 🚀 Project Overview

This project is a **multi-user ToDo SaaS application** that demonstrates a complete, production-style software engineering workflow. It goes beyond a simple CRUD app to showcase:

| Pillar | What's Demonstrated |
|---|---|
| **Backend Development** | Flask REST API, session auth, per-user data isolation |
| **DevOps** | Git branching strategy, Docker, semantic versioning |
| **Automation** | CI/CD pipelines with GitHub Actions |
| **Quality Assurance** | Automated pytest suite, branch protection rules |
| **Deployment** | DockerHub image registry, GitHub Releases |

The application stores data in flat JSON files per user (no external database dependency), making it easy to run locally or in a container.

---

## 🏛 Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Client (Browser)                   │
└────────────────────────┬─────────────────────────────┘
                         │ HTTP
┌────────────────────────▼─────────────────────────────┐
│              Flask Application (app.py)               │
│                                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │
│  │  Auth Layer │  │ Task Routes  │  │ Stats Routes│  │
│  │  /api/login │  │ /api/tasks/* │  │  /api/stats │  │
│  │  /api/reg.. │  │              │  │             │  │
│  └──────┬──────┘  └──────┬───────┘  └──────┬──────┘  │
│         └────────────────┼─────────────────┘         │
│                          │                            │
│                 ┌────────▼────────┐                   │
│                 │  Session Store  │                   │
│                 │  (Flask Cookie) │                   │
│                 └────────┬────────┘                   │
└──────────────────────────┼───────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
┌────────▼───────┐ ┌───────▼──────┐ ┌───────▼──────┐
│  users.json    │ │data_<u>_     │ │data_<u>_     │
│  (all users)   │ │tasks.json    │ │subtasks.json │
└────────────────┘ └──────────────┘ └──────────────┘
```

**Key design decisions:**
- **Per-user file isolation**: Each user's tasks and subtasks are stored in separate JSON files (`data_<username>_tasks.json`), ensuring complete data separation without a database.
- **Session-based auth**: Flask server-side sessions with SHA-256 password hashing.
- **Stateless REST API**: All routes are protected by a `require_login()` guard that checks the session.

---

## ✨ Features

### 1. Authentication System

A full multi-user auth system is implemented from scratch with no third-party auth libraries.

- **Registration** (`POST /api/register`): Creates a new user with validation (min lengths, allowed characters). Passwords are SHA-256 hashed before storage. User is automatically logged in on success.
- **Login** (`POST /api/login`): Authenticates against hashed password. Sets a secure server-side session.
- **Logout** (`POST /api/logout`): Clears the session entirely.
- **Session check** (`GET /api/me`): Returns the currently logged-in username.
- **Login guard**: All task and stats routes return `401 Unauthorized` if no valid session is present.

**Validation rules enforced:**
- Username: minimum 3 characters, alphanumeric + `-_` only
- Password: minimum 4 characters
- No duplicate usernames

---

### 2. Task Management

Full CRUD operations for tasks, with a rich data model:

| Field | Type | Description |
|---|---|---|
| `id` | Integer | Auto-incremented unique ID |
| `title` | String | Task title |
| `description` | String | Optional detailed description |
| `category` | String | Grouping label (default: `General`) |
| `priority` | String | `High`, `Medium`, or `Low` |
| `due_date` | String | ISO date string |
| `completed` | Boolean | Completion status |
| `created_at` | String | ISO datetime, set on creation |
| `tags` | Array | List of string tags |

A dedicated **toggle endpoint** (`PUT /api/tasks/<id>/toggle`) flips `completed` without requiring a full task payload, making UI interactions simpler.

---

### 3. Subtask System

Tasks can have nested subtasks, supporting:

- **Create, update, delete** subtasks independently of their parent task
- **Drag-and-drop reordering** via `PUT /api/tasks/<id>/subtasks/reorder` — accepts an ordered list of subtask IDs and updates the `order` field on each
- **Cascade delete**: Deleting a parent task automatically removes all its subtasks
- **Ordered retrieval**: Subtasks are always returned sorted by their `order` field

---

### 4. Search, Filters & Sorting

- **Search**: Query tasks by title and description using the `q` query parameter
- **Filter by status**: Completed / Pending
- **Filter by priority**: High / Medium / Low
- **Filter by due date**: Overdue, Today, This Week
- **Sort by**: Due date, Created date, Priority

---

### 5. Statistics Dashboard

The `GET /api/stats` endpoint returns a live summary for the current user:

```json
{
  "total": 12,
  "completed": 5,
  "pending": 7,
  "priority_counts": {
    "High": 2,
    "Medium": 3,
    "Low": 2
  }
}
```

`priority_counts` only counts **incomplete** tasks per priority level, making it useful for a live work-queue dashboard.

---

## 📡 API Reference

All endpoints are prefixed with `/api`. All task/subtask/stats routes require an active session (401 if not logged in).

### Auth Endpoints

| Method | Endpoint | Description | Auth Required |
|---|---|---|---|
| `POST` | `/api/register` | Register new user | No |
| `POST` | `/api/login` | Login | No |
| `POST` | `/api/logout` | Logout | No |
| `GET` | `/api/me` | Get current user | Yes |

### Task Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/tasks` | List all tasks for current user |
| `POST` | `/api/tasks` | Create a new task |
| `PUT` | `/api/tasks/<id>` | Update a task |
| `DELETE` | `/api/tasks/<id>` | Delete task (and its subtasks) |
| `PUT` | `/api/tasks/<id>/toggle` | Toggle completed status |

### Subtask Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/tasks/<id>/subtasks` | List subtasks (ordered) |
| `POST` | `/api/tasks/<id>/subtasks` | Add a subtask |
| `PUT` | `/api/subtasks/<id>` | Update a subtask |
| `DELETE` | `/api/subtasks/<id>` | Delete a subtask |
| `PUT` | `/api/tasks/<id>/subtasks/reorder` | Reorder subtasks |

**Reorder request body:**
```json
{ "order": [3, 1, 4, 2] }
```
Pass an array of subtask IDs in the desired display order.

### Stats Endpoint

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/stats` | Get task statistics for current user |

---

## 🌿 Git Workflow

### Branch Structure

```
main          ← Production-ready, protected
  └── dev     ← Integration branch, protected
        ├── feature/task-descriptions-and-metadata
        ├── feature/search-tasks
        └── feature/filters-and-sorting
```

### Feature Branches

| Branch | Description |
|---|---|
| `feature/task-descriptions-and-metadata` | Added `description`, `priority`, `due_date`, `tags` fields + DB migration + UI updates |
| `feature/search-tasks` | Search by title/description using `?q=` query param + search bar UI |
| `feature/filters-and-sorting` | Filter by status/priority/due date; sort by due date, created date, or priority |

### Workflow Rules

- ❌ No direct commits to `main`
- ❌ No direct feature commits to `dev`
- ✅ All features merged via Pull Requests with description and test evidence
- ✅ CI must pass before any PR can be merged
- ✅ Feature → dev → main, never feature → main directly

---

## 🐳 Docker & Versioning

### Image Details

- **Registry**: DockerHub — `qrq12/todo-saas`
- **Versioning**: Semantic Versioning (`MAJOR.MINOR.PATCH`)
  - `MAJOR`: Breaking changes
  - `MINOR`: New features, backwards-compatible
  - `PATCH`: Bug fixes

### Current Releases

| Tag | Description |
|---|---|
| `0.1.0` | First feature release (metadata + search + filters) |
| `latest` | Always points to the most recent stable version |

### Build & Push

```bash
# Build with version tag
docker build -t qrq12/todo-saas:0.1.0 .

# Push versioned image
docker push qrq12/todo-saas:0.1.0

# Tag and push as latest
docker tag qrq12/todo-saas:0.1.0 qrq12/todo-saas:latest
docker push qrq12/todo-saas:latest
```

The CD pipeline handles these steps automatically on every GitHub Release — no manual pushing needed for production releases.

---

## ⚙️ CI/CD Pipeline

### Continuous Integration (CI)

**File:** `.github/workflows/ci.yml`

**Triggers:**
- Push to `main` or `dev`
- Any Pull Request

**Steps:**
1. Checkout code
2. Set up Python
3. Install dependencies (`pip install -r requirements.txt`)
4. Run linter (`flake8`)
5. Run test suite (`pytest`)

If any step fails, the PR is blocked from merging.

### Continuous Delivery (CD)

**File:** `.github/workflows/cd.yml`

**Trigger:** GitHub Release published

**Steps:**
1. Checkout code
2. Extract version from release tag (e.g., `v0.1.0` → `0.1.0`)
3. Log in to DockerHub using stored secrets
4. Build Docker image with extracted version tag
5. Push versioned image to DockerHub
6. Tag and push as `latest`

### Secrets Management

Stored as GitHub Actions repository secrets — never hardcoded in workflow files.

| Secret | Purpose |
|---|---|
| `DOCKERHUB_USERNAME` | DockerHub account login |
| `DOCKERHUB_TOKEN` | DockerHub access token (not password) |

---

## 🧪 Testing

Tests are written using **pytest** with Flask's built-in test client, meaning tests run against the actual app logic without spinning up a real server.

### Test Coverage

| Test | What It Verifies |
|---|---|
| `test_create_task` | Task is created and returned with correct fields |
| `test_read_task` | Created task appears in the task list |
| `test_update_task` | Updated fields are persisted correctly |
| `test_delete_task` | Task is removed and no longer retrievable |
| `test_toggle_task` | `completed` flag flips on each toggle call |

### Test Flow (CRUD Lifecycle)

```
Create task → Assert exists → Update task → Assert changes → Delete task → Assert removed
```

### Running Tests

```bash
# Run all tests (quiet output)
pytest -q

# Verbose output
pytest -v

# Run a specific test file
pytest tests/test_crud.py
```

---

## 🛡️ Branch Protection & Quality Gates

Branch protection is enabled on both `dev` and `main`.

**Rules enforced:**
- Pull Request required before merging (no direct pushes)
- All CI status checks must pass before merge is allowed
- At least one approving review required (recommended for team use)

**Live demonstration of gates working:**

| Scenario | Result |
|---|---|
| Introduced a failing test → pushed to feature branch | ❌ CI failed → PR blocked from merging to dev |
| Fixed the test → pushed again | ✅ CI passed → PR approved and merged |
| Merged dev → main | ✅ Full pipeline ran cleanly |

---

## 📁 Project Structure

```
.
├── app.py                    # Main Flask application — all routes and logic
├── requirements.txt          # Python dependencies
├── Dockerfile                # Container build instructions
├── users.json                # Persisted user accounts (created at runtime)
├── data_<user>_tasks.json    # Per-user task storage (created at runtime)
├── data_<user>_subtasks.json # Per-user subtask storage (created at runtime)
├── templates/
│   ├── index.html            # Main app UI (requires login)
│   └── login.html            # Login / register page
├── tests/
│   └── test_crud.py          # pytest test suite
└── .github/
    └── workflows/
        ├── ci.yml            # Continuous Integration pipeline
        └── cd.yml            # Continuous Delivery pipeline
```

---

## 🖥️ Local Setup & Running

### Python Setup

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# 2. Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # macOS/Linux
# venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the app
python app.py
```

Then open: [http://127.0.0.1:5000](http://127.0.0.1:5000)

### Running with Docker

```bash
# Option A: Build locally
docker build -t todo-saas .
docker run -p 5000:5000 todo-saas

# Option B: Pull from DockerHub
docker pull qrq12/todo-saas:latest
docker run -p 5000:5000 qrq12/todo-saas:latest
```

Then open: [http://localhost:5000](http://localhost:5000)

### Environment Variables

| Variable | Default | Description |
|---|---|---|
| `SECRET_KEY` | Random hex (generated at startup) | Flask session secret. **Set this in production** — a random key means all sessions invalidate on every restart. |
| `PORT` | `5000` | Port Flask listens on |

To set a persistent secret key:
```bash
export SECRET_KEY="your-long-random-secret-here"
python app.py
```

Or in Docker:
```bash
docker run -p 5000:5000 -e SECRET_KEY="your-secret" qrq12/todo-saas:latest
```

---

## 🗃️ Data Model

### User (`users.json`)

```json
{
  "alice": {
    "password": "<sha256-hex>",
    "created_at": "2024-01-15T10:30:00.000000"
  }
}
```

### Task (`data_alice_tasks.json`)

```json
[
  {
    "id": 1,
    "title": "Build CI pipeline",
    "description": "Set up GitHub Actions for automated testing",
    "category": "DevOps",
    "priority": "High",
    "due_date": "2024-02-01",
    "completed": false,
    "created_at": "2024-01-15T10:31:00.000000",
    "tags": ["github", "automation"]
  }
]
```

### Subtask (`data_alice_subtasks.json`)

```json
[
  {
    "id": 1,
    "task_id": 1,
    "title": "Write ci.yml",
    "is_done": true,
    "order": 0
  }
]
```

---

## 📌 Key Takeaways & Learning Outcomes

Through this project, the following real-world engineering skills were practised end-to-end:

**Git & Collaboration**
- Feature branch strategy keeps development organized and safe
- Pull Requests enforce code review and prevent broken code from reaching main
- Resolving merge conflicts and maintaining a clean commit history

**Backend Development**
- Designing a RESTful API with proper HTTP status codes
- Implementing multi-user data isolation without a database
- Session-based authentication with secure password hashing

**DevOps & Infrastructure**
- Docker ensures identical environments across dev, CI, and production
- Semantic versioning gives clear, human-readable release history
- GitHub Releases act as deployment triggers for the CD pipeline

**Quality Engineering**
- Automated tests catch regressions before they reach main
- Branch protection rules enforce quality gates — CI isn't optional
- Linting enforces code style consistency across the codebase

---

## 🧑‍💻 Author

Developed as part of a coursework assignment covering:

- GitHub workflows and branching strategies
- Flask backend development
- DevOps practices (Docker, CI/CD, semantic versioning)
- Professional software engineering process

---

*For issues or questions, open a GitHub Issue on this repository.*
