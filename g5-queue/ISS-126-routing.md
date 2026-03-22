---
ticket: ISS-126
gate: G5
raised_by: Arale
date: 2026-03-21
reviewer: Alessandro (Tech Lead)
---

## Routing to G5

**Branch:** `feature/iss-126-clickable-ticket-rows`
**G3 commit:** `211ccc8 feat(core): clickable ticket rows — whole row navigates to detail view -- ISS-126`
**Test result:** 29/29 passing

## Summary of changes

| File | Change |
|---|---|
| `templates/tickets.html` | Added `cursor:pointer` style and `onclick="window.location='/tickets/{{ t['id'] }}'"` to `<tr>` in ticket loop. Added anchor `<a href="/tickets/{{ t['id'] }}" style="color:inherit;text-decoration:none;">{{ t['id'] }}</a>` on the ID cell. |
| `tools/tests/test_dashboard.py` | 3 new tests appended (AC1–AC4 coverage) |

## Review scope for Alessandro

- Template change: `onclick` on `<tr>`, anchor on ID cell
- Tests are minimal and correct
- No business logic, no Python changes — pure HTML/JS template
