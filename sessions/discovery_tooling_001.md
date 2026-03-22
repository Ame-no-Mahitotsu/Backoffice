# Discovery — Project Management Tooling Suite
> Live notes. Written incrementally during discovery. Last updated: 2026-03-13
> If session interrupted, resume from the last completed section marker.
> Orchestrator drives all sessions; Rob chairs multi-agent meetings.
> Project: tooling

## Status
- [x] Files deleted (dashboard.py, create_message.py, messages.md)
- [x] Live notes file created
- [x] BA session (Fran) — 10 user stories, 6 ambiguities flagged
- [x] PO session (Sofia) — MoSCoW done, 3 ambiguities ruled
- [x] Architecture meeting (Patrick + Alessandro, Rob chairs) — 4 decisions
- [x] Security session (Dominick) — 7 threats assessed, 6 hard requirements
- [x] Tech Lead session (Alessandro) — standards, task breakdown, DOR confirmed
- [x] Discovery summary written
- [x] Meetings & Sessions tab addendum — schema approved, ISS-011 raised
- [ ] **AWAITING: Owner approval to proceed to Sprint Planning**

## Context / Inputs
Origin: team brainstorm (see conversation b2203bfe). Key prior decisions:
- Architecture option B selected: stdlib HTTP server (not static generator, not Flask)
- UX: tabs Tickets/Messages, side panel detail, badges, blocking rows highlighted
- Port 8765 default, --port flag, manual start, auto-open browser
- 127.0.0.1 binding only
- Read + write actions in v1 (owner overruled Patrick's read-only recommendation)
- Single file in tools/ — NOT yet validated against modularity requirement
- Owner flags: modularity, decoupling, expandability, ability to modify are key constraints

## What is NOT yet decided
- Formal folder structure (tools/ flat vs. package?)
- Modularity approach: single file vs. multi-module
- API contract (sketch exists from Chris, not formally approved)
- Write action scope: what writes are in v1 exactly?
- How does this sit relative to existing tools/ (issue_reader.py, dispatch_reader.py)?
- Test strategy and test location
- Whether messages.md should live in memory/workspace/ or elsewhere

---

## SECTION 1 — BA Session (Fran)
> Date: 2026-03-13 | Author: Fran (BA)

### Actors
| Actor | Description |
|---|---|
| **Owner** | Human operator. Reads, triages, actions. Needs fast visual scanning. |
| **AI Agent** | Any role agent (Claire, Alessandro, Rob, etc.). Creates and reads programmatically. Needs token-efficient access — loading full markdown files into context is wasteful. |

### User Stories

---

#### US-T01 — Create a ticket
*As an AI Agent, I want to create a ticket in the backlog by running a CLI command with structured arguments, so that I can raise work items without editing markdown directly and without risk of corrupting the file.*

**Acceptance criteria:**
- Command accepts: type, component, title, owner, urgency, tags, notes (initial), sprint (optional)
- ID is auto-assigned (ISS-NNN, sequential, no collision between concurrent agents)
- Status on creation is always `open`
- All controlled fields (type, component, urgency, severity) validated against ticket_schema.py — invalid values rejected with a clear error, not silently accepted
- Severity required for `type=defect`; must be `—` for all other types (enforced)
- Created ticket is immediately visible in the backlog file and via the API
- Command prints the assigned ID and a confirmation line to stdout
- If the backlog file does not exist, it is created with the correct header

---

#### US-T02 — Read tickets (Owner, visual)
*As the Owner, I want to see all tickets in a filterable table in a browser, so that I can scan project state and identify what needs attention without opening markdown files.*

**Acceptance criteria:**
- Filterable by: status, component, owner, type, urgency, sprint
- Filters are independent and combinable
- Row count shown ("N of M tickets")
- Clicking a row opens a detail panel showing all fields and parsed notes
- Detail panel does not replace the list — both remain visible simultaneously
- Blocking/critical urgency items are visually distinct (highlighted row)
- Empty state handled gracefully ("No tickets match filters" — not a crash)
- Malformed backlog file shows an error in the UI, not a server 500

---

#### US-T03 — Read a single ticket (AI Agent, programmatic)
*As an AI Agent, I want to fetch a single ticket by ID via HTTP and receive structured JSON, so that I can act on it without loading the entire backlog file into my context.*

**Acceptance criteria:**
- `GET /api/tickets/ISS-NNN` returns a single JSON object with all fields
- Notes are returned as a parsed list `[{date, author, text}]`, not raw string
- Unknown ID returns `{"error": "ISS-NNN not found"}` with HTTP 404
- Response is stable in shape — agents can rely on field names not changing without notice

---

#### US-T04 — Read tickets list (AI Agent, programmatic)
*As an AI Agent, I want to query the ticket list with filters via HTTP and receive a compact JSON array, so that I can find relevant tickets without token overhead.*

**Acceptance criteria:**
- `GET /api/tickets` returns array of ticket summaries
- Query params filter results: `?status=open`, `?component=backend`, `?owner=Alessandro`, `?urgency=high`, etc.
- Filters are AND-combined when multiple provided
- Empty result returns `[]`, not an error

---

#### US-T05 — Update ticket status (Owner, from dashboard)
*As the Owner, I want to change a ticket's status from the dashboard, so that I can move work through the lifecycle without leaving the browser or editing markdown manually.*

**Acceptance criteria:**
- Status dropdown available in the detail panel for each ticket
- Valid transitions only (controlled by ticket_schema.py values)
- Change is written back to the backlog file immediately
- Updated ticket reflects instantly in the list (polling or immediate re-fetch)
- Concurrent write safety: if two agents write simultaneously, neither write is silently lost

**Flag for Architect:** concurrent write safety needs a design decision. File locking strategy TBD.

---

#### US-T06 — Create a message
*As an AI Agent, I want to create a message addressed to another agent or to all by running a CLI command, so that I can communicate blocking issues or handoff notes without editing markdown.*

**Acceptance criteria:**
- Command accepts: from, to, priority, subject (max 80 chars), body
- ID auto-assigned (MSG-NNN, sequential, no collision)
- Status on creation always `open`
- Priority `blocking` with subject not prefixed `[BLOCKING]` prints a warning (not an error)
- If messages file does not exist, it is created with correct header
- Assigns today's date automatically

---

#### US-T07 — Read messages (Owner, visual)
*As the Owner, I want to see all team messages in a filterable table, so that I can triage communication and spot blocking items before starting work each session.*

**Acceptance criteria:**
- Filterable by: status, priority, from, to
- Blocking messages have a visually distinct row background (red-tinted)
- Badge on tab shows count of open messages; badge pulses/highlights if any are `blocking` + `open`
- Detail panel shows: subject, body, reply, all metadata
- Empty state: "No messages yet" — not a crash

---

#### US-T08 — Reply to / resolve a message (Owner, from dashboard)
*As the Owner, I want to mark a message as resolved or write a reply from the dashboard, so that I can close communication loops without editing the markdown file.*

**Acceptance criteria:**
- "Mark resolved" action available in message detail panel
- Reply text field available; submitting writes reply text to the `**Reply:**` block in the file
- Status changes from `open` → `resolved` on resolution
- Change reflected immediately in list and badge count

---

#### US-T09 — Session startup check (AI Agent)
*As any AI Agent starting a session, I want to be able to check for messages addressed to me or to `all` quickly, so that I can fulfil the rules in messages.md (read at session start, act on blocking before anything else).*

**Acceptance criteria:**
- `GET /api/messages?to=Alessandro&status=open` (or `to=all`) returns relevant messages
- Response is fast enough to be a practical session-start check (local tool — no latency concern)
- Blocking messages are identifiable from the response (`priority: "blocking"`)

---

#### US-T10 — Live data (no stale state)
*As the Owner, I want the dashboard to reflect the current state of the files without requiring a manual page refresh, so that I can see changes made by agents during the session.*

**Acceptance criteria:**
- Dashboard polls for updates automatically (≤5 second interval)
- A "last updated" timestamp is visible
- If the server cannot read a file, the last known good state is preserved with an error indicator — not a blank screen

---

### Ambiguities / Flags for Other Roles

| # | Ambiguity | Escalate to |
|---|---|---|
| A1 | Concurrent write safety for US-T05 and US-T08 — file locking strategy needed | Architect |
| A2 | Should `dispatch_reader.py` be absorbed into the dashboard as a third tab, or stay separate? Dispatches are read-only historical records — different access pattern. | PO to prioritise, Architect to design |
| A3 | Where does messages.md live? Currently `memory/workspace/messages.md`. Is that the right permanent location? | Architect |
| A4 | `issue_reader.py` and `dispatch_reader.py` currently exist without tests. Are they in scope for this sprint (add tests) or handled separately? | PO + Tech Lead |
| A5 | Modularity requirement from Owner: single-file was brainstorm assumption. Does the team validate this against the modularity/decoupling/expandability constraint? | Architect |
| A6 | Write actions (US-T05, US-T08) — should agents also be able to write via API, or only via CLI scripts? | PO + Architect |

---

## SECTION 2 — PO Session (Sofia)
> Date: 2026-03-13 | Author: Sofia (PO)

### Vision Statement
The Project Management Tooling Suite is the operational backbone of the team. It replaces manual markdown editing with safe, structured CLI tools and gives the Owner a live visual window into project state. Without it, process invariants (no ticket = no work; blocking messages must be resolved first) cannot be reliably enforced.

### MoSCoW Prioritisation

| Story | Title | Priority | Reasoning |
|---|---|---|---|
| US-T01 | Create a ticket (CLI) | **MUST** | Foundation. No ticket = no work. Agents cannot operate without it. |
| US-T06 | Create a message (CLI) | **MUST** | Foundation. Team communication depends on it. |
| US-T03 | Read single ticket — API | **MUST** | Token efficiency for agents. Loading full backlog into context is unacceptable at scale. |
| US-T04 | Read ticket list — API | **MUST** | Agents need filtered reads. Same reasoning as US-T03. |
| US-T09 | Session startup check — API | **MUST** | Enforces the process rule: check for blocking messages before any work. |
| US-T02 | Visual ticket dashboard | **SHOULD** | Owner visibility is high value. Not strictly blocking agent work, but blocking Owner oversight. |
| US-T07 | Visual message dashboard | **SHOULD** | Same as US-T02 — Owner must be able to triage without opening files. |
| US-T10 | Live data / auto-polling | **SHOULD** | Dashboard without live data is only marginally better than opening the file. Needed to make SHOULD stories useful. |
| US-T05 | Update ticket status (dashboard) | **COULD** | Owner can use CLI or edit markdown for now. Adds convenience but not critical path. |
| US-T08 | Reply / resolve message (dashboard) | **COULD** | Same rationale as US-T05. Write-from-UI is quality of life, not blocker. |

**Won't Have (v1):**
- Dispatches as a third dashboard tab (see A2 ruling below)
- Bulk status updates
- Agent writes via API (see A6 ruling below)

---

### Ambiguity Rulings (PO scope)

**A2 — Dispatches tab:**
Won't Have in v1. Dispatches are a historical journal — read-only, no actionable state. The Owner reads them by running `dispatch_reader.py` which already works. Adding them to the dashboard is COULD at best, and only after the MUST and SHOULD stories are delivered and stable. Defer to v2.

**A4 — Tests for issue_reader.py and dispatch_reader.py:**
These are existing tools with no tests. They are not in scope for this sprint — but they cannot be called "done" under our TDD standard. I am raising a separate ticket for this. It belongs in Sprint 1 alongside tooling work, not in Discovery. Flagging to Rob.

**A6 — Agent writes via API vs. CLI only:**
~~CLI only for writes in v1.~~ **OVERRIDDEN BY OWNER 2026-03-13.**
Owner decision: API supports both read and write from v1. Agents may create/update tickets and messages via HTTP as well as via CLI. Reasons: agents can operate programmatically without shelling out; more flexible for future tooling. CLI tools remain as convenient wrappers but are no longer the exclusive write path.

---

### Notes for Architect
The MUST stories (US-T01, T03, T04, T06, T09) must be deliverable without the dashboard running. The CLI tools and API are independently valuable. Architecture must not bundle them such that the dashboard is required for agents to operate.

This directly implies: **the server and the CLI tools must be decoupled**. An agent should be able to run `create_ticket.py` with zero dependency on the dashboard server being up.

---

## SECTION 3 — Architecture Meeting
> Date: 2026-03-13 | Chair: Rob (SM) | Attendees: Patrick (Architect), Alessandro (Tech Lead)
> Meeting type: WRK | File: meeting_wrk_002.md (to be written post-meeting)

### Agenda
1. Module structure — single file vs. package
2. Folder structure
3. Concurrent write safety
4. File locations

---

### Item 1 — Module structure

**Rob:** Sofia ruled CLI tools must work without the server. The brainstorm decided single file. Those two things are in tension. Patrick, what's your read?

**Patrick:** The single-file decision was made under the assumption of read-only, simple tool. Write actions change the picture. The parsers need to be shared between the CLI writers and the server — if we inline them in a single dashboard.py, the CLI tools either re-implement parsing or import from the server file, which is backwards. A package is the right call.

There are three options:

- **Option A — Flat files, duplicated logic.** CLI tools and server each have their own parser. Simple to read, painful to maintain — any change to the markdown format must be made twice.
- **Option B — Package (`tools/pm/`).** Shared parsers, shared writers, thin CLI entry points, thin server entry point. No new external dependencies — all stdlib. Testable independently.
- **Option C — Single monolith.** Everything in dashboard.py. CLI tools call into it. Violates Sofia's decoupling requirement directly.

My recommendation: **Option B**. The `pm/` package is internal. It adds files, not complexity.

**Alessandro:** Agree with Patrick. From a standards and testability standpoint, Option B is the only viable path. Tests for the parser live in `tools/pm/tests/test_parsers.py`. Tests for the writers live in `tools/pm/tests/test_writers.py`. The server becomes a thin HTTP layer over the parsers. You can test parsers without starting a server at all.

One additional point: the expandability requirement. If we ever add a third data source (dispatches dashboard, future sprint file), we add `parsers_dispatches.py` to the package and a route to `server.py`. With a monolith, that's open-heart surgery.

**Rob:** Is there any dissent on Option B?

*(No dissent.)*

**Decision: Option B — `tools/pm/` package. Rationale: shared parsers/writers, testable in isolation, server is thin HTTP layer, expandable without restructuring.**

---

### Item 2 — Folder structure

**Patrick:** Proposed structure:

```
tools/
├── create_ticket.py        ← CLI entry point (thin: parses args, calls pm.writers)
├── create_message.py       ← CLI entry point (thin: parses args, calls pm.writers)
├── dashboard.py            ← Server entry point (thin: starts HTTPServer with pm.server)
└── pm/                     ← Internal package (no external deps, all stdlib)
    ├── __init__.py
    ├── parsers.py           ← parse_tickets(), parse_messages() — read-only, no side effects
    ├── writers.py           ← write_ticket(), write_message() — atomic file writes
    ├── server.py            ← DashboardHandler, routing, send_json(), send_html()
    ├── ui.py                ← HTML constant (keeps server.py readable)
    └── tests/
        ├── __init__.py
        ├── test_parsers.py
        ├── test_writers.py
        └── test_server_api.py
```

**Alessandro:** Agreed. One addition: `pm/__init__.py` should export nothing public — it is an internal package. The CLI entry points (`create_ticket.py`, `dashboard.py`) are the public interface. This prevents agents from ever `import pm.parsers` directly in application code — they go through the CLI or HTTP.

**Rob:** Does this handle the existing `issue_reader.py` and `dispatch_reader.py`?

**Alessandro:** They stay as-is for now — they're independent CLI tools and don't need the package. When the dashboard gets a dispatches tab (v2), `dispatch_reader.py` logic moves into `pm/parsers_dispatches.py` and the CLI becomes a thin wrapper. No breaking change.

**Decision: Folder structure as above. `issue_reader.py` and `dispatch_reader.py` unchanged. `pm/` is internal — not imported directly from outside the tools/ directory.**

---

### Item 3 — Concurrent write safety

**Rob:** Fran flagged this. If two agents run `create_ticket.py` simultaneously, what happens?

**Patrick:** Two approaches worth considering:

- **Lock file:** Script creates `.backlog.lock`, does its work, releases. Second script waits or fails fast. Stdlib-only. Works for our use case.
- **Atomic replace:** Write to a temp file in the same directory, then `os.replace(temp, target)`. On POSIX and Windows (Python 3.3+), `os.replace` is atomic at the OS level. No lock file needed.

My preference: **atomic replace only**. Lock files have failure modes (stale lock if a script crashes). For a local single-machine tool where concurrent writes are rare, atomic replace is sufficient and simpler.

**Alessandro:** Agreed. `os.replace()` in the writer handles it. One caveat: we need to read → modify → write in a single writer function, not spread across the script. The writer owns the full read-modify-write cycle. This is a coding standard we enforce in review.

**Decision: Atomic write via `os.replace()` (write to `.tmp` file, then replace). No lock file. Writer function owns full read-modify-write cycle atomically.**

---

### Item 4 — File locations

**Rob:** Two files to confirm: `memory/issue_backlog.md` and `memory/workspace/messages.md`. Are these the right permanent homes?

**Patrick:** `memory/issue_backlog.md` — correct. That's project memory, not workspace state. Stays.

`memory/workspace/messages.md` — this is trickier. `workspace/` was defined as transient working files. Messages are operational state — not transient. I'd argue they belong at `memory/messages.md` alongside `issue_backlog.md`. But changing it means updating all file path references in existing tools.

**Alessandro:** The argument for keeping it in `workspace/` is that it's team-internal, not project-permanent like the backlog. The argument against is that messages persist across sessions — they're not workspace artefacts. I side with Patrick: `memory/messages.md`.

**Rob:** Decision needed before any code is written. Patrick's argument is cleaner — messages are not transient.

**Decision: `memory/messages.md` (not `workspace/`). Rationale: messages persist across sessions and are operational state, not transient workspace artefacts. All path references in `pm/parsers.py` and `pm/writers.py` use this path.**

---

### Open items passed to Dominick
- Write action security: `html.escape()` confirmed, but validation of incoming write data (status transitions, field values) needs a security lens before implementation.
- Path construction in writers: confirm no dynamic path construction from user input.

### Open items passed to Alessandro (Tech Lead session)
- Confirm `pm/` package is excluded from project-level imports (Django, Next.js — completely separate concern)
- Define test runner for `tools/pm/tests/` — does this use the project pytest setup or its own?
- Confirm `__init__.py` convention

---

## SECTION 4 — Security Session (Dominick)
> Date: 2026-03-13 | Author: Dominick (Security Engineer)

### Scope
Full threat model for the Project Management Tooling Suite including write actions (US-T05, US-T08). Supersedes the incomplete review from the previous session.

### What is NOT a concern (and why)
| Non-issue | Reason |
|---|---|
| Authentication | 127.0.0.1 binding is the authentication. Only processes on this machine can connect. |
| TLS | Localhost HTTP is acceptable. No data leaves the machine. |
| CSRF | Same-origin by nature of localhost. Not applicable. |
| SQL injection | No database. No queries. |
| Secrets exposure | No credentials in this tool. Not applicable. |

### Threat Model

---

#### T1 — XSS via rendered content
**Risk:** Agent-created content (ticket notes, message bodies) contains `<script>` tags or HTML. If rendered raw in the browser, script executes in the Owner's browser session.
**Likelihood:** Low-medium. Agents don't intend injection, but markdown content is free-text and could contain `<` or `>` incidentally.
**Impact:** Low for a local tool (no auth cookies to steal, no sensitive sessions). But: execution of arbitrary JS in the Owner's browser is unacceptable regardless of environment.
**Mitigation:** `html.escape()` on **every** value before insertion into HTML. No exceptions. Applied in `pm/ui.py` rendering, not in the server or parsers. The parser returns raw text — the UI layer escapes it.
**Status:** REQUIRED. Must be in Definition of Done for every UI story.

---

#### T2 — Markdown structure corruption via write (pipe injection)
**Risk:** Free-text fields (notes, body, reply) written to markdown table rows may contain `|` characters, breaking the table structure. Parsers then misread subsequent rows, silently corrupting data.
**Likelihood:** Medium. Agents generating notes programmatically may include `|` in technical descriptions (e.g. "the values are `a | b | c`").
**Impact:** High. Silent data corruption is the worst kind — no error, just wrong data.
**Mitigation:** `pm/writers.py` must escape `|` in all free-text fields before writing to table rows. Convention: replace `|` with `\|` in table cells. Detail blocks (body, reply, notes below the table) are written as prose, not table cells — no escaping needed there.
**Status:** REQUIRED. Enforced in `pm/writers.py`. Covered by `test_writers.py`.

---

#### T3 — Invalid controlled-field values written to files
**Risk:** A write action (CLI or dashboard) passes an invalid status, type, or component value. The file accepts it — the schema is not enforced at write time. Future reads produce unexpected filtering behaviour or UI crashes.
**Likelihood:** Medium. Agents can make mistakes. Dashboard forms can be manipulated (browser dev tools).
**Impact:** Medium. Data integrity degraded; may cascade to incorrect ticket state.
**Mitigation:** `pm/writers.py` imports `ticket_schema.py` and validates all controlled fields before writing. Invalid value → exception, no write occurs. Error message returned to caller (CLI prints it; API returns 400 + JSON error). This is the single validation point — server never writes directly, only calls the writer.
**Status:** REQUIRED.

---

#### T4 — Write to non-existent ID (ghost update)
**Risk:** A dashboard write (status change, reply) targets a ticket or message ID that does not exist in the file. Writer creates a dangling update or silently does nothing.
**Likelihood:** Low. Requires stale UI state (UI showed an ID that was deleted before the write).
**Impact:** Low. Confusing UX, no data loss.
**Mitigation:** Writer reads current file state before writing. If ID not found, raises an error. API returns 404. No write occurs.
**Status:** REQUIRED.

---

#### T5 — Dynamic file path construction from request input
**Risk:** If the server constructs file paths from URL parameters or query strings, an attacker (local process) could read or write arbitrary files via path traversal (`../../../etc/passwd`).
**Likelihood:** Low. But local processes can be compromised (malicious package in a dev dependency).
**Impact:** High if exploitable. Reads or overwrites any file the process has permission to access.
**Mitigation:** `pm/parsers.py` and `pm/writers.py` use hardcoded `Path` constants for the two data files. No path is ever constructed from request parameters. The server routing maps URL patterns to handler functions — there is no generic file-serving endpoint.
**Status:** REQUIRED. Verified in code review.

---

#### T6 — LAN exposure
**Risk:** Server binds to `0.0.0.0` instead of `127.0.0.1`. Reachable from shared network.
**Likelihood:** Would only occur if the binding is miscoded.
**Impact:** Full dashboard read (and write in v1) accessible to LAN peers. Project backlog and team messages exposed.
**Mitigation:** `DEFAULT_HOST = "127.0.0.1"` hardcoded in `dashboard.py`. Not configurable. No `--host` flag.
**Status:** REQUIRED.

---

#### T7 — Atomic write window (read-modify-write race)
**Risk:** Two concurrent writers both read the file, both modify their copy, second write overwrites first write's changes.
**Likelihood:** Very low. Single operator, agents operate sequentially in normal use.
**Impact:** Low. One write lost. No corruption — file remains valid markdown.
**Mitigation:** `os.replace()` ensures the final write is atomic (no partial writes). For this tool at this scale, this is acceptable residual risk. Lock files are disproportionate. Accepted.
**Status:** ACCEPTED RISK. Documented. Revisit if concurrent agent use increases.

---

### Security Requirements Summary (for Alessandro / Tech Lead)

These are non-negotiable constraints for all code in `pm/`:

| Requirement | Where enforced | Test required |
|---|---|---|
| `html.escape()` on all values rendered in HTML | `pm/ui.py` | Yes — test_server_api.py |
| `\|` escaping in free-text table fields | `pm/writers.py` | Yes — test_writers.py |
| Controlled field validation against ticket_schema.py | `pm/writers.py` | Yes — test_writers.py |
| Writer validates ID exists before modifying | `pm/writers.py` | Yes — test_writers.py |
| Hardcoded file paths only — no dynamic construction | `pm/parsers.py`, `pm/writers.py` | Review gate |
| Server binds to 127.0.0.1 only — no `--host` flag | `dashboard.py` | Review gate |

### Sign-off
Security review complete. No blockers to proceeding to Tech Lead session and Sprint planning. All requirements above must be in the Definition of Done for any story that touches write operations.

---

## SECTION 5 — Tech Lead Session (Alessandro)
> Date: 2026-03-13 | Author: Alessandro (Tech Lead)

### Item 1 — Package isolation from Django / Next.js

`tools/pm/` lives in `SitoPresepi2/tools/` (project management directory).
Django apps live in `SitoPresepi2-src/django/`.
These are separate directory trees with no shared Python path. Structural isolation is already guaranteed — no import guard needed.

**Rule:** No Django app, no Next.js file, and no `SitoPresepi2-src/` file may ever import from `tools/pm/`. Enforced at code review. If this boundary is ever needed programmatically, a `py.typed` marker and a Ruff rule can enforce it — not needed now.

---

### Item 2 — Test runner for `tools/pm/tests/`

`pm/` is pure Python — no Django, no ORM, no settings file. It does not belong in the Docker test container.

**Decision:** `pytest tools/pm/tests/` run directly from the project root (`SitoPresepi2/`) on the host machine. No container. No Django settings. Just Python.

Why not Docker? The Docker test container carries Django's full startup cost. For pure-Python stdlib tools, that overhead is unjustified and slows TDD feedback loops.

`tools/pm/tests/` gets its own `pytest.ini` (minimal):

```ini
[pytest]
testpaths = tools/pm/tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

Pre-commit hook updated to run both test suites: Django (Docker) and pm tools (host).

---

### Item 3 — `__init__.py` convention

`tools/pm/__init__.py` — empty file. No exports. No public API surface.
`tools/pm/tests/__init__.py` — empty file.

The package is internal. Entry points (`create_ticket.py`, `create_message.py`, `dashboard.py`) are the public interface. Nothing imports `pm` directly except those three files.

---

### Item 4 — Coding standards for `pm/` package

**Same toolchain as the rest of the project:** Black, isort, Ruff, pre-commit. No exceptions for internal tools.

**Module responsibilities — hard boundaries:**

| Module | Owns | Must NOT |
|---|---|---|
| `pm/parsers.py` | Read files, return dicts/lists | Write to files, import server or writers |
| `pm/writers.py` | Read-modify-write cycle, validation | Import server or parsers (may re-read inline) |
| `pm/server.py` | HTTP routing, request/response | Write to files directly — calls writers only |
| `pm/ui.py` | HTML/CSS/JS string constant | Contain business logic |
| `create_ticket.py` | Arg parsing, call writers | Contain validation logic |
| `create_message.py` | Arg parsing, call writers | Contain validation logic |
| `dashboard.py` | Start HTTPServer | Contain routing or parsing logic |

**parsers.py standards:**
- Functions: `parse_tickets() -> list[dict]`, `parse_messages() -> list[dict]`, `parse_notes(raw: str) -> list[dict]`
- Returns plain dicts — no dataclasses, no Pydantic, no external deps
- Pure functions: same input → same output, no side effects
- If file does not exist: return `[]` (not an exception — empty state is valid)
- If file is malformed: skip the malformed row, log a warning to stderr, continue

**writers.py standards:**
- Each function owns the complete read-modify-write cycle
- Write pattern: `write to .tmp file` → `os.replace(.tmp, target)` — always
- Validate controlled fields against `ticket_schema.py` before any write
- Escape `|` in free-text table fields: `value.replace('|', '\\|')`
- Raise `ValueError` with a clear message on invalid input — never silently accept
- Return the assigned ID (for create operations) or `None` (for update operations)

**server.py standards:**
- Routing via `re.match()` — no framework
- Delegates immediately to parsers (reads) or writers (writes) — no inline logic
- `send_json()` and `send_html()` are module-level helpers
- `log_message()` suppressed (no per-request noise in the terminal)

**ui.py standards:**
- Single module-level constant: `HTML: str`
- `html.escape()` applied in the JavaScript layer via the `esc()` function — not in Python
- JS must call `esc()` on every value before insertion into innerHTML — no exceptions

**Naming:**
- All functions: `snake_case`
- All constants: `SCREAMING_SNAKE_CASE`
- Dict keys in returned data: `snake_case` (consistent with JSON API shape)

---

### Technical Task Breakdown (for sprint ticket creation)

These are the implementation units. Each becomes a ticket or a sub-task within a ticket.

**MUST (CLI + API core):**
```
TASK-A  pm/__init__.py, pm/tests/__init__.py, pytest.ini          [~15 min, test: trivially passing]
TASK-B  pm/parsers.py — parse_tickets(), parse_messages(),        [~2h, test: test_parsers.py]
         parse_notes()
TASK-C  pm/writers.py — write_ticket(), write_message()           [~2h, test: test_writers.py]
         (create operations only, with validation + atomic write)
TASK-D  tools/create_ticket.py — thin CLI wrapper over writers    [~45min, test: test_writers.py covers logic]
TASK-E  tools/create_message.py — thin CLI wrapper over writers   [~45min, same]
TASK-F  pm/writers.py — update_ticket_status(),                   [~1.5h, test: test_writers.py]
         update_message_status(), append_message_reply()
         (update operations, validation, ID-exists check,
          transition rules, immutability constraints)
TASK-F2 tools/update_ticket.py — thin CLI wrapper over update    [~1h, test: test_writers.py covers logic]
         writers; enforces Rob's transition constraint matrix;
         --note required for → resolved/wont-fix/deferred;
         --caller flag (default: agent)
```

**SHOULD (dashboard):**
```
TASK-G  pm/server.py — HTTP handler, GET routes for API           [~2h, test: test_server_api.py]
TASK-H  pm/ui.py — HTML/CSS/JS (tabs, filters, side panel,        [~3h, test: browser + test_server_api.py]
         badges, live polling, esc() on all values)
TASK-I  tools/dashboard.py — thin entry point, auto-open browser  [~30min, test: smoke test]
```

**Build order:** A → B → C → D → E → F → F2 → G → H → I. Each depends on the previous. No parallelism in v1 — single developer (Owner/AI).

---

### Definition of Ready — tooling tickets
A ticket is ready to enter a sprint when:
- [ ] User story written with acceptance criteria (Fran ✓)
- [ ] Security constraints documented (Dominick ✓)
- [ ] Architecture decision recorded (Patrick ✓)
- [ ] Technical tasks broken down (Alessandro ✓)
- [ ] No unresolved ambiguities (all Fran's flags addressed ✓)
- [ ] PO priority confirmed (Sofia ✓)

**All MUST stories meet Definition of Ready. SHOULD stories meet it pending security sign-off on US-T10 (live polling) — Dominick confirmed no concern.**

---

### What needs updating in ai-standards.md
After Owner approves discovery output, the following must be added to `ai-standards.md`:
1. `tools/pm/` package structure and module responsibility table
2. Test runner for pm tools (host pytest, not Docker)
3. Writer atomic write pattern (`os.replace`)
4. `|` escaping rule for free-text table fields
5. `html.escape()` / `esc()` requirement for all rendered values

---

## SECTION 6 — Discovery Summary
> Date: 2026-03-13 | Author: Orchestrator

### What was decided

| Topic | Decision | Owner |
|---|---|---|
| Module structure | `tools/pm/` package — parsers, writers, server, ui as separate modules | Patrick |
| CLI tools | Thin entry points: `create_ticket.py`, `create_message.py`, `dashboard.py` in `tools/` | Alessandro |
| Server/CLI decoupling | CLI tools work with zero dependency on the server being up | Sofia (requirement) |
| Concurrent writes | Atomic via `os.replace()` — no lock file | Patrick |
| File locations | `memory/issue_backlog.md` (unchanged), `memory/messages.md` (moved from workspace/) | Patrick |
| Test runner | `pytest tools/pm/tests/` on host — no Docker, no Django | Alessandro |
| Write scope | API supports read + write in v1; CLI tools remain as wrappers | Owner (overrides Sofia A6) |
| Dispatches tab | Won't Have v1 — defer to v2 | Sofia |
| Agent API writes | In v1 via HTTP POST/PATCH endpoints | Owner |

### User Stories — Priority Summary

| Story | Title | Priority |
|---|---|---|
| US-T01 | Create ticket (CLI) | MUST |
| US-T06 | Create message (CLI) | MUST |
| US-T03 | Read single ticket — API | MUST |
| US-T04 | Read ticket list — API | MUST |
| US-T09 | Session startup check — API | MUST |
| US-T02 | Visual ticket dashboard | SHOULD |
| US-T07 | Visual message dashboard | SHOULD |
| US-T10 | Live data / auto-polling | SHOULD |
| US-T11 | Update ticket via CLI (`update_ticket.py`) | MUST |
| US-T05 | Update ticket status (dashboard) | COULD |
| US-T08 | Reply / resolve message (dashboard) | COULD |

### Security requirements (non-negotiable)
1. `html.escape()` / `esc()` on all values rendered in HTML
2. `|` escaping in all free-text table fields written to markdown
3. Controlled field validation against `ticket_schema.py` before any write
4. Writer validates ID exists before modifying
5. Hardcoded file paths only — no dynamic construction from input
6. Server binds to `127.0.0.1` only — no `--host` flag

### Implementation tasks (in build order)
TASK-A through TASK-I defined in Section 5, plus TASK-F2 (update_ticket.py). MUST = A–F, F2. SHOULD = G–I.

### What happens next (Sprint 1 Planning)
1. **Owner approves this discovery document**
2. **Alessandro updates `ai-standards.md`** with `pm/` package standards
3. **Rob creates backlog tickets** for TASK-A through TASK-I using `create_ticket.py` — *except `create_ticket.py` doesn't exist yet, so Rob creates them by running the Discovery output through Sprint Planning manually*
4. **Rob runs Sprint 1 Planning** — assigns tasks, sets sprint
5. **Build starts: TASK-A first, TDD throughout**

### Open items (not blocking Discovery close)
- ISS-004 equivalent: `issue_reader.py` and `dispatch_reader.py` have no tests — Sofia flagged for a separate ticket
- `ai-standards.md` update pending Owner approval of this document
- `memory/messages.md` does not exist yet — created by `create_message.py` on first use (per US-T06 AC)

---

## ADDENDUM — API Write Contract & Schema Enforcement
> Date: 2026-03-13 | Raised by: Owner | Recorded by: Orchestrator

### The concern
Agents will use the API to create and update tickets and messages. The API must enforce the same constraints as the CLI tools and `ticket_schema.py`. An agent must not be able to submit:
- An invalid `type` (e.g. `"bug"` instead of `"defect"`)
- An invalid `status`, `component`, `urgency`, or `severity`
- A missing `severity` on a `defect` ticket
- A `severity` value on a non-defect ticket
- A subject over 80 characters (messages)

If it does, it must receive a clear, actionable error — not a silent acceptance or a 500.

### Where enforcement lives
`pm/writers.py` is already the single validation point. It imports `ticket_schema.py` and validates before any write. This design is correct. The API never writes directly — it always calls writers.

**What is missing:** the API response contract for write operations is not yet formally specified.

### API write contract (to be formally approved before TASK-G)

**POST /api/tickets**
```
Request body (JSON):
{
  "type":       required, TicketType enum value
  "component":  required, TicketComponent enum value
  "title":      required, non-empty string
  "owner":      required, string
  "urgency":    required, TicketUrgency enum value
  "severity":   required if type=defect, must be TicketSeverity value
                must be "—" for all other types
  "tags":       optional, list of strings (prefixed #)
  "notes":      optional, string (auto-timestamped by writer)
  "sprint":     optional, string (default "—")
}

Success → 201 Created
{ "id": "ISS-NNN", "status": "open" }

Validation failure → 400 Bad Request
{ "error": "invalid type: 'bug'. Valid values: story, task, defect, improvement, question, idea" }

File write failure → 500 Internal Server Error
{ "error": "write failed: <reason>" }
```

**PATCH /api/tickets/{id}**
```
Request body (JSON) — all fields optional, only provided fields updated:
{
  "status":   TicketStatus enum value
  "owner":    string
  "sprint":   string
  "urgency":  TicketUrgency enum value
}

Success → 200 OK
{ "id": "ISS-NNN", "updated": "<field>": "<new_value>" }

ID not found → 404 Not Found
{ "error": "ISS-NNN not found" }

Validation failure → 400 Bad Request
{ "error": "invalid status: 'done'. Valid values: open, in-refinement, ready, ..." }
```

**POST /api/messages**
```
Request body (JSON):
{
  "from":     required, string
  "to":       required, string
  "priority": required, one of: low, medium, high, blocking
  "subject":  required, max 80 chars
  "body":     required, non-empty string
}

Success → 201 Created
{ "id": "MSG-NNN", "status": "open" }

Validation failure → 400 Bad Request
{ "error": "subject exceeds 80 characters (got 95)" }
```

**PATCH /api/messages/{id}**
```
Request body (JSON):
{
  "status":  one of: open, acknowledged, deferred, resolved
  "reply":   string (appended to reply block)
}

Success → 200 OK
{ "id": "MSG-NNN", "updated": { ... } }

ID not found → 404 Not Found
```

### Schema discovery endpoint (new — raised by this concern)
Agents should not need to read `ticket_schema.py` to know valid values. The API should expose them.

**GET /api/schema**
```json
{
  "ticket": {
    "type":      ["story", "task", "defect", "improvement", "question", "idea"],
    "status":    ["open", "in-refinement", "ready", "in-development", "in-testing", "po-acceptance", "resolved", "wont-fix", "deferred"],
    "urgency":   ["low", "medium", "high", "critical"],
    "severity":  ["critical", "major", "minor", "trivial"],
    "component": ["catalogue", "commerce", "customers", "gallery", "content", "localisation", "admin-app", "api", "frontend", "auth", "devops", "core", "process"]
  },
  "message": {
    "priority": ["low", "medium", "high", "blocking"],
    "status":   ["open", "acknowledged", "deferred", "resolved"]
  }
}
```

This endpoint:
- Returns only static enum values — no file reads needed
- Allows agents to validate their own input before posting
- Stays automatically in sync because it reads from `ticket_schema.py` at request time

### Impact on task breakdown
- TASK-B (parsers): no change
- TASK-C (writers): validation error messages must be structured (include valid values list)
- TASK-G (server): add `POST /api/tickets`, `POST /api/messages`, `PATCH /api/tickets/{id}`, `PATCH /api/messages/{id}`, `GET /api/schema`
- TASK-G test scope increases: `test_server_api.py` must cover invalid input → 400, missing field → 400, unknown ID → 404, valid write → 201/200

### Approved by
Owner raised concern 2026-03-13. Contract above to be confirmed by Owner before TASK-G begins. Alessandro to review for technical correctness.

---

## ADDENDUM — Meetings & Sessions Dashboard Tab
> Date: 2026-03-13 | Raised by: Owner | Recorded by: Rob (SM)

### Decision
A third tab — **Meetings & Sessions** — will be added to the dashboard (alongside Tickets and Messages). Both use cases confirmed: search/filter past decisions AND status view (open/in-flight vs. closed).

This tab covers two file types:
- **Meeting files** (`meeting_<prefix>_<NNN>.md`) — already have formalised 9-field header. No schema change needed.
- **Session files** (`session_*.md`) — currently free-form. New schema defined below.

---

### Session File Schema (approved 2026-03-13)

```
---
ID: session_<role>_<name>_<NNN>_<topic>.md  ← filename is the ID
Type: session
Role: <role abbreviation, e.g. ba, architect, tech-lead, ux, security>
Topic: <one-line description of what was worked on>
Date: YYYY-MM-DD
Participants: Owner, <role agents present>
Ticket: <ticket ID or —>
Sprint: <sprint name or —>
Status: open | closed
Outputs: <comma-separated list of files created, decisions made, tickets raised>
---
```

**File naming convention (approved 2026-03-13):**
`session_<role>_<name>_<NNN>_<main topic in one word>.md`

Examples:
- `session_architect_patrick_001_architecture.md`
- `session_tech-lead_alessandro_002_repository.md`
- `session_ux_lota_001_ia.md`
- `session_security_dominick_001_checklist.md`

**Design rationale:**
- `<role>` — enables filter by agent type
- `<name>` — disambiguates if multiple agents share a role in future
- `<NNN>` — sequential per role, sortable
- `<topic>` — human-readable at a glance without opening the file

---

### Dashboard tab behaviour
- **Status view:** Filter `Status: open` to see in-flight work sessions or open meeting actions.
- **Search/filter:** By Role, Date, Sprint, Ticket, Topic (substring). "Find where decision X was made" → filter by Ticket or search Topic.
- Meetings and sessions rendered in a single tab, distinguishable by `Type` field.

---

### Backfill
All existing session files in `memory/workspace/` must be retrofitted:
1. Prepend the new header block (inferring fields from file content)
2. Rename files to the new naming convention
3. Any file where fields cannot be reliably inferred — flag for Owner to fill manually

This is tracked as **ISS-011**. Alessandro owns the backfill.

---

### Impact on task breakdown
- TASK-B (parsers): add `parse_sessions()` and `parse_meetings()` alongside `parse_tickets()` / `parse_messages()`
- TASK-G (server): add `GET /api/sessions` and `GET /api/meetings` endpoints
- TASK-H (ui): add third tab rendering sessions + meetings combined

### Approved by
Owner 2026-03-13.

---

## ADDENDUM — Project Field, Workspace Structure, Repository Layout
> Date: 2026-03-13 | Meeting: Rob (chair), Fran, Patrick, Alessandro | Type: WRK

### Trigger
Two active project tracks (tooling, site) require a `project` dimension on all records. Sprint name prefixes alone are insufficient — filter and search must work on structured metadata.

---

### Decision 1 — Project enum (approved)

`TicketProject` enum in `ticket_schema.py`:
```python
class TicketProject(str, Enum):
    TOOLING = "tooling"
    SITE    = "site"
    GENERAL = "general"
```
- Extensible: add a line per new project — parsers, writers, and `GET /api/schema` pick it up automatically
- `general`: cross-project records (e.g. security checklists, governance tickets)
- **Mandatory on all record types:** tickets, messages, meetings, sessions — no record may exist without a project value

---

### Decision 2 — Workspace subfolder structure (approved)

```
memory/workspace/
├── tooling/    ← tooling project sessions, meetings, sprints, discovery
├── site/       ← website project sessions, meetings, sprints, discovery
└── shared/     ← cross-project artefacts (this file, governance)
```

Shared files at `memory/` root (constitution, backlog, messages, ai-standards) are unchanged — they are genuinely cross-project.

**Parser rule (Alessandro):** folder is organisational aid only. `project` field in the file header is the authoritative data source. Parsers use `glob('memory/workspace/**/*.md', recursive=True)`.

**ISS-011 scope expanded:** backfill now covers (1) session headers, (2) rename to new convention, (3) add `Project:` field, (4) move files into correct workspace subfolder.

---

### Decision 3 — Repository layout (approved, pending Owner move)

```
c:\temp\ClaudeProjects\
├── repos/
│   └── SitoPresepi2.git      ← bare repo (moved from SitoPresepi2/remote.git)
├── SitoPresepi2/             ← PM artefacts, memory, tools
└── SitoPresepi2-src/         ← working clone (update remote URL after move)
```

One-time move before the site repo is created. Tracked as ISS-015.

**Tools in PM repo:** `tools/pm/` stays in `SitoPresepi2/` tracked by `SitoPresepi2.git`. No separate repo needed — internal infrastructure, not a deployable product.

---

### Decision 4 — Sprint naming convention (approved)

| Track | Format | Examples |
|---|---|---|
| Tooling | `tools-0`, `tools-1` … | `tools-0` = tooling Sprint 0 |
| Website | `site-0`, `site-1` … | `site-0` = website Sprint 0 |

Tooling track completes at minimum TASK-D and TASK-F2 before website Sprint 0 begins.

---

### Impact on task breakdown

**New TASK-0 (pre-TASK-A, Sprint tools-0):**
```
TASK-0  ticket_schema.py — add TicketProject enum                [~15min, no test needed — enum only]
         issue_backlog.md — add Project column header
         Meeting/session header schema — add Project: field
         Backfill ISS-001..ISS-012 with project values
```

**Revised build order:** 0 → A → B → C → D → E → F → F2 → G → H → I

**TASK-D amended:** `create_ticket.py` must include `--project` (required flag)
**TASK-E amended:** `create_message.py` must include `--project` (required flag)
**TASK-H amended:** dashboard project filter dropdown (driven by `GET /api/schema`)
**US-T06 AC amended:** `create_message.py` acceptance criteria must require `--project` — noted by Sofia

---

### New user stories

| ID | Story | Priority |
|---|---|---|
| US-T12 | Project field on all record types | MUST |
| US-T13 | Filter by project in dashboard | SHOULD |

---

### What needs to go in constitution.md (pending Owner approval)
1. Sprint naming convention: `tools-N` / `site-N`
2. Workspace subfolder structure (`tooling/`, `site/`, `shared/`)
3. Repository layout: `repos/` folder at `ClaudeProjects` level
4. Project enum as canonical multi-project extension point

### Approved by
Owner 2026-03-13.
