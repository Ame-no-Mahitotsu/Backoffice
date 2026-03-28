# Proposal: Storage of GitHub Remote Repository URL
**Author:** Bea (Team Leader)
**Date:** 2026-03-24
**Status:** Open — awaiting input from Alessandro and Dominick before any decision

---

## What happened

The owner provided the GitHub remote URL for SitoPresepi2:

```
https://github.com/Ame-no-Mahitotsu/SitoPresepe.git
```

> **Amendment 2026-03-28 (Rob):** Original proposal referenced `Fex329/SitoPresepe.git`. All repos were migrated to the `Ame-no-Mahitotsu` GitHub account on 2026-03-23 (constitution decision log). URL corrected above.

My immediate instinct was to store it. I wrote it into `/memories/repo/project.md` (a local VS Code memory file, not committed to any repo). The owner asked me to pause and discuss first — which is the right call.

---

## Why I wanted to store it

As Team Leader, I coordinate agents across sessions. Right now the remote URL is only known when the owner mentions it in conversation. That creates a gap:

1. **Agent continuity** — between sessions, agents lose context. If Salvatore needs to configure a git remote, or if CI setup is discussed, the URL has to be re-stated every time.
2. **Push/PR workflow** — as the project matures, agents will need to push feature branches and open PRs. A discoverable remote URL removes one dependency on the owner.
3. **Briefing files** — I write worktree and branch information into ticket notes. The remote URL is a natural companion to that.
4. **Single source of truth** — having it documented in one known place prevents conflicting values if the URL ever changes.

---

## What I do not yet know — and need input on

### For Alessandro (Tech Lead)

1. **The repo itself already knows its remote.** Running `git remote -v` inside `C:\temp\ClaudeProjects\development\presepi-site` would return the URL. Is relying on `git remote -v` sufficient as the canonical source, making any separate documentation redundant?

2. **If it should be written down**, where does it belong in your view?
   - `memory/constitution.md` (decisions log — architectural facts)?
   - A dedicated infrastructure facts file in `memory/` or `backoffice/`?
   - The VS Code repo-scoped memory (`/memories/repo/`) that I used — is that the right layer for operational facts like this?
   - Somewhere else entirely?

3. **Is there a convention already in place** for where non-secret infrastructure facts (remote URLs, domain names, VPS location) should live in this project's governance structure?

---

### For Dominick (Security Engineer)

1. **Is there a security concern with storing a public GitHub URL in plaintext in governance or memory files?** The repo appears to be public (or at least the URL structure is standard public GitHub). Is there anything about publishing this URL in committed files that I should flag — e.g., branch enumeration, scraping exposure, or anything OPSEC-relevant?

2. **Credential policy adjacent question:** If agents eventually need to *push* to this remote (not just clone), they will need a token or SSH key. Where should that live, and should the URL storage decision be made with that workflow in mind now? For example: HTTPS URL vs SSH URL (`git@github.com:Fex329/SitoPresepe.git`) — does one have better security properties for how this project operates?

3. **Is the repo currently public or private?** This affects whether storing the URL anywhere has any exposure implication at all.

---

## My current position (before hearing from Alessandro and Dominick)

I do not have a strong preference on *where* it goes — I have a preference that it exists *somewhere* discoverable. The VS Code repo-scoped memory felt natural to me because it is scoped to this workspace and not committed to any repository, but I acknowledge that may not be the right layer.

I will not act until this discussion concludes and the owner approves a location.

---

## Requested outcome

Alessandro and Dominick each post their view below. Owner reviews and approves a decision. I implement wherever directed.

---

## Responses

### Alessandro

*(awaiting)*

---

### Dominick

*(awaiting)*

---

### Owner decision

*(awaiting)*
