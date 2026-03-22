---
ID: session_architect_athena_001_storage_abstraction
Type: session
Role: architect
Topic: Storage abstraction principle — pm/ package must not couple to .md format
Date: 2026-03-15
Participants: Owner, Athena (external architect reviewer)
Ticket: ISS-018, ISS-017, ISS-027
Sprint: tools-1
Status: closed
Project: tooling
Outputs: Architectural finding — storage abstraction layer required; meeting with Alessandro + Patrick scheduled
---

## Context

Athena reviewed the tooling track architecture and identified a lost/missing decision: the `pm/` package must not be coupled to any specific storage format.

## Finding

The current implementation (ISS-017 — parsers.py, now resolved) bakes markdown table parsing directly into `MarkdownBacklogReader`. ISS-018 (writers.py) is about to do the same for writes.

**The principle stated:** writers and parsers must not assume storage format. The `pm/` package should be swappable between `.md` and `.json` (or other formats) without changing business logic.

## Architectural pattern implied

A storage abstraction layer — essentially a repository pattern:

```
CLI / API → pm/ business logic → BacklogStorage (interface)
                                       ↓              ↓
                                 md_storage.py   json_storage.py
```

## Immediate implications identified (by Owner, informed by Athena's review)

1. **ISS-017 at risk** — `parsers.py` was just delivered and closed. Was it written format-agnostically or does it hardcode markdown table parsing? Needs a check before ISS-018 starts.
2. **ISS-018 spec must be rewritten** — `append_ticket()` should call through a storage interface, not write markdown directly.
3. **ISS-027 Decision 4 (BacklogWriter Protocol) is incomplete** — it does not capture format abstraction.

## Next step

Three-way session: Alessandro (Tech Lead) + Patrick (Architect) + Athena (listening, will intervene if needed).
