---
ID: session_security_dominick_001_baseline
Type: session
Role: security
Topic: Security baseline — 5 layers, RBAC, TOTP auth, session timeouts, audit log
Date: 2026-03-12
Participants: Owner, Dominick
Ticket: ISS-008, ISS-009, ISS-010
Sprint: —
Status: closed
Project: site
Outputs: security baseline approved, security_checklist.md, ISS-009 raised (image upload), ISS-010 raised (architecture §15)
---

# Session: Dominick (Security) — Security Baseline
**Date:** 2026-03-12
**Status:** Approved by Project Owner

---

## Security Baseline — Approved

### Layer 1 — Network Perimeter
- Nginx is the only public entry point
- Django and Next.js on Docker internal network — not publicly reachable
- SSL/TLS termination at Nginx — HTTP not allowed
- fail2ban on VPS for aggressive IP probing

### Layer 2 — Identity and Authentication
- Two completely separate identity domains: Admin and Customer (future)
- Admin: session-based auth, short session lifetime, login throttling
- Customer (future): JWT via SimpleJWT — separate from admin domain
- No shared tokens, sessions, or middleware between domains
- No email-based password reset in v1 — maintainer resets via Django management command

### Layer 3 — API Security
- Every endpoint: public read-only OR authenticated — no middle ground
- Rate limiting on all endpoints at Nginx level
- Contact form rate limited — spam vector if unprotected
- CORS locked to Next.js origin only
- Admin app calls Django directly, not through public API

### Layer 4 — Data
- PostgreSQL with dedicated minimum-privilege Django user
- No sensitive data in logs
- GDPR applies — Italy, UK, Germany — personal data in orders
- Privacy policy, data retention policy, cookie consent required before launch
- Fran (BA) to be involved in GDPR compliance items

### Layer 5 — Admin Interface
- Non-obvious URL path (e.g. `/gestione`)
- Two-factor authentication: username + password + TOTP (django-otp) — **required for all admin users including Gabriele**
- Session cookie: HttpOnly, Secure, SameSite=Strict
- Session inactivity timeouts (env-configurable):
  - `ADMIN_SESSION_TIMEOUT_OWNER=3600` (1h) — Django superuser (Owner)
  - `ADMIN_SESSION_TIMEOUT_ARTISAN=7200` (2h) — `artisan` Django Group (Gabriele)
  - Timeouts are inactivity-based — reset on every request; active sessions not interrupted
- Login throttling: 5 failed attempts → 15 min IP lock
- Audit log: all logins, logouts, failed attempts, content changes

### Layer 5b — Admin RBAC (BAR-001, 2026-03-13)
Two roles. No third role.

**Django superuser (Owner):**
- Full access to all admin functions
- Product management, user management, site configuration, orders, content

**`artisan` Django Group (Gabriele):**
- Upload images (gallery, time-lapse, product images)
- Edit localised text content (gallery captions, page text)
- Read-only access to orders and sales figures
- Cannot: manage products, manage users, manage languages, change site configuration
- Acts independently — no owner approval step required for permitted actions

### Password Handling
- Django native PBKDF2-SHA256 (or Argon2 — one-line config change)
- Unique salt per password — no plain text ever stored
- Django validators: minimum length, common password rejection, numeric-only rejection
- TOTP: 30-second window, secret key encrypted in database

### Admin Login Flow
1. Username + password → Django verifies hash
2. If correct → TOTP prompt
3. Six-digit code from authenticator app → Django verifies
4. Both pass → session cookie issued
5. Either fails → no session, attempt logged, throttle incremented

### Documentation Dominick Produced
1. ✅ Nginx configuration reference — delivered (nginx/nginx.conf)
2. ✅ Docker Compose setup — delivered (docker-compose.yml)
3. ✅ Security checklist — delivered 2026-03-13 (security_checklist.md, ISS-008 resolved)

### Follow-on tickets raised from checklist
- ISS-009: Image upload security (Alessandro) — open; acts when gallery/upload feature is built
- ISS-010: Architecture doc §15 + Contact model GDPR field (Patrick) — open; Sprint 1

### Owner Profile Note
Owner has corporate reverse proxy and IDM experience — Nginx configuration will be familiar territory.

---

## Open Items
- GDPR: privacy policy, data retention, cookie consent — involve Fran before launch
- Argon2 vs PBKDF2 — decision deferred, either acceptable
