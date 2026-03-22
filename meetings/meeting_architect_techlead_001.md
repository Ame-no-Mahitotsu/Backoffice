---
ID: ARQ-HIST-001
Type: architecture-review
Tags: stack-decision, django, wagtail, self-hosted, superseded
Date: 2026-03-12
Chair: Patrick
Participants: Patrick, Alessandro, Owner
Ticket: —
Sprint: —
Status: closed
Project: site
---

# Meeting: Architect + Tech Lead — Stack Decision
**Date:** 2026-03-12
**Participants:** Patrick (Solution Architect), Alessandro (Tech Lead), Project Owner
**Outcome:** Decision reached — Django + Wagtail, self-hosted

---

## Transcript

**Patrick (Architect):** Opened with four non-negotiables from the BA/PO documents:
1. Localisation is structural — data model must support it from day one
2. Admin must be operable by a non-technical user
3. Clean API boundaries for future extensibility (payment, customer archive, loyalty)
4. Payment stub in v1 is an opportunity — design the contract without provider lock-in

Asked Alessandro: CMS-based, headless CMS, or something else?

---

**Alessandro (Tech Lead):** Pushed back on framing. Flagged the scale of the project — small artisanal business, likely one non-technical person operating it after build. Raised concerns:
- Headless CMS = two systems to maintain, higher operational complexity
- WordPress = plugin ecosystem dependency, hard to evolve cleanly
- Leaned toward a modern self-contained framework with integrated CMS layer
- Asked Patrick: what is the long-term hosting and maintenance reality?

---

**Project Owner clarified:**
- Technical maintainer is the owner (developer)
- Artisan's admin needs are narrow: upload images, edit text labels and page content
- Cost is a factor — no clear picture of managed CMS pricing

---

**Alessandro (Tech Lead):** Self-hosted confirmed as the right direction. Narrow admin surface means no heavyweight CMS needed — just a clean interface for specific operations. Shared rough cost landscape:
- Managed CMS platforms: free tiers exist, paid tiers expensive, vendor dependency
- Self-hosted open source: €5–20/month VPS, no licence fees
- Custom-built: no platform cost but significant extra build time

---

**Patrick (Architect):** Reframed options as three:
- **Option A:** Headless CMS (Strapi/Payload) + frontend framework — clean API, good localisation, more DevOps complexity
- **Option B:** Django + Wagtail — integrated stack, localisation built-in, clean admin, one deployable unit
- **Option C:** Lightweight custom stack — maximum control, minimum bloat, but 2–3x build time

Leaned toward Option B. Asked Alessandro about Node.js preference.

---

**Project Owner asked:** How much more work is Option C?

**Alessandro (Tech Lead):** Option C is 2–3x build time vs Option B to reach admin feature parity. Patrick added: Option C also carries higher long-term maintenance cost — every new feature requires a developer, whereas Option B makes many changes configuration rather than code.

Both aligned: **Option B, Django + Wagtail.**

---

**Project Owner asked:** Are these open source / full products?

**Alessandro:** Both fully open source, no licence fees, large active communities. Django since 2005, Wagtail maintained by Torchbox with commercial backing.

**Patrick:** Production-grade. Django powers Instagram, Pinterest. Wagtail used by NASA, Google, NHS. Hosting cost only — €5–20/month VPS.

---

**Three-way comparison + third contender (Laravel + Filament):**

| | Pros | Cons |
|---|---|---|
| **Django + Wagtail** | Battle-tested, localisation built-in, clean admin, one deployable unit | Learning curve on Wagtail; Python context switch if maintainer prefers JS |
| **Strapi + frontend** | Modern, API-first, Node.js ecosystem | Two systems to run, more DevOps complexity |
| **Laravel + Filament** | Cheapest hosting (PHP everywhere), beautiful admin out of the box | PHP stigma, introduces new language vs Python already in use |

**Patrick ranking:** Wagtail → Laravel → Strapi
**Alessandro ranking:** Wagtail → Laravel → Strapi

Both aligned. Recommendation brought to Project Owner.

---

## Decision

**Stack: Django + Wagtail, self-hosted on a VPS (~€5–20/month)**

Approved by Project Owner.

---

## Open Questions for Next Sessions
- Hosting provider choice (VPS selection)
- Deployment approach (Docker? bare metal?)
- Frontend strategy — does Wagtail's built-in templating suffice, or do we want a separate frontend layer for the public site?
- Localisation implementation detail — django-modeltranslation vs Wagtail's built-in i18n

