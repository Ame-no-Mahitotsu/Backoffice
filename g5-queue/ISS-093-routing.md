---
ticket: ISS-093
title: Recommend and document secrets scanner config for pre-commit (detect-secrets vs gitleaks)
branch: feature/iss-093-secrets-scanner-recommendation
base_branch: main
run_id: manual
routed_at: 2026-03-19
routed_by: Dominick
---

ISS-093 is at `in-testing`. Branch `feature/iss-093-secrets-scanner-recommendation` is ready for G5 review.

This is a documentation-only ticket. G2 (red tests) was not applicable — flagged in G1 and G2 output files.
Commit: `b52de00 docs(security): secrets scanner recommendation — ISS-093`

## What was done

Evaluated `detect-secrets` vs `gitleaks` for pre-commit secrets scanning.
Recommendation: `detect-secrets==1.5.0` — Python-native, pip-installable, native pre-commit hook, proportionate to project scale.

Deliverable: `memory/workspace/shared/secrets_scanner_recommendation.md` — includes tool decision rationale, step-by-step ISS-090 implementation guide (pyproject.toml additions, .pre-commit-config.yaml snippet, baseline generation, false-positive analysis for this repo), and baseline management policy.

ISS-090 is unblocked.

## AC verification (Dominick)

ISS-093 has no formal numbered ACs. Stated deliverable from ticket notes:
- Tool choice: PASS — detect-secrets==1.5.0 selected, rationale documented
- Baseline configuration: PASS — full baseline generation steps provided
- Project-specific exclusion patterns: PASS — docker-compose.yml and test file false positives analysed; baseline strategy documented
- Salvatore-implementable output: PASS — step-by-step commands, config snippets, pyproject.toml additions all in the recommendation doc

Note for Alessandro (G5): Please confirm that G2 N/A is acceptable for a documentation/research ticket of this type, or raise a process ticket if a different convention should apply.
