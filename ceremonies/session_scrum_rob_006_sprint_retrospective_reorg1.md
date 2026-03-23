---
ID: session_scrum_rob_006_sprint_retrospective_reorg1
Type: session
Role: scrum
Topic: Sprint Retrospective — reorg-1
Date: 2026-03-23
Participants: Rob (SM), Salvatore, Alessandro, Chris, Claire — Owner to read and add after
Ticket: —
Sprint: reorg-1
Status: open
Project: tooling
Outputs: Retrospective minutes, action items
---

## Sprint Retrospective — reorg-1
**Date:** 2026-03-23
**Facilitator:** Rob (SM)
**Format:** Start / Stop / Continue + action items
**Time-box:** 45 minutes

---

## Opening — Rob

reorg-1 was not a standard sprint. No TDD, no G1-G7 gates, no feature work — this was a structural operation: split the monorepo, wire five repos together, keep the team functional. On the whole, the outcome is correct and clean. The new layout is already paying dividends — separation of concerns is enforced at the repo level, not just by convention.

But we have two things to address honestly: the sprint sat in a half-closed state for longer than it should have, and one fix in ISS-133 was incomplete on delivery. Neither is serious, but both are patterns we've seen before.

Round the table.

---

## Salvatore

**Went well:**
The sequencing held. Phases 0 through 2 landed cleanly and unblocked everything downstream. The bare repo setup, GitHub remotes, and hephestus population were done without incident. Phase 4, 5, and 6 — docs, backoffice, and the development folder — followed once the blockers cleared. The VS Code multi-root workspace was the right call: it means the owner works in one editor window across all five repos, which is the correct experience for a single-operator project.

**Didn't go well:**
I delivered the infra work and then effectively stopped tracking the sprint. Phases 3a (ISS-132) and 6 (ISS-137) were done, but I didn't close the tickets — I committed and moved on. If the sprint close had depended on my ticket hygiene alone, it would still be open. That's not acceptable in a tracked project.

**What I'd change:**
Close the ticket immediately after the commit is pushed and verified. Not at the end of the session. Not the next day. At the point of "DONE WHEN" satisfied — that's when the status changes.

---

## Alessandro

**Went well:**
The validation pass in ISS-140 caught what it needed to catch. The cross-reference updates in ISS-138 were more complete than expected — Phases 7 and 10 were done with full coverage of all document paths. The `.gitignore` entry for `development/worktrees/` in ISS-137 was a clean detail that would have caused noise if missed.

The multi-root workspace approach is architecturally sound. We now have clean repo boundaries with no bleed between governance, tooling, docs, backoffice, and site code. That's the right shape.

**Didn't go well:**
Phase 10 was the last commit, and after it landed the sprint looked done from a git perspective — commits made, EXP-007 written. But six tickets were still `open`. I did not do a backlog check before declaring it done in my own head. Rob had to flag the discrepancy.

The ISS-133 incomplete fix is also partly on me as Tech Lead. WORKSPACE_DIR in message.py was not in the ticket's explicit scope, but it was the same class of problem — stale pre-reorg path — and I reviewed the ticket without catching it. Alessandro caught it during this retro cycle, not during the sprint.

**What I'd change:**
End-of-sprint checklist before anyone says "done": run `ticket.py sprint reorg-1` and confirm zero open tickets. That command takes five seconds. There's no excuse for not running it.

---

## Claire

**Went well:**
ISS-133 and ISS-134 were well-scoped. The DB path fixes were mechanical but necessary, and the `--handoff-dir` flag in ISS-134 mirrors exactly what was done in handoff.py — consistent pattern, low cognitive overhead.

The pytest suite running green immediately after the path fixes (206 tests) gave instant confidence that the import changes hadn't broken anything. The test suite as a safety net for structural changes: that's TDD paying a dividend it didn't even specifically plan for.

**Didn't go well:**
The WORKSPACE_DIR miss. ISS-133 said "fix DB_PATH in three files" and I fixed those three files. But the same stale path pattern was sitting on line 25 of message.py in a different constant and I didn't look for it. I was ticket-scoped when I should have been problem-scoped. The problem was "stale pre-reorg paths" — I should have searched for all of them, not just the ones explicitly listed.

**What I'd change:**
When a ticket asks me to fix a pattern (stale paths, wrong imports, broken references), I should run a search for all instances of the pattern — not just fix the ones named. A `grep -rn "memory"` across the tools directory before closing ISS-133 would have caught it in thirty seconds.

---

## Chris

**Went well:**
The cross-reference work (ISS-138) was more thorough than I expected to find. The agent files, constitution, ai-standards, and workspace paths were all updated consistently. The reorg didn't leave dangling references in the documents that matter.

The expedition dispatches entry (EXP-007) is a good permanent record — it explains the why of the reorg, lists all tickets, and notes the archive location. Future team members (or future AI agents) will be able to reconstruct the full context from that entry alone.

**Didn't go well:**
This sprint had no test coverage gate, which is correct for infra work — you can't write a unit test for "does the git remote point at GitHub." But it means we relied entirely on manual verification, and the WORKSPACE_DIR miss is the direct consequence. When you remove the test-green signal, you need a compensating check. We didn't define one explicitly.

**What I'd change:**
For future infra/devops sprints: define a "done signal" up front that compensates for the absence of a test suite. In reorg-1 that could have been: "all scripts smoke-test clean from the new location." A `python ticket.py list` and `python message.py list` from the new tools/ path would have exercised both DB_PATH and WORKSPACE_DIR. One command, thirty seconds, would have caught the miss.

---

## Rob — Summary

### Continue
- Phased sprint structure for complex infra work — the dependency chain (ISS-129 → 130 → 131 → 132 → 133 → …) held and executed correctly
- VS Code multi-root workspace approach — the right productivity decision for a single-operator multi-repo project
- Expedition Dispatches as the permanent record — EXP-007 is complete and comprehensive
- Pytest suite as structural change safety net — 206 green tests after the reorg path fixes is a meaningful signal

### Stop
- Declaring a sprint done based on git commits rather than backlog state — the ticket list is the source of truth, not the last commit message
- Ticket-scoped thinking when the actual problem is pattern-scoped — search for all instances, not just the listed ones

### Start

| Action | Owner | When |
|---|---|---|
| Before any sprint close: run `ticket.py sprint <name>` and confirm zero open tickets | Alessandro (and all roles) | From next sprint |
| When fixing a pattern (stale paths, wrong imports): grep for all instances before closing the ticket | Claire | Immediately |
| For infra/devops sprints with no test suite: define a smoke-test command up front as the "done signal" | Salvatore + Rob | At sprint planning |

---

## Owner — Your Turn

Read the discussion above. Please add:
1. Anything the team missed or got wrong
2. Whether you agree with the three Start items above
3. Any observations on the new multi-repo layout from your perspective as sole operator

---

## Owner Response

**Owner — 2026-03-23:** Agrees with all conclusions. Three action items approved. Raise them as tickets and open site-1.

---

## Close

**Status: closed — 2026-03-23**
Action tickets raised: see sprint planning session.
reorg-1 retrospective complete.
