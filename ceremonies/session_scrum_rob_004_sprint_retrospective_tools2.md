---
ID: session_scrum_rob_004_sprint_retrospective_tools2
Type: session
Role: scrum
Topic: Sprint Retrospective — tools-2
Date: 2026-03-17
Participants: Rob (SM), Claire, Alessandro, Chris — Owner to read and add after
Ticket: —
Sprint: tools-2
Status: open
Project: tooling
Outputs: Retrospective minutes, action items, ISS-063 raised
---

## Sprint Retrospective — tools-2
**Date:** 2026-03-17
**Facilitator:** Rob (SM)
**Format:** Start / Stop / Continue + action items
**Time-box:** 45 minutes

---

## Opening — Rob

tools-2 was a solid sprint on output. Nine tickets resolved, the CLI is hardened, the agent protocol has teeth, and the G5/G6 two-reviewer model held up across every ticket. That's genuine progress.

But we have two things to address honestly: a process violation that repeated from tools-1 (ISS-059 — code on main before G1), and a structural miss that has now spanned two full sprints — Lauren's absence. I'll cover both.

Round the table — what went well, what didn't, what changes.

---

## Claire

**Went well:**
The scope in tools-2 felt right-sized. ISS-051 (list subcommand) and ISS-057 (markdown removal) were clean, self-contained pieces of work. The TicketQuery API on ISS-055 gave the reader a proper interface — that's the kind of small thing that prevents future coupling. I'm proud of that one.

**Didn't go well:**
ISS-059. I did it again. I had the code written before I had a branch and before I'd posted a G1 plan. The branch was even on main for a while. Alessandro caught it, I moved the work correctly, and the outcome was fine — but the process was wrong and I knew it while I was doing it. I convinced myself the ticket was small enough to shortcut. That's the exact reasoning the protocol is designed to override.

**What I'd change:**
I need to treat the branch as the first action, before I open any file. Not "I'll create it when I have something to commit." Branch first, always. The G1 discipline is on me.

---

## Alessandro

**Went well:**
The G1 branch-confirmation template (ISS-054) was the right intervention from tools-1's retro and it landed in the first week of tools-2. Every G1 after that carried the branch confirmation. That's a process improvement that will compound.

ISS-035's G5/G6 reviews were done independently and both reached the same conclusion — the fix was correct, no issues. That's the two-reviewer model working exactly as intended.

**Didn't go well:**
I have to own the ISS-059 catch being reactive, not proactive. I found the uncommitted changes on main during G5 — which is the wrong moment to catch a branching violation. The right moment is G1. The template I introduced in ISS-054 asks for branch confirmation at G1 approval — but ISS-059 never had a G1 posted, so there was nothing to approve. The gap is: what do we do when an implementer skips G1 entirely and presents work at G5?

**What I'd change:**
If I receive a G5 request on a ticket with no G1 note in the ticket record, I decline the G5 and send it back. No exceptions. That's the gate discipline. I was too accommodating in ISS-059 by reviewing the work at all before the process was clean.

---

## Chris

**Went well:**
93 tests, all green, 93% coverage at the point of ISS-059's G6. Coverage is well above the 80% gate. Claire's test for the deferred-editable path was end-to-end and tested the right thing — not "does this line execute" but "is the ticket actually editable after the fix." That's the standard.

The ISS-059 code split of `_CLOSED_STATUSES` into `_IMMUTABLE_STATUSES` and `_NOTE_REQUIRED_STATUSES` is the kind of naming clarity that makes future readers' lives easier. It's a small change with outsized legibility value.

**Didn't go well:**
Lauren. We are building a tooling track with a solid test suite, and we are heading into a site track with no E2E strategy, no accessibility gate, no CI definition. That's Lauren's work. She has a strategy document from tools-1 that covers the tooling unit test layer well — but there is nothing for Playwright, nothing for WCAG 2.1 AA enforcement, nothing for CI gates. If site-1 opens without that, we will either skip testing or improvise it, and both outcomes are bad. ISS-063 is the right ticket. It needs to be in the next sprint, first item, non-negotiable.

**What I'd change:**
Lauren needs a concrete deliverable assigned with a hard dependency: site-1 does not open until ISS-063 is resolved.

---

## Rob — Summary

### Continue
- Two-reviewer model (G5/G6 independent) — working, keep it
- TDD discipline — all new tests were red-first
- Coverage gate at 80% — holding, no exceptions
- G1 branch-confirmation template — introduced and used from ISS-054 onward

### Stop
- Skipping G1 before implementation starts — no exceptions, no matter how small the ticket
- Committing to main before a branch exists
- Reviewing at G5 when G1 has no record — Alessandro to decline and send back

### Start

| Action | Owner | When |
|---|---|---|
| Branch creation is the first physical action on any ticket — before opening any file | Claire | Immediately |
| G5 declined if no G1 note exists in the ticket record | Alessandro | Immediately |
| ISS-063 (Lauren's E2E + CI plan) is sprint gate: site-1 does not open until resolved | Rob | Sprint planning |
| Lauren delivers ISS-063 before any site track ticket enters in-development | Lauren | Before site-1 |

---

## Owner — Your Turn

Read the discussion above. Please add:
1. Anything the team missed or got wrong
2. Whether you agree ISS-063 should gate site-1 opening
3. Any other observations on tools-2

---

## Owner Response

**Owner — 2026-03-17:** Agrees with all conclusions. Two additional points approved:
1. Agent files must document the ticket query API (TicketQuery, SQLiteBacklogReader) so agents know how to search tickets correctly — ISS-064 raised.
2. A recurring end-of-sprint agent file review ticket must be created at each sprint planning to verify AI files reflect the latest tools and process — ISS-065 raised.
Chris's addition noted: the agent file review must explicitly check that the API documented in agent files matches the actual codebase — drift is a bug.

---

## Close

**Status: closed — 2026-03-17**
Action tickets raised: ISS-063 (Lauren E2E plan), ISS-064 (agent file API docs), ISS-065 (end-of-sprint agent review).
Minutes committed. tools-2 retrospective complete.
