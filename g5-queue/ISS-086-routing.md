---
ticket: ISS-086
title: test_dashboard_messages: fix seed status value from 'active' to valid MessageStatus enum value
branch: feature/iss-086-fix-seed-message-status
base_branch: main
worktree: —
run_id: TL-2026-03-19-001
routed_at: 2026-03-19
routed_by: Arale
---

ISS-086 is at `in-testing`. Branch `feature/iss-086-fix-seed-message-status` is ready for G5 review.

Pre-commit hooks: not configured (ISS-090 gap, known and owner-approved for this pilot run).
Tests green: 169/169 passed (full suite).
Coverage: 66% (no `fail_under` threshold configured).
Commit: `5e60d5b fix(process): replace invalid seed status 'active' with 'open' — ISS-086`

## What was done

`_seed_messages()` in `tools/tests/test_dashboard_messages.py` was inserting rows with `status='active'`, which is not a valid `MessageStatus` enum value. Three tuple values replaced with `'open'`. A new TDD test `test_seed_messages_status_is_valid_enum_value` was added to the same file — it was red before the fix and green after.

## AC verification (Arale)

- AC1: PASS — `_seed_messages()` uses `status='open'` in all three tuples
- AC2: PASS — 8/8 tests in `test_dashboard_messages.py` pass
- AC3: PASS — only `tools/tests/test_dashboard_messages.py` in the commit
