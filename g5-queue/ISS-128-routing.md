---
ticket: ISS-128
gate: G5
raised_by: Arale
date: 2026-03-21
reviewer: Alessandro (Tech Lead)
---

## Routing to G5

**Branch:** `feature/iss-128-merge-check`
**G3 commit:** `8389f88 feat(process): add merge_check.py + po/fullstack agent updates -- ISS-128`
**Test result:** 184/184 passing (4 new tests in test_merge_check.py)

## Summary of changes

| File | Change |
|---|---|
| `merge_check.py` | New CLI script at project root: `--ticket ISS-NNN`, reads notes via `ticket.py view`, extracts branch via regex, checks `git branch --merged main`, exits 0/1 |
| `.github/agents/po.agent.md` | G7 Step 1 merge check prerequisite added |
| `.github/agents/fullstack.agent.md` | G6 note must end with merge commit hash |
| `tools/tests/test_merge_check.py` | 4 new tests (all subprocess-mocked) |

## Review scope for Alessandro

- `merge_check.py` design: subprocess usage, sys.executable for ticket.py call, regex `r"Branch\s+(feature/[^\s|]+)"`, exit codes
- `po.agent.md` and `fullstack.agent.md` scope changes are minimal and targeted
- Tests: correct use of monkeypatch for subprocess mocking
- No Django, no DB, no HTTP — pure CLI tooling
