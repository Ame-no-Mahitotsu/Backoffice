---
ID: ARQ-HIST-002
Type: architecture-review
Tags: stack-decision, frontend-strategy, drf-nextjs, monolith, replaceability, supersedes-wagtail
Date: 2026-03-12
Chair: Orchestrator
Participants: Patrick, Alessandro, Lota, Ash, Fran, Dominick, Owner
Ticket: —
Sprint: —
Status: closed
Project: site
---

# Meeting: Full Team — Architecture & Frontend Strategy
**Date:** 2026-03-12
**Participants:** Patrick (Architect), Alessandro (Tech Lead), Lota (UX/UI), Ash (Frontend), Fran (BA), Dominick (Security)
**Outcome:** Recommendation reached — pending owner approval per individual discussions

---

## Agenda
1. Brief Lota and Ash on stack decision (Django + Wagtail)
2. Debate frontend strategy — Wagtail templates vs headless vs hybrid
3. Each technical member raises 3 concerns + alternatives
4. Owner flags: replaceability of parts + no monolith
5. Three paths defined and debated
6. Dominick joins — security assessment
7. Final recommendation synthesised

---

## Key Discussion Points

### Frontend Strategy
- Patrick proposed the question: Wagtail templates vs separate frontend layer
- Ash introduced the **islands architecture** — HTMX + Alpine.js for interactive components within Django templates
- Lota confirmed design freedom is sufficient with the hybrid approach
- Alessandro validated HTMX + Alpine.js with one concern: cart state complexity
- Alessandro flagged: Wagtail API mode should be kept available as a future seam

### Concerns Raised (Round 1 — independent)

**Ash:** Cart state complexity with HTMX; no component reusability story in Django templates; image optimisation not automatic in Wagtail.

**Patrick:** Single deployable = single point of failure; Wagtail i18n complexity if set up wrong early; no deployment strategy defined yet.

**Alessandro:** Wagtail learning curve; HTMX + Django CSRF handling needs deliberate setup; no testing strategy defined.

**Lota:** Wagtail StreamField gives editors too much layout freedom; no visual language defined yet; mobile-first not explicitly confirmed.

### Owner Flags
1. **Checkout complexity** — noted as a real concern
2. **No monolith** — replaceability of parts is the core requirement

### Three Paths Defined
| Path | Description |
|------|-------------|
| Path 1 | Modular monolith — Django + Wagtail, internal boundaries |
| Path 2 | Wagtail headless + separate frontend + integrated commerce |
| Path 3 | Three-layer — Wagtail (content) + Saleor (commerce) + frontend |

### Orchestrator Input
- Owner is a developer using AI assistance — changes the build time equation significantly
- Proposed a fourth option: Django REST Framework (no Wagtail) + Next.js + custom admin
- Registered Concern: custom stack means owning security, localisation, image handling

### Dominick's Assessment
- Path 2 (headless) is most auditable from security standpoint
- Non-negotiables: Nginx as reverse proxy, separate identity domains for admin vs customers, SimpleJWT or equivalent, rate limiting from day one
- Approved deployment topology: Docker containers, Nginx as single entry point, internal Docker network for service communication

### Patrick + Alessandro on Deployment
- Docker containers on single VPS
- Django API and Next.js as separate containers
- Nginx as reverse proxy — only public entry point
- Admin app isolated — not exposed through public API

---

## Final Recommendation

| Layer | Technology | Purpose |
|-------|-----------|---------|
| API | Django REST Framework | Data, business logic, auth, image handling |
| Frontend | Next.js | Public-facing site — all three languages |
| Admin | Custom Django admin app | Artisan control panel — isolated from public API |
| Reverse proxy | Nginx | Single entry point, SSL, rate limiting |
| Deployment | Docker on VPS | Process isolation, ~€10–20/month |
| Database | PostgreSQL | Reliable, proven, Django-native |

**Supersedes:** Django + Wagtail decision from earlier session.

---

## Next Steps (pending owner approval)
1. **Dominick** — security baseline document
2. **Patrick** — full architecture diagram and data model outline
3. **Lota** — information architecture and design direction
4. **Alessandro** — deployment setup and test contract

---

## Open Items
- Owner to have individual discussions with each team member before final approval
- Visual language / design direction not yet defined (Lota)
- Localisation implementation approach not yet decided (Alessandro/Patrick)
- Cart state management pattern not yet defined (Ash)
