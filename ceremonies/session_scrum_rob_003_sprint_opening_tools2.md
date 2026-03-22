---
ID: session_scrum_rob_003_sprint_opening_tools2
Type: session
Role: scrum
Topic: Sprint Opening — tools-2
Date: 2026-03-16
Participants: Rob (SM), Claire, Alessandro, Chris — Owner observing
Ticket: ISS-014,ISS-032,ISS-035,ISS-038,ISS-051,ISS-053,ISS-054,ISS-039
Sprint: tools-2
Status: closed
Project: tooling
Outputs: tools-2 sprint opened, team briefed, board committed, process changes active
---

## Sprint Opening — tools-2
**Date:** 2026-03-16
**Facilitator:** Rob (Scrum Master)
**Format:** Sprint opening briefing — team read, no ceremony required
**Status:** tools-2 officially open

---

## Sprint Goal

> **Make the PM CLI fully queryable and well-bounded, and embed all retro process corrections from tools-1.**

This is a tighter sprint than tools-1. Eight tickets. No migrations, no architectural pivots. The work is completeness and discipline.

---

## Sprint Board

| ID | Title | MoSCoW | Owner |
|----|-------|---------|-------|
| ISS-054 | G1 template: branch name confirmation required | Must | Alessandro |
| ISS-053 | `message.py` private `_connect` boundary violation | Must | Claire |
| ISS-035 | `ticket.py update`: unhandled ValueError | Must | Claire |
| ISS-051 | `ticket.py list`: query interface + list subcommand | Should | Claire |
| ISS-032 | `ticket.py create`: `--severity` for defect type | Should | Claire |
| ISS-038 | `ticket.py create`: `--session` flag | Should | Claire |
| ISS-014 | Dashboard: project filter | Should | Claire |
| ISS-039 | Agent files: mark `.github/agents/*.agent.md` as canonical | Could | Alessandro |

**Must items are sprint blockers.** Should items all ship if pace holds — none are large. ISS-039 is a Could: Alessandro picks it up when ISS-054 is done.

---

## Sequencing guidance

**Alessandro — start with ISS-054.** It is a Must and it governs how every other ticket begins. Until your G1 template is updated, the branch confirmation requirement is not enforced. Do it first.

**Claire — start with ISS-053.** Small, bounded, retro action. Clear the retro debt before touching larger tickets. ISS-035 next. Then ISS-051 — but read the note below on ISS-051 before you start.

**ISS-051 — design step at G2.** Sofia has revised the scope: the query interface (protocol, class, or equivalent abstraction) must be defined and Alessandro-approved at G2 before implementation begins. `list` is a consumer of that interface. If Alessandro's G2 review takes time, pick up ISS-032 or ISS-038 in parallel — do not sit idle.

---

## Process changes active from tools-2 day one

These come directly from the retro. They are not suggestions.

### Alessandro — G1 approval template updated
Every G1 approval must now include explicit branch confirmation:

> *"Branch `feature/xxx` created — confirmed before approving."*

No G1 approval without it. If Claire posts a G1 plan without a branch name, ask for it before approving.

### Claire — hard stop at every gate
No work begins until the preceding gate has explicit owner acknowledgment. Not "I'll post the plan and start while I wait." A genuine stop.

To be precise:
- G1 plan posted → wait for "go"
- G2 red test posted → wait for "confirmed red, go"
- G3 green posted → wait for "confirmed green"

There is no ambiguity here. If you are waiting, you are waiting. If the owner is unavailable, you are blocked — log it and stop.

### Chris — call out combined gate notes in the review
If you see a G3 and G5 posted in the same message, or any gate notes that suggest skipping, call it in your G6 review comment. Not in the retro — at the time.

### Owner-direct work (if it happens)
If the owner implements directly outside the gate process, I will prompt for a one-line post-hoc note: *"Owner implemented directly — Alessandro reviewed post-hoc."* The audit trail must exist.

---

## ISS-052 note
ISS-052 was closed as a duplicate of ISS-051 at planning. ISS-051 with revised scope absorbs it. Do not reference ISS-052.

---

## tools-2 is open

The board is committed. Process changes are active. ISS-054 is first.

Rob — standing by to facilitate. Flag any blockers as they arise.
