
#  IT Ticketing System: Secure, Containerized, and Scalable

## Table of Contents
- [Overview](#overview)
- [Goals & Motivation](#goals--motivation)
- [Architecture & Design Principles](#architecture--design-principles)
- [Tech Stack](#tech-stack)
- [Folder Structure](#folder-structure)
- [Security & Best Practices](#security--best-practices)
- [Testing Strategy](#testing-strategy)
- [Docker Setup](#docker-setup)
- [CI/CD — GitHub Actions](#cicd--github-actions)
- [Key Learning Outcomes](#key-learning-outcomes)
- [Author](#author)


## Overview
This project is a **secure IT ticketing system** built with **FastAPI**, **PostgreSQL**, and **Docker**, designed to demonstrate professional **software engineering** and **cybersecurity** practices.

The system enables users to create, manage, and resolve IT support tickets with **role-based access control (RBAC)**, **audit logging**, and **secure authentication**.  
It follows clean architectural principles, uses modern DevOps tooling, and is ready for CI/CD deployment.

This project is part of my personal learning path as a **Computing Science student at the University of Groningen**, exploring the intersection of **software engineering** and **cybersecurity** through hands-on, production-style systems.

---

##  Goals & Motivation
I’m building this project to:
- Apply **software engineering best practices** — clean architecture, testing, CI/CD, Docker.
- Explore **secure backend development** (auth, RBAC, SQL injection prevention).
- Bridge **cybersecurity and engineering** through practical implementation.
- Gain hands-on experience with a real-world, maintainable codebase.



---

##  Architecture & Design Principles

### System Overview
This system is designed around layered architecture for decoupling and testability:

```
Request → Router → Service → Repository → Database
↓
Security
↓
Audit Logging
```

**Layers**
- **API Layer:** Request/response handling via FastAPI routers, input validation with Pydantic.
- **Service Layer:** Business logic, orchestration, and transaction boundaries.
- **Repository Layer:** Database access via SQLAlchemy ORM (no raw SQL).
- **Core Utilities:** Security, authentication, and configuration management.
- **Tests:** Unit and integration tests for full coverage.

**Key Design Practices**
- Dependency Injection via FastAPI `Depends()`.
- Repository Pattern for decoupling business logic from data access.
- RBAC for role enforcement (Admin, Technician, Requester).
- JWT-based authentication with token expiration.
- Parameterized queries to eliminate SQL injection risk.
- 12-Factor App principles (configuration, environment separation).
- Clean, testable architecture and consistent code style.

### RBAC and Audit Logging Design
The project implements **fine-grained Role-Based Access Control (RBAC)** at the service layer.  
Each role (Admin, Technician, Requester) is granted only the actions relevant to their context.  
- **Admins** can view and manage all tickets, assign technicians, and modify roles.  
- **Technicians** can access and update tickets assigned to them.  
- **Requesters** can only view or update their own tickets.  

All critical user actions: such as login, role changes, and ticket modifications: are tracked through an **Audit Logging system**.  
The audit log records:
- The authenticated user performing the action  
- The type of action (create, update, delete)  
- The target resource (ticket ID, user ID)  
- Timestamp and source IP  

This ensures full traceability of actions and supports both debugging and security monitoring.


---

## Tech Stack

| Layer | Technology |
|-------|-------------|
| Framework | FastAPI |
| Language | Python 3.11 |
| Database | PostgreSQL 15 |
| ORM | SQLAlchemy + Alembic |
| Auth | JWT + Passlib (bcrypt) |
| Containerization | Docker, Docker Compose |
| Testing | Pytest, HTTPX, pytest-asyncio |
| CI/CD | GitHub Actions |
| Code Quality | Black, Ruff, Mypy, Bandit, Safety |

---

##  Folder Structure
```plaintext
ticketing/
├─ app/
│  ├─ api/v1/routers/        # Routers (tickets, auth, users)
│  ├─ core/                  # Security, events, settings
│  ├─ db/                    # Session, migrations, base
│  ├─ models/                # SQLAlchemy models
│  ├─ repositories/          # Data access layer
│  ├─ schemas/               # Pydantic schemas
│  ├─ services/              # Business logic
│  ├─ tests/                 # Unit & integration tests
│  └─ main.py                # App entrypoint
├─ docker/
│  ├─ Dockerfile
│  └─ docker-compose.yml
├─ .github/workflows/ci.yml  # Continuous Integration
├─ requirements.txt
└─ README.md
```
## Security & Best Practices

This system is built with security in mind from day one:

-  **No raw SQL** — only parameterized SQLAlchemy ORM queries.  
-  **Passwords** securely hashed with bcrypt via Passlib.  
-  **JWT authentication** with signature validation and expiry.  
-  **RBAC** at service layer (not only UI).  
-  **Input validation** via Pydantic (type & semantic).  
-  **Audit logging** for privileged actions.  
-  **Database migrations** handled through Alembic for version control and schema integrity.
-  **Static analysis** with Bandit in CI/CD.  
-  **Dependency scanning** with Safety.  
-  **Configuration via environment variables**, never hardcoded.  
-  **Automated tests** to prevent regression & ensure safe refactoring.

---

## Testing Strategy

| Type | Description |
|------|--------------|
| **Unit Tests** | Validate services and repositories using mocks |
| **Integration Tests** | Run full API tests against a test PostgreSQL instance |
| **Security Tests** | Verify authentication, RBAC, and SQL injection safety |
| **Static Analysis** | Bandit + Ruff + Mypy checks in CI |

Run all tests:
```bash pytest -v```


## Docker Setup
Prerequisites: Docker & Docker Compose

Run locally:
docker-compose -f docker/docker-compose.yml up --build


Then open the interactive API docs at: http://localhost:8000/docs

## Local Development (without Docker)
pip install -r requirements.txt
uvicorn app.main:app --reload

## CI/CD — GitHub Actions

A complete CI/CD workflow runs on every push and pull request.

.github/workflows/ci.yml

```
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-test-lint:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: ticketing_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready -U postgres"
          --health-interval=5s
          --health-timeout=5s
          --health-retries=5

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint and format check
        run: |
          black --check .
          ruff check .
          mypy app/

      - name: Security checks
        run: |
          bandit -r app/ -ll
          safety check --full-report || true

      - name: Run tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/ticketing_test
        run: pytest -v

```
## What this pipeline does

Installs dependencies.

Runs code formatting and type checks.

Executes security scans (Bandit + Safety).

Spins up a PostgreSQL container.

Runs all Pytest unit and integration tests.

Future enhancements:

Add Docker image build & push to container registry.


## Key Learning Outcomes

By completing this project, I will demonstrate:

Strong grasp of software architecture and design patterns.

Secure coding principles and threat modeling awareness.

CI/CD fluency with GitHub Actions and DevSecOps practices.

The ability to design, implement, test, and deploy secure applications.



# Author

[Sorana Gavril]
3rd-year Computing Science student — University of Groningen 
Exploring secure software engineering and DevSecOps.

GitHub: [Sorana2](https://github.com/Sorana2)
