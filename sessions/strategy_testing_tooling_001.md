# Testing Strategy — Tooling Track
> Created: 2026-03-14 | Author: Lauren (SDET)
> Project: tooling
> Status: approved
> Last amended: 2026-03-14 — Python version updated to 3.14.3; venv setup section added; Known Blockers updated

## Scope
Pure Python tooling package (`pm/` and CLI scripts). No Django, no database, no HTTP. All tests in this track are **unit tests**. Playwright / E2E / accessibility testing applies to the `site-N` sprint track only.

---

## Python Version
**Python 3.14.3** — approved 2026-03-14 (Constitution decisions log). Single version for all project Python: tooling track (host) and site track (Docker image).

**Django constraint:** Django must be pinned to `>=5.2.8` in the site track. Earlier Django 5.x releases do not support Python 3.14.

---

## Environment Setup (pre-TASK-A, owner action — ISS-024)

Before any test can run, the following must be in place on the host machine:

```bash
# 1. Install Python 3.14.3 on host (windows.python.org or winget)

# 2. Create venv at SitoPresepi2/ root
cd c:\temp\ClaudeProjects\SitoPresepi2
py -3.14 -m venv .venv

# 3. Activate venv
.venv\Scripts\activate      # Windows CMD/PowerShell
# or
source .venv/Scripts/activate  # Git Bash

# 4. Install test dependencies
pip install pytest pytest-cov

# 5. Verify (failures expected — green not required at this stage)
python -m pytest memory/tests/test_ticket_schema.py -v
```

Versions are pinned in `pyproject.toml` once TASK-A creates it. Until then, installed versions are acceptable for unblocking the sprint.

**All canonical invocations below assume the venv is activated.**

---

## Framework
**pytest** — approved 2026-03-12 (Constitution decisions log).
**pytest-cov** — approved 2026-03-14 for coverage enforcement. Security conditions (Dominick, 2026-03-14):
- Pin exact version in `pyproject.toml` (supply chain control)
- Declared under `[project.optional-dependencies] dev` only — never in production Docker image
- `.coverage` and `htmlcov/` added to `.gitignore`

---

## Test Location

| Phase | Test path | Code path |
|-------|-----------|-----------|
| Now (pre TASK-A) | `memory/tests/` | `memory/ticket_schema.py` |
| Post TASK-A | `tools/tests/` | `tools/pm/` + CLI scripts |

Migration from `memory/tests/` to `tools/tests/` is tracked in `memory/ticket_schema.py` (migration note) and will execute as part of TASK-A.

---

## Pytest Configuration
Lives in `pyproject.toml` at the project root (created in TASK-A alongside Black/isort/Ruff config).

```toml
[tool.pytest.ini_options]
testpaths = ["memory/tests"]   # updated to ["tools/tests"] after TASK-A migration
pythonpath = ["."]             # makes `from memory.ticket_schema import ...` resolve
```

After TASK-A migration, `testpaths` is updated to `["tools/tests"]`.

---

## Canonical Invocation

```bash
# Run all tests
python -m pytest

# Run with coverage (required before any PR)
python -m pytest --cov=memory --cov=tools --cov-fail-under=80

# Run a single test file
python -m pytest memory/tests/test_ticket_schema.py -v
```

---

## Fixtures
- `conftest.py` at `memory/tests/conftest.py` — create now, empty. Anchor for future shared fixtures (sample ticket dicts, temp file paths).
- After TASK-A migration: second `conftest.py` at `tools/tests/conftest.py`.
- No project-root `conftest.py` for this track.

---

## Naming and Structure

**Test naming** (from ai-standards.md):
```
test_<subject>_<condition>_<expected_result>
```

**Test structure:** Arrange / Act / Assert. No exceptions.

**File naming:** `test_<module_name>.py` mirroring the module under test.

---

## What to Test per TASK

| TASK | Behaviour to prove |
|------|--------------------|
| A — pm/ package setup | Package imports without error; `__init__.py` exports correct names |
| B — pm/parsers.py | Each parser returns the correct Python type; malformed input raises a named exception (not generic `Exception`); field mapping is exact |
| C — pm/writers.py (create ops) | File is created with correct content; no file created on validation failure; file not overwritten if already exists (where applicable) |
| D — create_ticket.py | CLI exits 0 on valid input; correct fields written to file; exits non-zero with clear stderr on invalid input; `--caller` flag written in correct format |
| E — create_message.py | Same pattern as TASK-D |
| F — pm/writers.py (update ops) | Existing file updated correctly; original fields untouched; Notes field appends, does not overwrite |
| ISS-012 — update_ticket.py | `--caller` flag written; status transitions validated against enum; invalid status rejected with non-zero exit |

**What NOT to test:**
- OS-specific path separators — use `pathlib.Path` throughout, tests must be OS-agnostic
- Internal enum string formatting implementation details
- `print()` output formatting (test behaviour, not presentation)

---

## Coverage Gate
**80% line coverage minimum** across `memory/` and `tools/`.

- Below 80% → PR cannot merge. No exceptions, no deferrals.
- Enforced via `--cov-fail-under=80` in the canonical invocation above.
- Coverage report format: terminal summary only (no HTML artefacts committed).

---

## Definition of Done — Tooling Ticket
- [ ] Failing test written and committed **before** any implementation code (TDD, Constitution §6)
- [ ] Test name follows `test_<subject>_<condition>_<expected_result>`
- [ ] Test proves the specific behaviour named in the acceptance criterion — not just "it runs"
- [ ] Coverage gate passes (≥80%)
- [ ] All pre-existing tests still pass (no regression)
- [ ] Tech Lead (Alessandro) review approved
- [ ] Domain specialist review approved (Backend/Fullstack as applicable)
- [ ] PO acceptance confirmed before ticket moves to `resolved`

---

## Known Blockers

Four issues must be resolved before the existing test in `memory/tests/` can run. Build order is sequential.

| # | Ticket | Issue | Fix | Status |
|---|--------|-------|-----|--------|
| 0 | ISS-024 | Python 3.14.3 not installed; no venv; pytest not installed | Owner: install Python 3.14.3, create `.venv`, install pytest + pytest-cov | ready |
| 1 | ISS-022 | `memory/__init__.py` missing — ensures explicit package declaration for deterministic pytest import behavior under `--import-mode=prepend` | Add empty `memory/__init__.py` | ready |
| 2 | ISS-023 | No `conftest.py` in `memory/tests/` — fixture anchor required | Create empty `memory/tests/conftest.py` | ready |
| 3 | ISS-016 | No pytest config — rootdir detection unreliable | Create `pyproject.toml` with `[tool.pytest.ini_options]` (TASK-A) | ready |

ISS-024 is owner-action only. ISS-022 and ISS-023 are pre-TASK-A fixes. ISS-016 (TASK-A) resolves blocker 3.

**Note on ISS-022:** The original `ModuleNotFoundError` rationale was incorrect — Python 3.14 namespace packages allow the import without `__init__.py`. The file is still required for pytest import mode determinism. See ISS-022 ticket notes for full correction.

---

## Out of Scope (this track)
- Playwright / E2E — site track only
- Accessibility (WCAG 2.1 AA) — site track only
- Jest / React Testing Library — site track only
- Django `pytest-django` — site track only
- CI/CD gates — deferred until test suite matures (Constitution decisions log 2026-03-12)
