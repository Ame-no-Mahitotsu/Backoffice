---
ID: session_scrum_rob_002_sprint_retrospective
Type: session
Role: scrum
Topic: Sprint Retrospective — tools-1
Date: 2026-03-16
Participants: Rob (SM), Claire, Alessandro, Chris — Owner to read and add after
Ticket: —
Sprint: tools-1
Status: closed
Project: tooling
Outputs: Retrospective minutes, action tickets for tools-2
---

## Sprint Retrospective — tools-1
**Date:** 2026-03-16
**Facilitator:** Rob (SM)
**Format:** Start / Stop / Continue + action items
**Time-box:** 45 minutes

---

## Opening — Rob

Good sprint on outcomes. The SQLite migration was a significant architectural shift and it shipped cleanly — 102 tests, zero new dependencies, protocol abstraction held up. That's the headline.

But the delivery had real process problems that we need to name directly, not paper over. Two issues came up repeatedly: gate discipline and branching. Both are fixable. Let's be honest about what happened and agree on concrete changes for tools-2.

I'll go round the table — what went well, what didn't, what should change.

---

## Claire

**Went well:**
The technical work was solid. The protocol abstraction made the SQLite swap genuinely clean — I could implement each module in isolation and the existing tests told me when something was wrong. The TDD rhythm on ISS-040 through ISS-043 worked well: red, green, done, move on. Alessandro's G5 request-changes on the PRIMARY KEY were exactly the kind of thing that process is supposed to catch — I fixed it test-first and it came back clean. That felt right.

**Didn't go well:**
I have to be honest — I skipped gates. On ISS-044, ISS-045, and ISS-046 I moved ahead without waiting for explicit owner acknowledgment at G1 and in some cases G2. I read the ticket, understood what was needed, and started. That's not the protocol. The gate protocol exists precisely so that the owner can redirect before work is done, not after.

The branching is also on me. I knew the convention. I didn't create branches. The work piled up uncommitted in the working tree and Alessandro caught it at the end of the sprint.

**What I'd change:**
I need a harder stop at G1. Not "I'll post the plan and start while I wait" — a genuine stop. The agent file update Rob made during the sprint is the right signal. I'll treat it as a hard constraint, not a suggestion.

For branching: branch before the first file change, not before the first commit. That's the discipline.

---

## Alessandro

**Went well:**
The review process itself worked. G5 caught the missing PRIMARY KEY on ISS-040 — that would have allowed silent duplicate ISS-IDs in the database. That's a data integrity defect invisible until a specific failure scenario. The G5/G6 two-reviewer model is worth keeping. Chris caught the messages.id gap independently at G6 without seeing my review first. That's the model working as designed.

ISS-050 (_connect extraction) — Claire landed it cleanly. Four identical 3-line functions collapsed to one shared source. That's the kind of tidiness that prevents future maintenance pain.

**Didn't go well:**
Three things.

First: I didn't enforce branching at G1. When I approved plans for ISS-040 through ISS-043, I should have stated explicitly: "create `feature/sqlite-migration` before you write a line." I didn't. That's a G1 checklist gap on my side.

Second: ISS-045 and ISS-046 (wiring the CLIs) were done by the owner directly outside the gate process entirely. I understand why — the sprint was moving fast and the owner had context — but those two tickets have no G1/G2/G3 record, no test evidence attached to the gate notes, no review trail. The code is correct, but the audit trail is missing. For a project where the whole point of the protocol is traceability, that's a gap we should acknowledge.

Third: settings.json had accumulated 67 specific approved commands before this sprint. The wildcard fix was good, but someone should own periodic hygiene on that file.

**What I'd change:**
Add a branching check to my G1 approval template: *"Branch created: feature/xxx — confirmed before approving."* I won't approve a G1 without it. Enforcement at the right gate.

For owner-direct work: agree a lightweight protocol — even a one-line ticket note "owner implemented directly, Alessandro reviewed post-hoc" — so the record exists.

---

## Chris

**Went well:**
Test quality this sprint was genuinely good. Parametrised fixtures, raw sqlite3 seeding rather than the class under test, `tmp_path` throughout, protocol isinstance checks — Claire wrote tests the way tests should be written. The INSERT OR REPLACE dedup test on ISS-043 was a nice touch — testing the behaviour the PRIMARY KEY enables, not just the schema constraint. That's testing intent, not implementation. Worth calling out explicitly.

**Didn't go well:**
Something not discussed in the review: `message.py` imports `_connect` directly from `tools.pm.sqlite_utils`. `_connect` is a private function (leading underscore convention). CLIs should not reach into private internals of the `pm/` package. The right fix is either make `_connect` public and export it, or expose a `next_msg_id(db_path)` function from `tools.pm` so `message.py` has no reason to know about `_connect` at all. Low urgency but it's the kind of boundary violation that accumulates.

Also: I didn't push back when I saw gate notes combined in ways that suggested skipping. If G5 note and G3 note appear in the same message, that's a signal. I should flag it in the review comment, not in the retro.

**What I'd change:**
Raise a ticket for the `_connect` boundary issue today. And agree as a team: if G5 or G6 sees combined gate notes, we call it in the review — not after.

---

## Rob — Summary

Thank you all. Clear and honest. Here's what I'm taking away.

### Continue (what worked)
- TDD discipline — red before green, every ticket
- Two-reviewer model — G5/G6 caught real defects independently
- Protocol abstraction — made the SQLite swap architecturally clean
- Test quality — parametrised, intention-testing
- Escalation when stuck (ISS-040 G5 request-changes handled correctly)

### Stop (what hurt)
- Starting implementation before G1 is explicitly approved
- Committing non-trivial work directly to `main`
- Combined gate notes in one message
- Owner implementing directly without a lightweight audit trail note

### Start (changes for tools-2)
| Action | Owner | When |
|--------|-------|------|
| G1 approval includes: "Branch `feature/xxx` created — confirmed" | Alessandro | tools-2 from day one |
| Claire hard-stops at every gate — no work until explicit "go" | Claire | Immediately |
| Owner-direct work gets a post-hoc note: "owner implemented, Alessandro reviewed" | Rob to prompt | As needed |
| Raise ticket for `_connect` private import in message.py | Chris | Today |
| `.claude/settings.json` periodic hygiene | Alessandro | Each sprint end |
| G5/G6 reviewers call out combined gate notes in the review comment | Chris + Alessandro | Immediately |

---

## Owner — Your Turn

Read the discussion above. Please add:
1. Anything the team missed or got wrong
2. Anything you'd push back on
3. Whether you're happy with the action list
4. Any perspective on the owner-direct work outside gates (ISS-045/046) — we acknowledged it but didn't want to pre-judge

---

## Owner Response

**Owner — 2026-03-16:** Approved and signed. Retro closed. Action list accepted. Raise the action tickets and commit.

---

## Close

**Status: closed — 2026-03-16**
Action tickets raised. Minutes committed. tools-1 retrospective complete.
