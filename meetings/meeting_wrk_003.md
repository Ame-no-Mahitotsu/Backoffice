# Meeting Minutes — WRK-003

ID: WRK-003
Type: working-session
Tags: permissions, ticket-automation, agent-permissions, arale, vscode, settings
Date: 2026-03-23
Chair: Rob (Scrum Master)
Participants: Rob (SM), Dominick (Security), Owner
Ticket: —
Sprint: tools-8
Status: closed
Project: tooling

---

## Purpose

Two inter-related problems to resolve in one session:

1. **`agent_permissions.md` is missing from the active repos.** The file was authored during tools-4 (ISS-083) but was not migrated from `SitoPresepi2-archive/memory/` to `hephestus/memory/` during reorg-1. The `friendly_agreement.md` references it as a required read — that reference is currently a dead link. The policy must be rehoused and updated for the new multi-repo structure.

2. **Ticket automation is blocked by the current permission policy.** Arale (`operation-manager.agent.md`) runs `ticket.py update`, `ticket.py create`, and status transitions as part of its unattended batch loop. The current policy (Rule 2a in `agent_permissions.md`) classifies every ticket write operation as Owner-prompt-required. This is incompatible with batch automation — the Owner would be interrupted on every gate transition. The policy needs a scoped automation allowance that preserves the control model without blocking Arale.

---

## Agenda

### Item 1 — Permissions file: status and migration

- Confirm whether the archived policy (`SitoPresepi2-archive/memory/agent_permissions.md`) is still current, or whether the reorg has already broken its assumptions
- Identify every path reference in the policy that has changed since reorg (e.g. `tools/tests/` → `development/tools/tools/tests/`, `memory/pm.db` → `development/tools/pm.db`)
- Decide: migrate-and-update in place, or rewrite from scratch?
- Assign authorship (Dominick) and a target ticket

### Item 2 — Ticket write automation: what can Arale auto-approve?

The question to answer: **which `ticket.py` write operations can be auto-approved when the caller is `operation-manager` or an Arale-dispatched automated subagent?**

Boundary conditions to discuss:

| Command | Current policy | Proposed for discussion |
|---|---|---|
| `ticket.py update ISS-NNN --field status --value in-development --caller operation-manager` | Owner prompt required | Auto-approve? (batch claim — Arale owns it) |
| `ticket.py update ISS-NNN --field notes --value "..." --caller backend-automated` | Owner prompt required | Auto-approve? (gate note — evidence only) |
| `ticket.py update ISS-NNN --field status --value in-testing --caller operation-manager` | Owner prompt required | Auto-approve? (G3 verified — Arale advancing) |
| `ticket.py create --type defect ...` (discovered during batch) | Owner prompt required | Owner prompt still required? (new record, not part of plan) |
| `ticket.py update ISS-NNN --field status --value ready` | Owner prompt required | Owner prompt still required (only Owner sets ready) |

Rob's initial position: the control point should be **gate 4 (Sofia/PO acceptance) and gate 7 (Owner sign-off)** — not individual ticket.py writes inside the batch. Arale is already governed by the prime directive, the batch_claim field, and the pre-flight checks. Blocking every write wastes Owner attention on things Arale is designed to own.

Dominick's input required: what is the minimum control model that preserves auditability without requiring Owner prompts inside the batch loop?

### Item 3 — VSCode `settings.json` changes needed

Once Items 1 and 2 are resolved, identify any `settings.json` changes required to implement the agreed policy. These must be proposed as a ticket (or added to an existing one) — no agent self-modifies `settings.json`.

---

## Pre-Meeting Reads (for all participants)

- `SitoPresepi2-archive/memory/agent_permissions.md` — current (archived) policy
- `hephestus/memory/friendly_agreement.md` — dead reference to note
- `hephestus/.github/agents/operation-manager.agent.md` — Arale's batch protocol and pm.db write requirements
- `hephestus/.github/agents/backend-developer-automated.agent.md` — example subagent ticket write pattern

---

## Outcomes Expected

1. Decision: migrate + update vs. rewrite `agent_permissions.md`
2. Agreed allowlist for Arale-context ticket writes (auto-approve conditions)
3. Ticket(s) raised for implementation
4. `friendly_agreement.md` reference to `agent_permissions.md` confirmed (once file is live in `hephestus/memory/`)

---

## Minutes

### 2026-03-23 — Owner opening statement

Owner described the lived experience of working with Claire in interactive sessions:

> "When I assign tickets to Claire I have to authorise all her work in git, and on the ticket, I also have to authorise the creation and editing of files. Even with Claire's profile this makes the experience quite stuttering."

Three distinct friction points identified:

| Friction point | Current behaviour |
|---|---|
| File creation and editing (Edit/Write tools) | Owner prompted on every file touch |
| Git operations (add, commit, push, checkout) | Owner prompted on each write-side command |
| Ticket operations (ticket.py update/create) | Owner prompted on each write |

**Rob's observation:** This is not a permissions policy problem in isolation — it is a tool approval model problem. The `.vscode/settings.json` `chat.tools.terminal.autoApprove` controls terminal commands, but file edits (Edit/Write tool calls in Claude Code) are governed separately by VS Code's tool approval settings. All three friction points need to be addressed together.

**Key context for Dominick's threat model:** The Owner is a solo technical operator. There is no team of agents operating unsupervised — every Claire session is Owner-initiated, Owner-scoped, and Owner-closed. The risk profile is fundamentally different from a shared-team environment.

---

### 2026-03-23 — Agreed decisions

#### Layer model (three distinct permission layers)

| Layer | Mechanism | Scope |
|---|---|---|
| File operations (Edit/Write) | Claude Code tool approval model | Auto-approve within workspace |
| Terminal commands | `chat.tools.terminal.autoApprove` in settings.json | Expanded per policy below |
| Git write operations | Terminal auto-approve | Selective — see below |

#### Terminal auto-approve: agreed allowlist additions

| Command | Decision | Condition |
|---|---|---|
| `git add` + `git commit` to feature branches | Auto-approve | Feature branch only — not main/develop |
| `git push` | Keep Owner prompt | Externally visible, hard to reverse |
| `git merge / reset / force` | Keep Owner prompt | Destructive or irreversible |
| `ticket.py update` (status + notes) | Auto-approve | Within batch lifecycle only |
| `ticket.py update --field status --value ready` | Auto-approve | Caller: `scrum` only |
| `ticket.py update --field status --value in-development` | Auto-approve | Caller: `tech-lead` only (G5 rejection) |
| `ticket.py create` | Keep Owner prompt | New scope — always Owner decision |
| `ticket.py close / reopen` | Keep Owner prompt | Gate 7 / Owner-only |
| `handoff.py write` | Auto-approve | Caller: known set (`scrum`, `operation-manager`, `*-automated`) |
| `message.py create` | Auto-approve | Caller: known set only |
| `docker compose ps / logs` | Auto-approve | No conditions |
| `docker compose up / down / build` | Keep Owner prompt | Starts services / executes Dockerfiles |
| `python -m pytest django/` | Auto-approve | Confirm test conftest uses test DB only |
| `docker compose --profile test run --rm test` | Auto-approve | Isolated test DB confirmed |
| `npm run lint` / `npx tsc --noEmit` | Auto-approve | No conditions |
| `npm run dev / build / test` | Keep Owner prompt | Network calls / writes |
| `npm ci` / `npm install` | Keep Owner prompt | Introduces third-party code |
| `pip install / uninstall` | Keep Owner prompt | No change |
| `manage.py migrate` / `db_init.py` | Keep Owner prompt | No change |
| Any path outside workspace | Always Owner prompt | Hard rule — one non-negotiable |

#### Agent Execution Protocol change

G3 Owner gate removed. Owner gate is G1 only. Rationale: G1 approval commits the plan, files, and test names. G3 is execution of that approved plan. Alessandro catches any drift at G5. Pre-commit hooks are the last line of defence.

**Condition (Dominick):** G1 must remain a hard Owner gate. No shortcuts, no waived G1s. The model depends on it.

#### Status transition authority

| Transition | Who may execute without Owner prompt |
|---|---|
| `open` → `ready` | Rob (`--caller scrum`) |
| `ready` → `in-development` | Arale (`--caller operation-manager`) |
| `in-development` → `in-testing` | Arale (`--caller operation-manager`) |
| `in-testing` → `in-development` | Alessandro (`--caller tech-lead`) — G5 rejection |
| Any → `resolved` | Owner only — Gate 7 |
| Any → `wont-fix` / `deferred` | Owner only |

#### Workspace boundary

Approval prompts protect against AI scope creep, not external threats. Laptop vulnerability profile is unchanged. The workspace boundary is a behavioral constraint (not an OS sandbox). Agents must be reminded in their agent files that git + pre-commit hooks are the real safeguards.

#### `agent_permissions.md`

Decision: rewrite from scratch. Archived version is stale and predates the multi-repo structure and Arale. Archive copy is reference only.

---

### 2026-03-23 — Tickets raised

| Ticket | Title | Owner | Blocked by |
|---|---|---|---|
| ISS-147 | Write agent_permissions.md v2 — reorg-aware permissions policy from scratch | Dominick | — |
| ISS-148 | Implement settings.json auto-approve rules per agent_permissions.md v2 | Claire | ISS-147 |
| ISS-149 | Update Agent Execution Protocol: remove G3 Owner gate; update agent files with new permission model | Alessandro | ISS-147 |

---

### Owner sign-off

**Owner — 2026-03-23:** Approved. Decisions recorded. Tickets raised.
