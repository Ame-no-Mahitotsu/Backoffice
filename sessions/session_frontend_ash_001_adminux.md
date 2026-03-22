---
ID: session_frontend_ash_001_adminux
Type: session
Role: frontend
Topic: Admin UX review — C3 contamination audit, Gabriele interface spec, TOTP UX
Date: 2026-03-13
Participants: Owner, Ash
Ticket: —
Sprint: —
Status: closed
Project: site
Outputs: C3 admin UX review complete, Gabriele interface spec (Images/Texts/Orders), TOTP UX for both roles, no contaminated assumptions found
---

# Session: Ash (UX/UI) — BAR-001 Admin UX Review
**Date:** 2026-03-13
**Status:** Complete — pending group check-in with Patrick (§2/§13 review)
**Trigger:** BAR-001 action item — review admin UX direction for C3-contaminated assumptions

---

## Context

Architecture constraint C3 was "Admin operable by non-technical user." BAR-001 confirmed this was a false assumption: the Owner is the sole permanent technical operator. Gabriele (artisan) has a narrow content scope. All downstream UX decisions that relied on C3 must be reviewed.

---

## C3-Contaminated Assumptions — Identified and Corrected

### 1. "Simplify everything in admin"
- **Old:** Protect a non-technical operator from making mistakes — hide fields, soften labels, limit visible options.
- **Corrected:** Owner is technical. Admin can be functional and data-dense. Standard labels and Django-standard conventions are acceptable for the Owner's scope. No UX padding needed.

### 2. "One user type in admin"
- **Old:** Designing one admin UX for one audience.
- **Corrected:** Two distinct user contexts inside the same auth shell:

| User | Technical level | Scope |
|---|---|---|
| Owner (Django superuser) | Technical professional | Full — products, orders, users, config, translations |
| Gabriele (`artisan` group) | Non-technical | Image upload, text editing, read-only orders/sales |

**Decision:** Single admin at `/gestione/`. After login, nav and content areas render based on Django group. Owner sees full admin nav. Gabriele sees three sections: **Images**, **Texts**, **Orders** (read-only). Items outside his permission scope are not rendered — not hidden-but-accessible, not visible-but-disabled.

### 3. "Block editor needs hand-holding"
- **Old:** Page body editor needed heavy guidance for a non-technical user.
- **Corrected:** Block editor is for the Owner only (page editing is not in Gabriele's permission scope). Clean functional editor — block type selector (heading / paragraph / image / divider / CTA) plus ordered list with drag-to-reorder. Tutorial-style UX is unnecessary.

---

## Gabriele's Interface — Where Simplicity Still Applies

Gabriele IS a non-technical user. His scope is narrow and sessions infrequent.

**Images section:**
- Large drop zone, visually distinct per context: Gallery / Product images / Time-lapse
- Inline caption and alt text fields immediately below each uploaded image
- No configuration, no metadata beyond caption and alt
- Explicit confirmation on save

**Texts section:**
- List of editable text items identified by plain-language label, not slug or technical ID
- One field per item (no multi-field forms)
- Italian / English / German tabs per item — Gabriele edits Italian only; translation management is Owner's responsibility
- No translation status visibility for other languages

**Orders section:**
- Read-only: order reference, date, product name + size, status, customer name
- No editable fields, no action buttons
- Default sort: date descending

**TOTP setup (Gabriele-specific):**
- Larger QR code
- Plain-language instruction: "Open your authenticator app, tap +, scan this code"
- Manual entry key shown below QR in large monospace font
- Explicit confirmation screen after first verified TOTP code before entering admin

---

## TOTP Setup — Owner
Standard flow. QR code, manual key, verification. No hand-holding.

---

## Session Expiry UX (Both Users)
Inactivity timeout → redirect to `/gestione/login/` with: "Your session expired. Please log in again." Username field pre-filled. No timeout duration disclosed to the user.

---

## §13 Note for Architecture Document
Constraint in §13 read "CSS modules or single utility framework (to be agreed)." This is already resolved in architecture_document.md (line 484): **Tailwind CSS + `tailwind.config.ts`**. No further action needed — confirmed as already closed.

---

## Open Items
- Group check-in pending Patrick completing his §2 / §13 review for remaining C3-influenced decisions
- Gabriele's 5 questions (Lota) still outstanding — blocks visual direction but not admin UX structure
