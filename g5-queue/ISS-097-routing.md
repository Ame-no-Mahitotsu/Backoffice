# ISS-097 — G5 Routing Note

**Routed to:** Alessandro (Tech Lead) — G5 review

**Date:** 2026-03-20
**Run:** TL-2026-03-20-002
**Branch:** `feature/iss-097-ticket-detail-view`
**Base branch:** `feature/iss-096-sprint-status-links` (ISS-097 branches from ISS-096
to include ISS-096's test_dashboard_sprint.py changes and avoid test_dashboard.py conflicts)
**Commit:** `7003675`
**Status:** in-testing

## What was built
New Flask route `GET /tickets/<ticket_id>` in `dashboard.py`, new template
`templates/ticket_detail.html` displaying all ticket fields, and a "View" link
column added to `templates/tickets.html`. 4 new tests in `test_dashboard.py`.

## Gate artifacts
- G1: `memory/workspace/tooling/gate-output/ISS-097-G1.md`
- G2: `memory/workspace/tooling/gate-output/ISS-097-G2.md` (red: 3 failed, 1 passed)
- G3: `memory/workspace/tooling/gate-output/ISS-097-G3.md` (green: 40 passed, 93% coverage)

## Files changed
- `dashboard.py` — `abort` import + new `ticket_detail` route
- `templates/ticket_detail.html` — new template (all ticket fields)
- `templates/tickets.html` — View link column added
- `tools/tests/test_dashboard.py` — 4 new tests

## Test results
40 passed, 0 failed. All 36 pre-existing tests pass (AC5).
Coverage: dashboard.py 93%.

## Acceptance criteria
- AC1 ✅ View link → `/tickets/<id>` detail page (test: happy path + view link)
- AC2 ✅ All 17 ticket fields displayed
- AC3 ✅ Back link uses `request.referrer or "/tickets"`
- AC4 ✅ `abort(404)` for missing ID (test: 404 confirmed)
- AC5 ✅ 4 new tests (≥ 2 required); all 36 existing tests pass

## G6 reviewer
Chris (Senior Full Stack) — domain review for new Flask route + templates.

## Notes for Alessandro
ISS-097 branched from ISS-096's feature branch (not main) to carry ISS-096's
`sprint.html` and `test_dashboard_sprint.py` changes forward. The `ticket_detail`
route uses `sqlite3.Row` factory (inherited from `_get_db()`) so template access
via `ticket['field']` is correct. No business logic in the view — all query logic
is in the route handler using the shared `_get_db()` pattern already established
in the codebase. Review focus: confirm `abort(404)` is the correct Flask idiom here,
and that the Back link preserving Referer is acceptable (vs URL query param forwarding).
