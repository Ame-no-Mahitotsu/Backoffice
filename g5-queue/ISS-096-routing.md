# ISS-096 — G5 Routing Note

**Routed to:** Alessandro (Tech Lead) — G5 review

**Date:** 2026-03-20
**Run:** TL-2026-03-20-002
**Branch:** `feature/iss-096-sprint-status-links`
**Commit:** `33c8313`
**Status:** in-testing

## What was built
Conditional anchor rendering in `templates/sprint.html`: each non-zero status
count card now links to `/tickets?sprint=<sprint>&status=<status>`. Zero counts
remain plain text. Link styled with `color:inherit; text-decoration:none` so
appearance matches the existing count typography.

## Gate artifacts
- G1: `memory/workspace/tooling/gate-output/ISS-096-G1.md`
- G2: `memory/workspace/tooling/gate-output/ISS-096-G2.md` (red: 1 failed, 9 passed)
- G3: `memory/workspace/tooling/gate-output/ISS-096-G3.md` (green: 36 passed, 92% coverage)

## Files changed
- `templates/sprint.html` — conditional `<a>` tags in 4 status count cards + link CSS
- `tools/tests/test_dashboard_sprint.py` — 2 new tests

## Test results
36 passed, 0 failed. 10 sprint tests + 26 dashboard tests. Coverage 92%.

## Acceptance criteria
- AC1 ✅ Non-zero counts → `<a href="/tickets?sprint=...&status=...">` anchors
- AC2 ✅ Zero counts → plain text (no anchor)
- AC3 ✅ Link resolves to correct filtered ticket list (no backend change needed)
- AC4 ✅ All 34 pre-existing sprint + dashboard tests pass unchanged

## G6 reviewer
Chris (Senior Full Stack) — domain review for dashboard template changes.

## Notes for Alessandro
Template-only change. No Python logic modified. The `&amp;` encoding in the Jinja2
href is correct HTML (Flask renders it as the literal `&` in the browser).
AC3 (end-to-end: Sprint tab → Tickets tab filtered) depends on the `/tickets` route
accepting `?sprint=` and `?status=` query params — this was confirmed working in
`test_sprint_non_zero_status_count_is_a_link` via the seeded data test.
