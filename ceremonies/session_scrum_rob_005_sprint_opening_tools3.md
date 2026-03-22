---
ID: session_scrum_rob_005_sprint_opening_tools3
Type: session
Role: scrum
Topic: Sprint Opening — tools-3
Date: 2026-03-17
Participants: Rob (SM), Claire, Alessandro, Chris, Lauren — Owner observing
Ticket: ISS-056,ISS-058,ISS-062,ISS-063,ISS-064,ISS-065,ISS-066,ISS-067,ISS-068,ISS-071,ISS-072,ISS-073
Sprint: tools-3
Status: closed
Project: tooling
Outputs: tools-3 sprint opened, board committed, wave sequencing confirmed, Owner approved
---

## Sprint Planning — tools-3
**Date:** 2026-03-17
**Facilitator:** Rob (Scrum Master)
**Format:** Sprint planning — team read, Owner approved before close
**Status:** tools-3 officially open

---

## Sprint Goal

> **Complete the PM CLI, document all tooling for agents, and unblock site-1.**

---

## Sprint Board

| ID | Title | MoSCoW | Owner | Wave |
|----|-------|---------|-------|------|
| ISS-062 | Agent Read First: replace `issue_backlog.md` ref with `ticket.py list` | Must | Alessandro | 1 |
| ISS-056 | G1 Approval Template: clarify base branch by track | Must | Alessandro | 1 |
| ISS-065 | End-of-sprint agent file review (tools-3 instance) | Must | Alessandro | 1 (close) |
| ISS-063 | Lauren: E2E + CI test plan for site track | Must | Lauren | 1 |
| ISS-066 | `ticket.py view` subcommand | Must | Claire | 2 |
| ISS-058 | `ticket.py reopen` subcommand | Should | Claire | 2 |
| ISS-068 | `ticket.py sprint` summary subcommand | Should | Claire | 2 |
| ISS-067 | `message.py list/view` subcommands | Should | Claire | 2 |
| ISS-071 | ticket.py: blocks/blocked-by dependency field | Should | Claire | 2 |
| ISS-072 | ticket.py list: --gate filter for review queue | Should | Claire | 2 |
| ISS-073 | ticket.py search: full-text search across titles and notes | Should | Claire | 2 |
| ISS-064 | Agent files: document ticket query API for all roles | Must | Alessandro | 3 (last) |

---

## Wave Sequencing — Hard Dependencies

**Wave 1** — process and documentation fixes, no code dependencies. Alessandro and Lauren work in parallel.

**Wave 2** — all CLI subcommands and schema additions. Claire works through in order:
1. ISS-066 (`view`) — smallest, highest value
2. ISS-058 (`reopen`) — removes a real workflow gap
3. ISS-068 (`sprint` summary) — Rob needs for standups
4. ISS-067 (`message.py list/view`) — completes message CLI
5. ISS-071 (dependency field) — schema change, do after simpler subcommands
6. ISS-072 (gate filter) — builds on list subcommand
7. ISS-073 (search) — last, self-contained

**Wave 3 — ISS-064 is gated on all Wave 2 tickets merged to main.**
Alessandro documents the finished tooling once — no partial documentation, no amendments. This is the sprint's closing gate.

ISS-065 (end-of-sprint agent review) executes after ISS-064 is merged — it is the final act of the sprint.

---

## Sequencing Guidance

**Alessandro:** ISS-062 and ISS-056 first — small, high-impact, unblock every agent operating in this sprint. Then stand by for G5 reviews on Claire's Wave 2 work. ISS-064 and ISS-065 close the sprint.

**Lauren:** ISS-063 is your sole focus. Deliverables: (1) Playwright E2E strategy for site track, (2) accessibility gate definition WCAG 2.1 AA, (3) CI/CD gate proposal. The tooling unit test strategy already exists — do not repeat it. Site-1 does not open without this. Mid-sprint pace risk applies — Rob will escalate to Owner if no G1 plan is posted by mid-sprint.

**Claire:** Wave 2 in order above. Branch is the first action before opening any file — no exceptions.

**Chris:** No assigned tickets. Standing by for G6 reviews on all Claire Wave 2 tickets.

---

## Process Rules — Active From Day One

| Rule | Owner |
|---|---|
| Branch creation is the first physical action on any ticket — before opening any file | Claire |
| G5 declined if no G1 note exists in the ticket record — send back, no review | Alessandro |
| Call out combined gate notes in G6 review comment at the time, not in the retro | Chris |
| ISS-064 not started until all Wave 2 tickets are merged to main | Alessandro |

---

## Backlog — Not in tools-3

| ID | Title | Reason |
|---|---|---|
| ISS-060 | OR filters on list | Could — deferred to tools-4 |
| ISS-069 | handoff.py list/latest | Could — deferred to tools-4 |
| ISS-070 | ticket.py close shortcut | Could — deferred to tools-4 |
| ISS-014 | Dashboard project filter | Site track not open |

---

## Owner Sign-Off

**Owner — 2026-03-17:** Approved. Sprint committed.

---

## tools-3 is open

Board committed. Wave sequencing confirmed. ISS-064 gates on all Wave 2 work. ISS-065 closes the sprint.

Rob — standing by to facilitate. Flag blockers as they arise.
