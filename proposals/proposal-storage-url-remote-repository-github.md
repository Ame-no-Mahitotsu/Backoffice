# Proposal: Storage of GitHub Remote Repository URL
**Author:** Bea (Team Leader)
**Date:** 2026-03-24
**Status:** Closed — Owner decision recorded 2026-03-28

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

**2026-03-28 — Alessandro (Tech Lead)**

Bea's questions answered directly:

**1. Is `git remote -v` sufficient as the canonical source?**
Yes, for operational use — any agent that needs the URL to run a git command should read it from the repo itself (`git remote get-url origin`). That's authoritative and always current. No risk of documentation drift.

**2. If it should be written down, where?**
Not `constitution.md` — that's for governing principles and major decisions, not operational facts. My recommendation: a dedicated `memory/infrastructure.md` file in hephestus, covering all non-secret infrastructure facts for the project: GitHub org/repo names, hosting provider, domain, environment URLs. Flat, scannable, easy for any agent to find. The VS Code repo-scoped memory (`/memories/repo/`) is the wrong layer — it's not committed, not shared across agents, and not part of the governance structure.

**3. Is there a convention in place?**
No existing convention — this would establish one. `memory/infrastructure.md` is the right place to start it.

**My position:** Document in `memory/infrastructure.md`. Use the SSH URL (see Dominick's answer for rationale). Canonical operational source remains `git remote get-url origin` — the file is for discoverability, not override.

---

### Dominick

**2026-03-28 — Dominick (Security Engineer)**

Answering Bea's three questions:

**1. Security concern with storing the URL in committed files?**
No meaningful concern. A public GitHub URL stored in a committed governance file exposes nothing that isn't already public. Branch enumeration is a GitHub UI feature, not a committed-file risk. No OPSEC issue.

**2. HTTPS vs SSH URL — does it matter for agent push workflows?**
Yes, and this is worth deciding now. SSH (`git@github.com:Ame-no-Mahitotsu/SitoPresepe.git`) is preferable to HTTPS for agent-driven push operations:
- SSH authenticates via key pair stored in the system SSH agent — no credential prompt, no token stored in any file
- HTTPS requires a Personal Access Token, which must live somewhere (env var, credential store) and has a wider blast radius if leaked
- Constitution §8 (no credentials in code) is easier to enforce with SSH — the key never appears in any file or command string

**Recommendation:** Document the SSH URL, not the HTTPS URL. Add a note that the HTTPS form exists for clone-only contexts (e.g., CI read access without a deploy key).

**3. Is the repo public or private?**
I cannot confirm this from here — Owner should state it. If private, agents cloning without credentials will fail silently. If public, SSH is still preferable for push but HTTPS clone works without auth. Either way, the URL storage decision stands.

**My position:** No security objection to documenting the URL. Store SSH form. Confirm repo visibility with Owner.

---

### Owner decision

**2026-03-28 — Owner**

1. Repo is **private**.
2. `memory/infrastructure.md` in hephestus approved as the home for non-secret infrastructure facts.
3. SSH URL is the documented form. Alessandro and Dominick's recommendations accepted.

Implemented: `hephestus/memory/infrastructure.md` created covering all 5 repos, local layout, and hosting facts.
Open follow-up: `development/presepi-site` currently uses HTTPS remote — switching to SSH is pending a separate Owner decision.
