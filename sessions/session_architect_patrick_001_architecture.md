---
ID: session_architect_patrick_001_architecture
Type: session
Role: architect
Topic: Architecture plan — deployment topology, Django app structure, data model, localisation strategy
Date: 2026-03-12
Participants: Owner, Patrick
Ticket: —
Sprint: —
Status: closed
Project: site
Outputs: architecture plan, deployment topology, data model, localisation strategy, architecture_document.md inputs, constitution entries
---

# Session: Patrick (Architect) — Architecture Plan
**Date:** 2026-03-12
**Status:** Approved by Project Owner

---

## Deployment Topology

```
Internet
    │
  Nginx  (reverse proxy, SSL, rate limiting)
    │
    ├──► Next.js  (public frontend — port 3000)
    │
    └──► Django   (API + Admin app — port 8000)
              │
           PostgreSQL  (internal only — never exposed)
```

## Django Internal Structure

```
django/
├── api/          ← public REST API
├── admin_app/    ← artisan admin (separate auth)
├── catalogue/    ← product data models
├── commerce/     ← orders, cart sessions
├── gallery/      ← images, time-lapse
├── content/      ← pages, localised text
└── core/         ← shared models, settings, auth
```

## Data Model — Key Entities

| Entity | Key fields |
|--------|-----------|
| Product | name (localised), description (localised), price, sizes |
| ProductImage | product FK, image file, alt text (localised), order |
| Size | name, statue_height_min, statue_height_max, description |
| GalleryItem | image, title (localised), description (localised), tags |
| TimeLapseItem | video/image URL, title (localised), description (localised) |
| Order | customer details, items, status, created_at |
| OrderItem | order FK, product FK, size FK, quantity |
| Page | slug, title (localised), body (localised) |
| Language | code (it/en/de), is_active |

## Localisation Strategy
TranslatedField pattern — text field variants per language stored in related table, not same row. Adding a language = adding rows, not changing schema.

## Decisions Made

| Decision | Choice | Reason |
|----------|--------|--------|
| Image storage | Local filesystem + django-storages abstraction | Scale doesn't justify object storage at launch; easy to migrate later |
| Next.js rendering | SSG with revalidation | Better performance and SEO for catalogue site |
| API style | REST | Simpler to build and audit than GraphQL at this scale |

## Patrick's Deliverable
Formal architecture document covering:
1. Full deployment diagram
2. Django app structure with boundaries
3. Complete data model with localisation pattern
4. API endpoint map
5. Key constraints for all team members
