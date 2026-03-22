---
run_id: TL-2026-03-21-001
sprint: tools-7
intent: "PM Dashboard clickable rows (ISS-126), sprint page ticket list (ISS-127), G6 merge verification (ISS-128)"
created_by: Rob
created_at: 2026-03-21T00:00:00
last_updated: 2026-03-21T09:00:00
last_updated_by: Arale
run_status: IN-PROGRESS
interrupted_reason: —
venv_path: c:/temp/ClaudeProjects/SitoPresepi2/.venv/Scripts/activate
---

# Arale Briefing — tools-7 Run 001

## Scope

Three tickets. All owned by Claire. Two components: `core` (dashboard) and `process` (merge check tooling).

```
Wave 1 — PARALLEL:
  ISS-126  clickable ticket rows (tickets.html) — core/dashboard
  ISS-128  merge_check.py + po/fullstack agent file updates — process

Wave 2 — SEQUENTIAL (after ISS-126 fully resolved):
  ISS-127  sprint page ticket list — core/dashboard — blocked-by ISS-126
```

ISS-128 is fully independent of ISS-126 and ISS-127 (different files, different component). Run in parallel with ISS-126. ISS-127 must wait for ISS-126 — it depends on the clickable-row pattern established there.

---

## Ticket Table

| Field | ISS-126 | ISS-128 | ISS-127 |
|---|---|---|---|
| `id` | ISS-126 | ISS-128 | ISS-127 |
| `title` | PM Dashboard: clickable ticket rows — whole row links to detail view | G6 gate: merge verification check — detect unmerged feature branches | PM Dashboard: sprint page — ticket list for selected sprint with clickable rows |
| `type` | story | improvement | story |
| `component` | core | process | core |
| `sprint` | tools-7 | tools-7 | tools-7 |
| `working_dir` | c:/temp/ClaudeProjects/SitoPresepi2 | c:/temp/ClaudeProjects/SitoPresepi2 | c:/temp/ClaudeProjects/SitoPresepi2 |
| `base_branch` | main | main | main |
| `wave` | 1 | 1 | 2 |
| `batch_claim` | Arale-TL-2026-03-21-001 | Arale-TL-2026-03-21-001 | Arale-TL-2026-03-21-001 |
| `current_status` | in-development | in-development | in-development |

---

## Notes Snapshots (verbatim from pm.db at briefing creation)

### ISS-126
```
2026-03-21: As a user on any ticket list in the dashboard, I can click anywhere on a ticket row to open the detail view at /tickets/<id>, not just the View button. Applies to tickets.html. AC1: clicking a row navigates to /tickets/<id>. AC2: View button still works. AC3: cursor:pointer on hover. AC4: at least 1 test. | 2026-03-21: Agent Execution Protocol applies — all 7 gates are owner-blocking. Escalation gate applies. See ai-standards.md Agent Execution Protocol.
```

### ISS-128
```
Root cause: ISS-095 G6 approved but --no-ff merge never executed. Arale agent files lost until manually recovered. Design agreed 2026-03-21: (A) deliver merge_check.py script; (C) Sofia runs it at G7 Step 1 — if fail, back to Chris to fix the merge, not escalated to owner directly. Instructions go in po.agent.md and fullstack.agent.md only — do NOT touch ai-standards.md (avoids bloating other agents context). AC1: merge_check.py accepts --ticket ISS-NNN. AC2: reads branch name from G1 note in ticket notes field. AC3: checks git branch --merged main for that branch. AC4: exit 0 if merged, exit 1 with clear message naming the unmerged branch. AC5: at least 2 tests. AC6: po.agent.md updated — G7 Step 1 first action is run merge_check.py; if exit 1, Sofia posts a merge-missing notice back to Chris and halts; she does NOT post PO acceptance until check passes. AC7: fullstack.agent.md updated — G6 note must end with the merge commit hash from git log --oneline -1 main after the merge.
```

### ISS-127
```
2026-03-21: As a user on the Sprint page, I can see the full list of tickets for the selected sprint below the status cards, and click any row to open the ticket detail view. AC1: ticket list renders below progress bar for selected sprint. AC2: all statuses shown (not filtered like status cards). AC3: each row links to /tickets/<id>. AC4: empty state message if no tickets. AC5: at least 2 tests (list renders, row link correct). | 2026-03-21: Agent Execution Protocol applies — all 7 gates are owner-blocking. Escalation gate applies. See ai-standards.md Agent Execution Protocol.
```

---

## Allowed Paths

### ISS-126 (component: core)
```
templates/tickets.html
tools/tests/
```

### ISS-128 (component: process)
```
merge_check.py            ← new file at project root
.github/agents/po.agent.md
.github/agents/fullstack.agent.md
tools/tests/
```

### ISS-127 (component: core)
```
dashboard.py
templates/sprint.html
tools/tests/
```

Cross-cutting files (`conftest.py`, `pyproject.toml`) permitted only if explicitly required. Do not touch `memory/`, `tools/pm/`, `ticket.py`, `message.py`, or any agent file not listed above.

---

## Current State — Key Files to Read First

### For ISS-126
- `templates/tickets.html` — existing ticket table. Rows have a `View` link (`<a href="/tickets/{{ t['id'] }}">`). Make the entire row clickable.
- `tools/tests/test_dashboard.py` — use `seeded_client` fixture pattern.

### For ISS-128
- `tools/tests/` — for test structure reference.
- `.github/agents/po.agent.md` — Sofia's G7 section; add merge check as Step 1 prerequisite.
- `.github/agents/fullstack.agent.md` — Chris's G6 section; add merge commit hash requirement to G6 note.
- `ticket.py` — understand how to read ticket notes field (to extract branch name from G1 note). Use `python ticket.py view ISS-NNN` output, not direct DB access.

### For ISS-127 (read after ISS-126 is resolved)
- `dashboard.py` — `/sprint` route returns status counts only; add ticket query for selected sprint.
- `templates/sprint.html` — add ticket list below progress bar, reusing row style from ISS-126.
- `templates/ticket_detail.html` — read-only reference.

---

## Implementer Assignment

All tickets → **Claire — Backend Developer (Automated)**

---

## Gate Output Paths

| Ticket | G1 | G2 | G3 | G5 | G6 |
|---|---|---|---|---|---|
| ISS-126 | `memory/workspace/tooling/gate-output/ISS-126-G1.md` | `memory/workspace/tooling/gate-output/ISS-126-G2.md` | `memory/workspace/tooling/gate-output/ISS-126-G3.md` | `memory/workspace/tooling/gate-output/ISS-126-G5.md` | `memory/workspace/tooling/gate-output/ISS-126-G6.md` |
| ISS-128 | `memory/workspace/tooling/gate-output/ISS-128-G1.md` | `memory/workspace/tooling/gate-output/ISS-128-G2.md` | `memory/workspace/tooling/gate-output/ISS-128-G3.md` | `memory/workspace/tooling/gate-output/ISS-128-G5.md` | `memory/workspace/tooling/gate-output/ISS-128-G6.md` |
| ISS-127 | `memory/workspace/tooling/gate-output/ISS-127-G1.md` | `memory/workspace/tooling/gate-output/ISS-127-G2.md` | `memory/workspace/tooling/gate-output/ISS-127-G3.md` | `memory/workspace/tooling/gate-output/ISS-127-G5.md` | `memory/workspace/tooling/gate-output/ISS-127-G6.md` |

---

## G5 Routing Note Paths

| Ticket | Path |
|---|---|
| ISS-126 | `memory/workspace/tooling/g5-queue/ISS-126-routing.md` |
| ISS-128 | `memory/workspace/tooling/g5-queue/ISS-128-routing.md` |
| ISS-127 | `memory/workspace/tooling/g5-queue/ISS-127-routing.md` |

---

## Reviewer Assignments

| Gate | Reviewer | ISS-126 | ISS-128 | ISS-127 |
|---|---|---|---|---|
| G5 | Alessandro (Tech Lead) | ✓ | ✓ | ✓ |
| G6 | Chris (Senior Full Stack) | ✓ | ✓ | ✓ |

Standard model — Claire implements, no conflict.

---

## Notes for Arale

- **Wave 1: ISS-126 and ISS-128 run in parallel.** They touch entirely different files. Start both immediately after owner approves this briefing.
- **Wave 2: ISS-127 must not start until ISS-126 is `resolved` and its branch is merged to main.** ISS-127 reuses the clickable-row style established in ISS-126.
- **ISS-128 branch name extraction:** the G1 note for any ticket contains the branch name in the format `Branch feature/iss-NNN-short-description from <base>`. Parse this from `python ticket.py view ISS-NNN` output. Do not query the DB directly.
- **ISS-128 scope is strictly limited** to `merge_check.py`, `po.agent.md`, `fullstack.agent.md`, and tests. Do not update `ai-standards.md` or any other agent file.
- ISS-126 uses JavaScript `onclick` on `<tr>` — do not restructure the table.
- ISS-127 backend change: pass ticket list to sprint template; template change: add table below progress bar.
- The `/tickets/<ticket_id>` route is already live (ISS-097). Do not touch it.
- `detect-secrets` pre-commit hook is active. Do not commit secrets.
- Do not touch site-track files (`SitoPresepi2-src/`) or any ticket not in this briefing.
