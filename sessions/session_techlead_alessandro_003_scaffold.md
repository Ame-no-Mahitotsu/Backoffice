---
ID: session_techlead_alessandro_003_scaffold
Type: session
Role: tech-lead
Topic: Django scaffold brief — full deliverable spec for ISS-004
Date: 2026-03-13
Participants: Owner, Alessandro
Ticket: ISS-004
Sprint: 0
Status: closed
Project: site
Outputs: Django scaffold deliverable spec, docker-compose.override.yml template, requirements.txt, pyproject.toml config, health endpoint TDD brief
---

# Session: Alessandro (Tech Lead) — Django Scaffold Brief
**Date:** 2026-03-13
**Status:** Approved by Project Owner — ready to execute

---

## Context

This is Alessandro's first implementation task. The goal is **TDD infrastructure before any models** — a working Django project in Docker with a green test suite. No business logic, no data models (except the health endpoint). Everything that comes after builds on this.

Decisions backing this brief: WRK-001 (dev environment), ARQ-001 (stack), `ai-standards.md` (coding standards, test standards, branch model).

---

## Repo Location

All code goes in `SitoPresepi2-src/` — the working clone of `SitoPresepi2/remote.git`.

```
c:\temp\ClaudeProjects\
├── SitoPresepi2\           ← project management only (constitution, memory, AI standards)
│   └── remote.git\         ← bare git repo — the authoritative remote
└── SitoPresepi2-src\       ← working clone — ALL code goes here
```

Work on branch: `feature/django-scaffold` branched from `develop`.

---

## Deliverables

### 1. `.gitignore` (first commit, before anything else)

```
# Secrets — never committed
.env
.env.local

# Dev environment override — local only
docker-compose.override.yml

# Python
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
*.egg-info/
dist/
build/
.eggs/

# Test / lint artefacts
.pytest_cache/
.ruff_cache/
.mypy_cache/
htmlcov/
.coverage
coverage.xml

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
```

Commit: `chore: add .gitignore`

---

### 2. `docker-compose.override.yml` (gitignored — created locally, never committed)

```yaml
# docker-compose.override.yml
# LOCAL DEV ONLY — gitignored, never committed.
# Overrides docker-compose.yml for local development.
# All ports bound to 127.0.0.1 — never exposed on LAN (WRK-D2).

services:

  django:
    command: python manage.py runserver 0.0.0.0:8000
    # runserver replaces gunicorn in dev. Hot-reload included.
    # 0.0.0.0 inside container is required for Docker port forwarding to work.
    # The host-side binding (127.0.0.1 below) is what prevents LAN exposure.
    environment:
      - DJANGO_DEBUG=True
      - DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
      - SESSION_COOKIE_SECURE=False
      - CSRF_COOKIE_SECURE=False
      - ADMIN_SESSION_TIMEOUT_OWNER=86400
      - ADMIN_SESSION_TIMEOUT_ARTISAN=86400
    ports:
      - "127.0.0.1:8000:8000"   # localhost only — not exposed on LAN
    volumes:
      - ./django:/app            # live source mount — edits reflected immediately

  nextjs:
    command: npm run dev
    environment:
      - NODE_ENV=development
      - NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
      - NEXT_INTERNAL_API_URL=http://django:8000/api/v1
    ports:
      - "127.0.0.1:3000:3000"   # localhost only
    volumes:
      - ./nextjs:/app            # live source mount

  nginx:
    profiles:
      - production               # excluded from default `docker compose up`

  test:
    build:
      context: ./django
      dockerfile: Dockerfile
    command: pytest --tb=short -q
    environment:
      - DJANGO_DEBUG=True
      - DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
      - DATABASE_URL=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}_test
      # Separate test database — pytest-django creates/destroys it per run.
      # Never shares state with the dev database.
    volumes:
      - ./django:/app
    networks:
      - internal
    depends_on:
      postgres:
        condition: service_healthy
    profiles:
      - test                     # only runs when explicitly invoked

# Run tests with:
#   docker compose --profile test run --rm test
```

---

### 3. `.env.example` — add dev section

Append to the existing `.env.example` (Dominick's file):

```bash
# =============================================================================
# DEV ONLY — never set these values in production
# =============================================================================
# Copy this file to .env and fill in values. .env is gitignored.

# Django dev overrides
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1

# Cookie security — must be False in dev (HTTP only, no TLS)
# Production always sets these to True via Django settings
SESSION_COOKIE_SECURE=False
CSRF_COOKIE_SECURE=False

# Session timeouts relaxed for dev (24h)
ADMIN_SESSION_TIMEOUT_OWNER=86400
ADMIN_SESSION_TIMEOUT_ARTISAN=86400

# Test database name (append _test to POSTGRES_DB value)
# Used by the test Docker service only
```

---

### 4. Django project scaffold

Location: `SitoPresepi2-src/django/`

```
django/
├── Dockerfile                  # FROM python:3.12-slim, non-root USER, gunicorn
├── requirements.txt            # pinned versions — see below
├── pyproject.toml              # pytest + Black + isort + Ruff config
├── conftest.py                 # PROJECT-LEVEL: pytest settings, Django setup
├── manage.py
└── sitopresepi/                # Django project package
    ├── __init__.py
    ├── settings/
    │   ├── __init__.py
    │   ├── base.py             # shared settings
    │   ├── development.py      # imports base, DEBUG=True overrides
    │   └── production.py       # imports base, production hardening
    ├── urls.py                 # root URL conf
    ├── wsgi.py
    └── asgi.py
```

**Django apps to scaffold (empty, with test stubs):**

```
django/
├── catalogue/       # Product, Size, ProductImage
├── commerce/        # Order, OrderItem, Payment stub
├── customers/       # Customer stub
├── gallery/         # GalleryItem, TimeLapseItem
├── content/         # Page, NavigationItem
├── localisation/    # Language, TranslatedField
├── admin_app/       # Admin panel, AuditLog
└── api/             # Public REST endpoints (includes /api/v1/health/)
```

Each app follows the standard structure from `ai-standards.md`:
```
app_name/
├── __init__.py
├── models.py
├── services.py
├── serializers.py
├── views.py
├── urls.py
└── tests/
    ├── __init__.py
    ├── conftest.py         # app-level fixtures only
    ├── test_models.py      # placeholder — one passing stub test
    ├── test_services.py    # placeholder
    └── test_views.py       # placeholder
```

---

### 5. `requirements.txt` (pinned)

```
Django==5.0.*
djangorestframework==3.15.*
django-otp==1.5.*
psycopg[binary]==3.*
django-storages==1.14.*
gunicorn==22.*
django-cors-headers==4.*

# Dev / test
pytest==8.*
pytest-django==4.*
black==24.*
isort==5.*
ruff==0.4.*
pre-commit==3.*
```

Note in the session file any version pinned — per `ai-standards.md` ("no new package without noting it and its purpose").

---

### 6. `pyproject.toml`

```toml
[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "sitopresepi.settings.development"
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "--strict-markers -q"

[tool.black]
line-length = 88
target-version = ["py312"]

[tool.isort]
profile = "black"

[tool.ruff]
line-length = 88
target-version = "py312"
select = ["E", "F", "W", "I"]
```

---

### 7. `conftest.py` (project-level)

```python
# django/conftest.py
import django
import pytest
from django.conf import settings


@pytest.fixture(scope="session")
def django_db_setup():
    """
    Project-level DB fixture.
    pytest-django handles creation/teardown of the test database.
    This fixture exists as the canonical hook point — extend here, not in app conftest.py.
    """
    pass
```

---

### 8. Health endpoint (first real code — TDD)

**Test first** (`api/tests/test_views.py`):

```python
import pytest
from django.urls import reverse


@pytest.mark.django_db
def test_health_endpoint_returns_200(client):
    response = client.get(reverse("api:health"))
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

This test must be **red** before any implementation is written. Then implement the minimal view to make it green.

Implementation (`api/views.py`):
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def health(request):
    return Response({"status": "ok"})
```

The health endpoint must also verify DB connectivity (add a `SELECT 1` check) — required by the Docker Compose `healthcheck` in production.

---

### 9. Definition of Done for this task

- [ ] `.gitignore` committed first, before any other file
- [ ] `docker-compose.override.yml` exists locally (gitignored, never in diff)
- [ ] `docker compose up` starts Django + PostgreSQL (no Nginx, no Next.js needed yet)
- [ ] `docker compose --profile test run --rm test` exits 0 — all tests green
- [ ] Health endpoint: `curl http://localhost:8000/api/v1/health/` → `{"status": "ok"}`
- [ ] All apps scaffolded with placeholder test stubs (green)
- [ ] Pre-commit hooks installed and passing (`black`, `isort`, `ruff`)
- [ ] PR raised to `develop` with Tech Lead + Backend review

---

### 10. Commit sequence (suggested)

```
chore: add .gitignore
chore: scaffold Django project structure and app skeletons
chore: add pyproject.toml with pytest, black, isort, ruff config
chore: add project-level conftest.py and app-level test stubs
chore: add requirements.txt with pinned dependencies
chore: add Django Dockerfile (python:3.12-slim, non-root user)
chore: add .env.example dev section
test: add failing health endpoint test
feat(api): implement health endpoint — test green
chore: add pre-commit config
```

Each commit is small, reviewable, and follows Conventional Commits format.

---

## Packages Introduced (log per ai-standards.md)

| Package | Purpose |
|---|---|
| `Django 5.x` | Web framework — approved ARQ-001 |
| `djangorestframework 3.x` | REST API layer — approved ARQ-001 |
| `django-otp 1.x` | TOTP two-factor auth — approved BAR-001 |
| `psycopg[binary] 3.x` | PostgreSQL driver (psycopg3, async-capable) |
| `django-storages 1.x` | Filesystem abstraction for media/static — approved ARQ-001 |
| `gunicorn 22.x` | WSGI server for production — approved in docker-compose.yml |
| `django-cors-headers 4.x` | CORS enforcement — required by Dominick's security baseline |
| `pytest 8.x` | Test runner — approved ARQ-001 |
| `pytest-django 4.x` | Django integration for pytest |
| `black 24.x` | Python formatter — approved by Alessandro (ai-standards.md) |
| `isort 5.x` | Import sorter — approved by Alessandro |
| `ruff 0.4.x` | Linter — approved by Alessandro |
| `pre-commit 3.x` | Git hook runner — approved by Alessandro |
| `dj-database-url 2.x` | Parses `DATABASE_URL` env var into Django `DATABASES` dict — standard pattern, avoids manual DSN parsing |

---

*Brief prepared by Orchestrator from WRK-001 decisions. Approved by Owner 2026-03-13.*
