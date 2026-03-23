---
ID: session_scrum_rob_007_sprint_opening_site1
Type: session
Role: scrum
Topic: Sprint Opening — site-1
Date: 2026-03-23
Participants: Rob (SM), Sarah, Alessandro, Claire — Owner approved
Ticket: ISS-099,ISS-100,ISS-101,ISS-102,ISS-103
Sprint: site-1
Status: closed
Project: site
Outputs: site-1 sprint opened, board committed, wave sequencing confirmed, Owner approved
---

## Sprint Planning — site-1
**Date:** 2026-03-23
**Facilitator:** Rob (Scrum Master)
**Format:** Sprint planning — team read, Owner approved before close
**Status:** site-1 officially open

---

## Sprint Goal

> **Establish the visual and data foundations of the site: UX direction, design tokens, and a reviewed data model — then scaffold the Django apps and core models.**

---

## Sprint Board

| ID | Title | MoSCoW | Owner | Wave | Reviewer |
|----|-------|---------|-------|------|---------|
| ISS-099 | UX: mood boards and design direction | Must | Sarah | 1 | Owner (single) |
| ISS-100 | UX: Tailwind design token spec | Must | Sarah | 1 | Alessandro (single) |
| ISS-101 | Architecture review: data model | Must | Alessandro | 1 | Owner (single) |
| ISS-102 | Django apps scaffold | Must | Claire | 2 | Alessandro (G5) + Chris (G6) |
| ISS-103 | Core models | Must | Claire | 2 | Alessandro (G5) + Chris (G6) |

---

## Wave Sequencing — Hard Dependencies

**Wave 1** — UX and architecture in parallel. No implementation starts until both ISS-100 and ISS-101 are signed off.

- Sarah works ISS-099 then ISS-100 in order
- Alessandro works ISS-101 independently and in parallel
- ISS-099 and ISS-101 are reviewed by the Owner
- ISS-100 is reviewed by Alessandro

**Wave 2 — gated on ISS-100 AND ISS-101 both resolved.**

Claire starts ISS-102 (apps scaffold) then ISS-103 (core models) in order.
Both go through full Agent Execution Protocol with the standard two-reviewer model (G5 Alessandro + G6 Chris).

---

## Review Model — site-1

UX deliverables (ISS-099, ISS-100) and the architecture review (ISS-101) use a **single reviewer** — they are design and document artefacts, not code. The two-reviewer model applies from Wave 2 onward where Claire writes code.

| Ticket | Review model | Reviewer |
|--------|-------------|---------|
| ISS-099 | Single | Owner |
| ISS-100 | Single | Alessandro |
| ISS-101 | Single | Owner |
| ISS-102 | G5 + G6 | Alessandro + Chris |
| ISS-103 | G5 + G6 | Alessandro + Chris |

---

## Sequencing Guidance

**Sarah:** ISS-099 first — direction before tokens. Mood boards and page/component design direction. Output goes to Owner for sign-off. ISS-100 second — Tailwind token spec (colours, typography, spacing). Output goes to Alessandro.

**Alessandro:** ISS-101 is yours from day one, in parallel with Sarah. Review the approved data model, flag any gaps or risks before Claire touches any code. Output goes to Owner for sign-off.

**Claire:** No action in Wave 1. Wave 2 opens when both ISS-100 and ISS-101 are resolved. Branch is the first physical action — before opening any file.

**Chris:** Standing by for G6 reviews on ISS-102 and ISS-103 in Wave 2.

---

## Owner Sign-Off

**Owner — 2026-03-23:** Approved. Sprint committed.

---

## site-1 is open

Board committed. Wave sequencing confirmed. Wave 2 gated on ISS-100 and ISS-101 both resolved.
