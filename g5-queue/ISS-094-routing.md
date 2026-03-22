# ISS-094 — G5 Routing Note

**Routed to:** Alessandro (Tech Lead) — G5 review

**Date:** 2026-03-20
**Run:** TL-2026-03-20-002
**Branch:** `feature/iss-094-note-length-cap`
**Commit:** `47956ed`
**Status:** in-testing

## What was built
Hard 2000-character cap on `--notes` (create) and `--note` (update) inputs
in `ticket.py`. Validation is in the handler layer (not the SQLite writer),
with explicit `sys.exit(1)` + stderr message. No silent truncation.

## Gate artifacts
- G1: `memory/workspace/tooling/gate-output/ISS-094-G1.md`
- G2: `memory/workspace/tooling/gate-output/ISS-094-G2.md` (red: 2 failed, 2 passed)
- G3: `memory/workspace/tooling/gate-output/ISS-094-G3.md` (green: 27 passed)

## Files changed
- `ticket.py` — `_MAX_NOTE_LEN = 2000` constant + guards in `handle_create` and `handle_update`
- `tools/tests/test_ticket_note_length.py` — new test file (4 tests)

## Test results
27 passed, 0 failed. All 23 pre-existing create/update tests continue to pass (AC4).

## Acceptance criteria
- AC1 ✅ Explicit error message + sys.exit(1); no truncation
- AC2 ✅ Cap on both create (--notes) and update (--note)
- AC3 ✅ Unit tests: 2001 chars → SystemExit; 2000 chars → accepted
- AC4 ✅ All existing tests pass

## G6 reviewer
Chris (Senior Full Stack) — domain review for tooling/process CLI changes.

## Notes for Alessandro
Validation is placed at the top of the handler, before DB operations, which is
the correct layer (operational hygiene control, not a DB integrity constraint).
Error message format: `Error: --notes value exceeds 2000-character limit (got N chars)`.
Confirm the message is clear enough for automated pipeline error surfacing.
