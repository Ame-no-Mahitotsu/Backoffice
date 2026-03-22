# Meeting Minutes — WRK-001

ID: WRK-001
Type: working-session
Tags: dev-environment, docker, containerisation, test-runner, local-dev, security
Date: 2026-03-13
Chair: Orchestrator
Participants: Dominick, Patrick, Alessandro, Owner
Ticket: —
Sprint: —
Status: closed
Project: site

---

## Background

Following BAR-001 close, the owner asked about local development security (laptop exposure risk). Dominick flagged that the existing `docker-compose.yml` is production-only and has no dev environment definition. This session resolved the dev environment strategy.

---

## Decisions Taken

| Ref | Decision | Approved |
|---|---|---|
| WRK-D1 | Dev environment: fully containerised via `docker-compose.override.yml` (Option A). Nginx excluded in dev via Docker Compose `profiles: ["production"]`. | Yes |
| WRK-D2 | All dev ports bound to `127.0.0.1` only — never `0.0.0.0`. Mitigates LAN exposure on the owner's laptop. | Yes |
| WRK-D3 | Django runs `manage.py runserver` in dev (replaces gunicorn); Next.js runs `next dev` (replaces production start). Source code mounted as volumes for hot-reload. | Yes |
| WRK-D4 | PostgreSQL service unchanged between dev and prod. No override needed. | Yes |
| WRK-D5 | Dedicated `test` Docker service in override (profile-gated via `profiles: ["test"]`). Separate test database. Invoked via `docker compose --profile test run --rm test`. | Yes |
| WRK-D6 | `conftest.py` at two levels: project-level (`django/conftest.py`) for pytest settings and Django setup; app-level (`app_name/tests/conftest.py`) for app-specific fixtures only. Alessandro scaffolds project-level on day one. | Yes |
| WRK-D7 | `.env.example` extended with a clearly marked `# DEV ONLY` section: `DJANGO_DEBUG=True`, `DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1`, `SESSION_COOKIE_SECURE=False`, `CSRF_COOKIE_SECURE=False`, `ADMIN_SESSION_TIMEOUT_OWNER=86400`, `ADMIN_SESSION_TIMEOUT_ARTISAN=86400`. | Yes |
| WRK-D8 | `docker-compose.override.yml` lives in `SitoPresepi2-src/` (alongside `docker-compose.yml`). Gitignored there. `SitoPresepi2/` (project management repo) never holds runtime artefacts. | Yes |
| WRK-D9 | `.gitignore` scaffolded by Alessandro on day one. Must include at minimum: `.env`, `.env.local`, `docker-compose.override.yml`, `__pycache__/`, `*.pyc`, `.pytest_cache/`. | Yes |

---

## Rationale

- **Option A over Option B (hybrid)**: Production parity from day one. AI agents run `pytest` in the same container image and Python version as production. No "works on my machine" failures.
- **`127.0.0.1` binding**: Docker default `0.0.0.0` exposes ports on all interfaces including LAN. On a shared/public network this is a real exposure. Localhost binding eliminates it at zero cost.
- **Dedicated test service over `docker compose run --rm django pytest`**: `run --rm` spins a second container alongside the running dev server, both competing for the dev database. A profile-gated test service with its own database connection is cleaner and safer for continuous TDD use.
- **`SESSION_COOKIE_SECURE=False` in dev**: Production sets `Secure=True` on session cookies. Without `False` in dev (HTTP only, no TLS), the cookie is never sent and admin login silently fails. Forgetting this would cost Alessandro hours of debugging.

---

## Alessandro's Deliverables (from this session)

See `session_techlead_alessandro_003_scaffold.md` for full brief.

1. `SitoPresepi2-src/.gitignore` — created on first commit
2. `SitoPresepi2-src/docker-compose.override.yml` — dev override (gitignored)
3. `.env.example` — dev section added
4. Django project scaffold in `SitoPresepi2-src/django/`
5. `django/conftest.py` — project-level pytest config
6. Per-app `tests/conftest.py` stubs for each Django app
7. `pyproject.toml` — pytest + Black + isort + Ruff config
8. Smoke test: `test_health_endpoint_returns_200` passing inside Docker

---

*Minutes recorded by Orchestrator. Approved by Owner 2026-03-13.*
