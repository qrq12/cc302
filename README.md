

# 📝 ToDo SaaS App — Full DevOps Project

A full-stack **Flask-based ToDo application** developed using professional software engineering practices including Git branching workflows, feature-based development, Docker containerization, automated testing, and CI/CD pipelines with GitHub Actions.

---

## 🚀 Project Overview

This project demonstrates:

* Scalable feature development using GitHub branches
* Clean integration via Pull Requests
* Containerization with Docker + semantic versioning
* Automated testing with pytest
* CI/CD automation using GitHub Actions

---

## 🧠 Learning Outcomes

Through this project, I learned how to:

* Use **Git workflows** (main → dev → feature branches)
* Develop features independently and merge safely using PRs
* Resolve merge conflicts and maintain clean commit history
* Build and push Docker images using **Semantic Versioning**
* Automate testing and deployment using **GitHub Actions**
* Write reliable backend tests using **pytest + Flask test client**

---

## 🌿 Git Workflow

### Branch Structure

main → Production-ready code
dev → Integration branch
feature/* → Individual features

### Implemented Feature Branches

* `feature/task-descriptions-and-metadata`
* `feature/search-tasks`
* `feature/filters-and-sorting`

### Workflow Rules

* ❌ No direct commits to `main`
* ❌ No direct feature commits to `dev`
* ✅ All features merged via Pull Requests
* ✅ Each PR includes description and testing evidence

---

## ✨ Features Implemented

### 1. Task Descriptions & Metadata

* Added:

  * Description
  * Priority
  * Due date
* Updated database schema (migration)
* UI updated to display task details

---

### 2. Search Tasks

* Search by title and description
* Query parameter: `q`
* Integrated search bar in UI

---

### 3. Filters & Sorting

* Filter by:

  * Status
  * Priority
  * Due date (overdue, today, this week)
* Sort by:

  * Due date
  * Created date
  * Priority

---

## 🐳 Docker & Versioning

### Image Details

* Repository: `dockerhub_user/todo-saas`
* Versioning format: **MAJOR.MINOR.PATCH**

### Current Release

* `0.1.0` → First feature release
* `latest` → Points to latest stable version

### Commands Used

```bash
docker build -t dockerhub_user/todo-saas:0.1.0 .
docker push dockerhub_user/todo-saas:0.1.0

docker tag dockerhub_user/todo-saas:0.1.0 dockerhub_user/todo-saas:latest
docker push dockerhub_user/todo-saas:latest
```

---

## 🏷️ GitHub Release

* Tag: `v0.1.0`
* Includes:

  * Task metadata feature
  * Search functionality
  * Filtering & sorting

---

## ⚙️ CI/CD with GitHub Actions

### 🔁 Continuous Integration (CI)

Triggered on:

* Push to `main` or `dev`
* Pull Requests

Steps:

* Install dependencies
* Run linting (flake8)
* Run tests (pytest)

---

### 🚀 Continuous Delivery (CD)

Triggered on:

* GitHub Release publish

Steps:

* Build Docker image
* Extract version from tag (e.g., v0.1.0 → 0.1.0)
* Push image to DockerHub

---

### 🔐 Secrets Used

* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN`

---

## 🧪 Testing (pytest)

### Test Coverage

* ✅ Create Task
* ✅ Update Task
* ✅ Delete Task
* ✅ Read/Verify operations

### Example Test Flow

* Create a task
* Verify it exists
* Update task
* Verify changes
* Delete task
* Verify removal

### Run Tests

```bash
pytest -q
```

---

## 🛡️ Branch Protection (Quality Gates)

Enabled for:

* `dev`
* `main`

Rules:

* Require Pull Requests before merging
* Require CI checks to pass

### Workflow Demonstration

* ❌ Introduced failing test → CI failed → Merge blocked
* ✅ Fixed test → CI passed → Merge allowed

---

## 🔄 End-to-End Pipeline Flow

1. Develop feature in branch
2. Push → CI runs automatically
3. Open PR → CI validates code
4. Merge to `dev`
5. Merge `dev` → `main`
6. Create GitHub Release (`vX.X.X`)
7. CD pipeline builds & pushes Docker image

---

## 📁 Project Structure

```
.
├── app.py
├── requirements.txt
├── Dockerfile
├── tests/
│   └── test_crud.py
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
```

---

## 📌 Key Takeaways

* Branching keeps development **organized and safe**
* Pull Requests improve **code quality and collaboration**
* Docker ensures **consistent deployment environments**
* CI/CD automates repetitive work and reduces errors
* Testing guarantees **application reliability**

---

## 📎 Submission Contents

This repository includes:

* Feature branch PRs
* Dev → Main PR
* DockerHub image tags
* GitHub release
* CI/CD pipelines
* Automated tests

---

## 🧑‍💻 Author

Developed as part of a coursework assignment on:

* GitHub workflows
* DevOps practices
* Flask application development


## ▶️ How to Run the Application

### 🔧 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

---

### 🐍 2. Set Up Python Environment

```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

---

### 📦 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

### ▶️ 4. Run the Flask App

```bash
python app.py
```

Then open your browser at:

```
http://127.0.0.1:5000/
```

---

### 🧪 5. Run Tests

```bash
pytest -q
```

---

## 🐳 Run Using Docker (Recommended)

### 🔨 Build the Image

```bash
docker build -t todo-saas .
```

---

### ▶️ Run the Container

```bash
docker run -p 5000:5000 todo-saas
```

Then open:

```
http://localhost:5000/
```

---

### 📥 Pull from DockerHub (if already pushed)

```bash
docker pull <dockerhub_user>/todo-saas:latest
docker run -p 5000:5000 <dockerhub_user>/todo-saas:latest
```

---


