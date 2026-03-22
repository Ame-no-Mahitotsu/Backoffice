---
ID: session_devops_salvatore_001_pm_repo
Type: session
Role: devops
Topic: Initialise git repository for SitoPresepi2 PM/tooling directory
Date: 2026-03-16
Participants: Owner, Salvatore
Ticket: ISS-036
Sprint: tools-1
Status: closed
Project: general
Outputs: repos/SitoPresepi2-pm.git (bare repo), SitoPresepi2/.git (working repo), .gitattributes, ai-standards.md updated (repo structure + PM branch model)
---

# Session: Salvatore (DevOps) — PM Repository Initialisation
**Ticket:** ISS-036
**Date:** 2026-03-16
**Status:** Closed — all gates passed

---

## What Was Done

The `SitoPresepi2/` project management directory had no version control. All governance
files (constitution, backlog, memory, agent definitions, tooling) were unprotected.

### Steps executed

1. Created bare repo: `c:\temp\ClaudeProjects\repos\SitoPresepi2-pm.git`
2. `git init` in `SitoPresepi2/` with remote pointing to bare repo
3. `.gitattributes` added — LF line ending normalisation for all text files
4. `.gitignore` confirmed correct (already existed, covered `__pycache__/`, `.coverage`, etc.)
5. Initial commit `e2263c3` on `main` — 84 files, 10171 insertions
6. Pushed to `origin/main`

### What is now versioned

All files in `SitoPresepi2/` except `__pycache__/`, `.coverage`, `.pytest_cache/`, `htmlcov/`,
`.venv/`, OS artefacts, and the `.claude/` auto-memory directory.

This includes: `docker-compose.yml`, `nginx/`, `memory/`, `tools/`, CLI scripts,
`.github/agents/`, `.github/copilot-instructions.md`, `CLAUDE.md`, `pyproject.toml`.

---

## Branching Strategy — SitoPresepi2 (PM repo)

**Approved: Option B** (owner decision, ISS-036 G1)

```
main        Stable. Direct commits permitted for trivial fixes (typos, date corrections).
feature/*   Branch from main for any non-trivial change. Merge to main via PR.
```

No `develop` branch — there is no deployment step for PM files.
Upgrade path to full GitFlow (Option C) is available if parallel PM work requires it.

---

## Changes to ai-standards.md

- Repository structure diagram updated: `SitoPresepi2-pm.git` added to `repos/`
- `SitoPresepi2/` description updated: "versioned" added
- Branch model section split: site code model and PM model documented separately

---

## Note for All Agents

**Effective immediately:** All changes to files in `SitoPresepi2/` must be committed.
- Non-trivial changes (new memory files, schema changes, tooling updates): use a `feature/` branch
- Trivial changes (typo fixes, date corrections): direct commit to `main` is permitted
- Commit format: Conventional Commits, same as `SitoPresepi2-src/`

The `.claude/` directory is intentionally excluded from version control (Claude auto-memory).

---

## Repositories — Full Reference

| Bare repo | Working clone | Branch model |
|---|---|---|
| `repos/SitoPresepi2.git` | `SitoPresepi2-src/` | Full GitFlow (main/develop/feature/hotfix) |
| `repos/SitoPresepi2-pm.git` | `SitoPresepi2/` | Lightweight (main/feature/*) |
