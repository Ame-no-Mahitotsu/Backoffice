---
ID: session_techlead_alessandro_001_deployment
Type: session
Role: tech-lead
Topic: Deployment strategy and TDD testing approach — Docker Compose, DigitalOcean, pytest
Date: 2026-03-12
Participants: Owner, Alessandro
Ticket: —
Sprint: —
Status: closed
Project: site
Outputs: DigitalOcean hosting decision, Docker Compose deployment plan, TDD testing strategy, constitution entries
---

# Session: Alessandro (Tech Lead) — Deployment & Testing
**Date:** 2026-03-12
**Status:** Approved by Project Owner

---

## Deployment Setup

```
docker-compose.yml
├── nginx          (reverse proxy — port 80/443)
├── django         (gunicorn — port 8000 internal)
├── nextjs         (node — port 3000 internal)
└── postgres       (port 5432 internal only)
```

- One docker-compose.yml runs entire stack
- Secrets via .env file on server — never in repository
- Daily PostgreSQL dumps via cron, stored outside container
- Images backed up separately via rclone to cloud storage
- Manual deployment for v1 — CI/CD added when test suite has sufficient coverage

## Hosting Decision
**DigitalOcean, Frankfurt region**
- 2-core / 4GB RAM droplet — ~€18/month
- Network-level cloud firewall (separate from VPS)
- EU jurisdiction — GDPR compliant
- Most robust security tooling of options considered

## Testing Strategy: TDD
Test-Driven Development adopted. Tests written before implementation. No feature is done until tests pass. No exceptions.

### Django (pytest + pytest-django)
- Every model: written test-first
- Every API endpoint: test defines contract before view exists
- Every business logic function: test before implementation
- Auth flows: login, fail, lockout, TOTP, session expiry
- Order flow: each step tested in isolation and end-to-end
- Localisation: every content endpoint tested for all three languages

### Next.js (Jest + React Testing Library)
- Every component tested before built
- User journey tests: B2C browse → cart → checkout
- Language switcher behaviour
- No visual regression tests in v1

### Integration
- API contract tests — Django and Next.js agree on response shapes
- Run against test database, never production

### Discipline
- Minimum test that proves the behaviour — not exhaustive edge cases upfront
- No "I'll add tests later"
