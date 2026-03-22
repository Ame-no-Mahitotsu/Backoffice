---
ID: session_scrum_rob_001_sprint_review_tools_
Type: session
Role: scrum
Topic: Sprint Review — tools-1 close
Date: 2026-03-16
Participants: Owner, Rob (SM), Sofia (PO), Claire (implementer), Alessandro (TL), Chris (domain)
Ticket: ISS-040,ISS-041,ISS-042,ISS-043,ISS-044,ISS-045,ISS-046,ISS-047,ISS-048,ISS-049,ISS-050
Sprint: tools-1
Status: closed
Project: tooling
Outputs: Sprint Review minutes, tools-1 formally closed
---

## Sprint Review — tools-1
**Date:** 2026-03-16
**Facilitator:** Rob (Scrum Master)
**Attendees:** Owner, Rob (SM), Sofia (PO), Claire (implementer), Alessandro (TL), Chris (domain reviewer)
**Time-box:** 60 minutes

---

## Sprint Goal (as committed)

Deliver a robust, test-covered PM tooling layer: replace fragile markdown-table storage with SQLite, complete the CLI (ticket.py + message.py), establish git versioning for the PM repo, and implement the AI handoff protocol.

---

## Increment Demo — What Was Built

### SQLite Migration (ISS-040 through ISS-050)

The core deliverable of tools-1. The markdown table storage backend — which suffered from pipe-escaping failures, silent row drops, and column drift — has been fully replaced with SQLite via Python stdlib `sqlite3`. Zero new dependencies.

| Ticket | Deliverable | Status |
|--------|-------------|--------|
| ISS-040 | `db_init.py` — `create_schema()` creates `tickets` + `messages` tables with PRIMARY KEY constraints | resolved |
| ISS-041 | `sqlite_reader.py` — `SQLiteBacklogReader` implementing `BacklogReader` protocol | resolved |
| ISS-042 | `sqlite_writer.py` — `SQLiteBacklogWriter` implementing `BacklogWriter` protocol | resolved |
| ISS-043 | `sqlite_message_writer.py` — `SQLiteMessageWriter` implementing `MessageWriter` protocol; session files remain `.md` | resolved |
| ISS-044 | `migrate_md_to_sqlite.py` — one-time migration script; all historical records migrated to `memory/pm.db` | resolved |
| ISS-045 | `ticket.py` wired to SQLite — `MarkdownBacklogReader/Writer` removed from CLI | resolved |
| ISS-046 | `message.py` wired to SQLite — `_next_msg_id` rewritten as SQLite MAX query | resolved |
| ISS-047 | `issue_reader.py` + `test_issue_reader.py` deleted — stale markdown reader removed | resolved |
| ISS-050 | `sqlite_utils.py` — `_connect()` extracted; all four SQLite modules import from single source | resolved |

**Evidence:** 102/102 tests passing at sprint close. Feature branch `feature/sqlite-migration` merged to `main` via `--no-ff` merge commit. `memory/pm.db` gitignored (runtime artefact).

---

### CLI Enhancements

| Ticket | Deliverable | Status |
|--------|-------------|--------|
| ISS-012 | `ticket.py update` subcommand — transition constraints, immutable fields, `--caller` flag | resolved |
| ISS-019 | `ticket.py create` subcommand | resolved |
| ISS-020 | `message.py create` subcommand | resolved |
| ISS-048 | `handoff.py` improvements — entry separator, whitespace-only file test, 3-entry ordering test | resolved |
| ISS-049 | `sqlite_writer.py` field allowlist — `_MUTABLE_FIELDS` guard prevents SQL f-string injection | resolved |

---

### Infrastructure & Process

| Ticket | Deliverable | Status |
|--------|-------------|--------|
| ISS-036 | PM repo git-initialised; bare repo at `repos/SitoPresepi2-pm.git`; branching strategy documented | resolved |
| ISS-037 | `handoff.py` — structured append-only AI handoff writer with required-field validation | resolved |
| ISS-025 | `dispatch_reader.py` tests added | resolved |
| ISS-027 | SOLID audit — protocols.py approved; storage abstraction as design invariant | resolved |
| ISS-029 | pm/ restructure — `protocols.py` + `md_reader.py` + `md_writer.py` | resolved |

### Wont-fix (2 tickets — superseded by SQLite migration)

| Ticket | Reason |
|--------|--------|
| ISS-026 | `issue_reader.py` tests — file deleted in ISS-047; moot |
| ISS-034 | `handle_create` unescaped pipe separator — markdown format retired; defect no longer exists |

---

## Final Ticket Count

| Status | Count |
|--------|-------|
| resolved | 28 |
| wont-fix | 2 |
| slipped to tools-2 | 1 (ISS-038) |
| **Total sprint tickets** | **31** |

---

## Slipped to tools-2

| Ticket | Reason |
|--------|--------|
| ISS-038 | `ticket.py create --session` flag — low urgency; site track held open too long to wait for it |

---

## Retrospective Items Noted During Review

- Gate protocol skipping by Claire (Copilot) — agent file reinforced during sprint; carry to retro
- Branching not followed — all SQLite work committed directly to `main`; retroactive branch created end of sprint; carry to retro
- `ticket.py` and `message.py` wired to SQLite outside the gate process — owner did this directly; acknowledged

---

## Actions

- [ ] Sofia: post PO acceptance below
- [ ] Rob: facilitate Retrospective immediately after
- [ ] Rob: open tools-2 sprint planning once retro complete
- [ ] Site track: ISS-006 returns to `ready` — Lota to resume IA / mood boards

---

## PO Sign-off

**Sofia — 2026-03-16**

Sprint goal ACCEPTED in full against all four commitments:

- ✅ SQLite storage replacement — ISS-040 to ISS-050, root cause eliminated architecturally
- ✅ CLI complete — ticket.py create + update, message.py create, all wired to SQLite
- ✅ Git versioning — PM repo initialised, branching strategy documented, retroactive branch handled
- ✅ AI handoff protocol — handoff.py delivered with ISS-048 improvements in same sprint

ISS-038 slip to tools-2 accepted. Process gaps (gate skipping, branching) noted for retro — do not affect product value.

**Awaiting Owner final sign-off to formally close tools-1.**

---

## Owner Sign-off

**Owner — 2026-03-16:** Signed off. tools-1 formally closed. Retrospective to follow.

