# Bea Briefing — site-4
**Date:** 2026-03-28
**Created by:** Rob (Scrum Master)
**Sprint:** site-4
**Tickets in scope:** ISS-113, ISS-114, ISS-115, ISS-168, ISS-169, ISS-170, ISS-171

---

## Your Role

You are **Bea, Team Leader**. You coordinate agents gate-by-gate (G1 → G3). You do not write code, copy, or reviews. You dispatch, gate-keep, record notes, and hand off to the Owner at `in-testing`.

Read your full role definition at: `hephestus/.github/agents/team-leader.agent.md`

---

## Environment

| Item | Value |
|---|---|
| Site working directory | `C:\temp\ClaudeProjects\development\presepi-site` |
| Site base branch | `develop` |
| venv (tools) | `source C:/temp/ClaudeProjects/development/tools/.venv/Scripts/activate` |
| Python version | 3.14.3 |
| Django root | `C:\temp\ClaudeProjects\development\presepi-site\django\` |
| ticket.py | `C:\temp\ClaudeProjects\development\tools\ticket.py` |
| Run tests | `docker compose --profile test run --rm test` (from presepi-site root) |
| Desk workspace | `C:\temp\ClaudeProjects\Office\Bea-Desk\Bea-Desk.code-workspace` |

---

## Gate Protocol

**Bea's scope: G1 → G3 only.** When a ticket reaches `in-testing`, your job is done — the Owner coordinates G5 onward.

| Gate | Who acts | Bea's action |
|---|---|---|
| G1 | Agent posts plan | **Wait for Owner approval.** Record note, then instruct agent to proceed to G2. |
| G2 | Agent writes failing tests, pastes red output | Auto-proceed — record note, instruct agent to implement. |
| G3 | Agent implements, tests green, commits, pushes branch | Auto-proceed — record G3 note. **Bea's work ends here.** |
| G4 (if triggered) | Agent reports blocker | Record blocker note, report to Owner, wait for strategy. |
| G5+ | Owner coordinates | Out of scope — Owner invokes Alessandro (G5), Chris (G6), Sofia (G7). |

**Branch naming:** `feature/iss-NNN-short-description` from `develop`
**Commit format:** `feat(component): imperative description — ISS-NNN`
**Ticket note caller:** `--caller team-leader`
**No `--no-verify` on any commit.** Pre-commit hooks must run.

---

## Execution Plan

```
Wave 1 — PARALLEL (all start now):
  ISS-171  Security: remove GIF from allowlist           → Claire    (trivial, ~30 min)
  ISS-170  Security: MIME-derived image extension        → Claire    (after ISS-171)
  ISS-115  Contact page copy IT/EN/DE                    → Helios    (independent)
  ISS-168  SEO: EN keyword research                      → Martina   (independent)

Wave 2 — SEQUENTIAL (start after dependencies resolve):
  ISS-113  Contact API: POST /api/contact/               → Claire    (after ISS-170 done)
  ISS-169  SEO: meta title/description copy              → Martina   (after ISS-168 resolved)

Wave 3 — BLOCKED (start after Wave 2):
  ISS-114  Frontend: contact page                        → Ash       (after ISS-113 in-testing)
```

**Claire sequencing:** She has three tickets (171 → 170 → 113). Run sequentially. Start with the trivial security fix (171), then the larger security refactor (170), then the API (113). Each gets its own worktree.

**Helios and Martina** work independently in their own desk clones — no worktrees needed (no code, no migrations).

---

## Worktree Setup — Claire

Each ticket gets an isolated git worktree. Create only when ready to start that ticket.

```bash
# ISS-171 (start first)
cd C:/temp/ClaudeProjects/development/presepi-site
git worktree add ../presepi-site-iss-171 -b feature/iss-171-remove-gif-allowlist develop

# ISS-170 (after ISS-171 merged)
git worktree add ../presepi-site-iss-170 -b feature/iss-170-mime-derived-extension develop

# ISS-113 (after ISS-170 merged)
git worktree add ../presepi-site-iss-113 -b feature/iss-113-contact-api develop
```

Remove each worktree after its branch is merged to develop:
```bash
git worktree remove ../presepi-site-iss-NNN
```

---

## Ticket Details

---

### ISS-171 — Security: remove image/gif from ALLOWED_MIME_TYPES

**Agent:** Claire (Backend Developer)
**Component:** core | **Urgency:** low
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Worktree:** `presepi-site-iss-171` | **Branch:** `feature/iss-171-remove-gif-allowlist`

**What this delivers:**
Remove `'image/gif'` from `ALLOWED_MIME_TYPES` in `core/validators.py`. GIF is not a valid format for nativity house product images. Reduces upload attack surface.

**Acceptance criteria:**
1. `'image/gif'` removed from `ALLOWED_MIME_TYPES`
2. Existing test updated to assert GIF is rejected (not accepted)
3. All other MIME type tests still pass

**Allowed paths:**
```
django/core/validators.py
django/core/tests/test_validators.py   (or wherever upload validation tests live)
```

**Notes verbatim (Dominick):**
> GIF is in ALLOWED_MIME_TYPES but nativity house product images are not GIFs. GIF adds unnecessary surface: GIF decompression bombs (mitigated by 20MB nginx limit but still adds risk) and legacy GIF XSS vectors. Simple fix: remove 'image/gif' from ALLOWED_MIME_TYPES in core/validators.py and update the test to verify GIF is rejected.

**Watch for:** Verify test file location before touching it. Do not modify any other validator logic.

---

### ISS-170 — Security: derive stored image extension from validated MIME type

**Agent:** Claire (Backend Developer)
**Component:** core | **Urgency:** low
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Worktree:** `presepi-site-iss-170` | **Branch:** `feature/iss-170-mime-derived-extension`
**Base branch:** `develop` (not from ISS-171 branch — ISS-171 will have merged first)

**What this delivers:**
Refactor `randomise_filename()` (or equivalent upload handler) to derive the stored file extension from the validated MIME type, not from the user-supplied filename. Prevents a JPEG uploaded as `malware.php` from being stored as `<uuid>.php`.

**MIME → extension map (post ISS-171):**
```
image/jpeg  → .jpg
image/png   → .png
image/webp  → .webp
```
GIF is removed (ISS-171). No other MIME types are in scope.

**Acceptance criteria:**
1. `randomise_filename()` (or upload handler) uses MIME-mapped extension, not `os.path.splitext(user_filename)[1]`
2. MIME map covers jpeg/png/webp only (gif removed)
3. Tests: each valid MIME type produces the correct extension; unknown MIME type is rejected
4. Existing upload tests still pass

**Allowed paths:**
```
django/core/validators.py          (or wherever randomise_filename lives)
django/core/storage.py             (if upload_to logic lives there — check first)
django/core/tests/                 (tests for validators/storage)
```

**Notes verbatim (Dominick):**
> Current: randomise_filename() uses os.path.splitext(user_filename)[1]. A JPEG with .php extension stores as <uuid>.php. Nginx does not execute it (no php-fpm; static-only /media/), but extension is user-controlled. Fix: map validated MIME to canonical ext and use that instead of user-supplied extension. Requires upload_to function to have MIME access — implementation design TBD.

**Watch for:** At G1, Claire must confirm whether the upload handler has access to the MIME type at filename-generation time. If not, she must propose how to pass it through — flag design decision before writing any code.

---

### ISS-113 — Contact API: POST /api/contact/

**Agent:** Claire (Backend Developer)
**Component:** api | **Urgency:** high
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Worktree:** `presepi-site-iss-113` | **Branch:** `feature/iss-113-contact-api`
**Base branch:** `develop`

**What this delivers:**
A `POST /api/contact/` endpoint that creates a `ContactEnquiry` record and sends an email notification to Gabriele. Products are individual nativity houses — contact form may be pre-filled with a product name from the catalogue page CTA.

**Acceptance criteria:**
1. `POST /api/contact/` accepts: `name`, `email`, `message`, `locale`, `product_name` (optional)
2. Creates a `ContactEnquiry` record in the database
3. Sends an email to Gabriele's address (configured via env var `CONTACT_EMAIL_RECIPIENT`)
4. Returns `201` on success; `400` with field errors on invalid input
5. GDPR: `ContactEnquiry` has a `created_at` timestamp (for data retention enforcement)
6. Tests: valid submission creates record + email sent (use Django's `mail.outbox`); invalid input returns 400; missing optional `product_name` is handled gracefully
7. SMTP config via env vars only — no hardcoded addresses or credentials

**Allowed paths:**
```
django/contact/views.py
django/contact/serializers.py
django/contact/urls.py
django/contact/tests/
django/sitopresepi/urls.py        (to wire the new endpoint)
```

**Watch for:**
- `ContactEnquiry` model already exists (ISS-103). Claire reads the existing model before designing the serializer — do not redefine it.
- At G1: confirm which email backend Django is configured to use (check `settings.py`) and whether a test email backend is already configured for tests.
- The `locale` field is a FK to `localisation.Language` (ISS-154).

---

### ISS-115 — Content: contact page copy IT/EN/DE

**Agent:** Helios (Content Strategist)
**Component:** content | **Urgency:** medium
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Desk:** Helios-Desk (`docs`, `presepi-site`)
**Branch:** `feature/iss-115-contact-page-copy` from `develop` in presepi-site clone

**What this delivers:**
All UI copy for the contact page in Italian, English, and German. Delivered as additions to the existing i18n locale files.

**Acceptance criteria:**
1. Page heading (IT/EN/DE)
2. Introductory text (1–2 sentences, warm and personal — consistent with brand voice)
3. Form labels: Name, Email, Message, Product (optional pre-fill label)
4. Submit button label
5. Success message (after submission)
6. Error message (generic form error)
7. German: CTAs may use „Sie" (formal); introductory text uses „du" (informal) — per constitution 2026-03-24

**Allowed paths:**
```
nextjs/messages/it.json    (or equivalent locale file structure)
nextjs/messages/en.json
nextjs/messages/de.json
```

**Watch for:** At G1, Helios must confirm the locale file structure used in the project (check `nextjs/messages/` or `nextjs/locales/` — verify which pattern ISS-112 used, then follow the same pattern exactly).

**Note for Bea:** This ticket has no code and no tests. G2 (red tests) is not applicable — skip directly from G1 to copy delivery. At G3: Helios commits the locale file additions and advances to `in-testing`.

---

### ISS-168 — SEO: EN keyword research

**Agent:** Martina (SEO Specialist)
**Component:** content | **Urgency:** medium
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Desk:** Martina-Desk (`docs`, `presepi-site`)

**What this delivers:**
A written recommendation on the correct English search term for Gabriele's product, with search volume rationale. Output is a short document committed to `docs/`.

**Product clarification (Owner, 2026-03-28):** Gabriele makes individual nativity **houses** (buildings/structures for the presepio scene), not full nativity scenes. EN copy and slug should reflect this distinction.

**Acceptance criteria:**
1. Comparison of candidate EN terms: "nativity house", "nativity stable", "nativity building", and any others Martina identifies as relevant
2. Search volume assessment for each (relative sizing acceptable if exact tools unavailable)
3. A clear single recommendation with rationale
4. Flag if `/en/catalogue/` URL slug needs to change based on the recommendation (current slug is neutral — no change may be required)
5. Flag any impact on ISS-169 (meta title/description) — Martina owns both

**Allowed paths:**
```
docs/seo/keyword-research-en-catalogue.md    (new file)
```

**Note for Bea:** Research/documentary task. G2 (red tests) not applicable — skip from G1 to research delivery. At G3: Martina commits the document and advances to `in-testing`. ISS-169 is blocked on this ticket being **resolved** (Owner sign-off), not just `in-testing`.

---

### ISS-169 — SEO: meta title and description copy (IT/EN/DE)

**Agent:** Martina (SEO Specialist)
**Component:** content | **Urgency:** medium
**Blocked by:** ISS-168 (must be resolved first)
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris

**What this delivers:**
`<title>` and `<meta name="description">` copy for catalogue list and product detail pages in all three locales.

**Acceptance criteria:**
1. Meta title for `/[locale]/catalogue/` (IT/EN/DE)
2. Meta description for `/[locale]/catalogue/` (IT/EN/DE)
3. Meta title template for `/[locale]/catalogue/[slug]/` (IT/EN/DE)
4. Meta description template for `/[locale]/catalogue/[slug]/` (IT/EN/DE)
5. EN terms aligned with ISS-168 recommendation
6. All within character limits: title ≤ 60 chars, description ≤ 155 chars
7. Helios reviews copy tone; Martina owns keyword alignment

**Do not start until ISS-168 is `resolved`.**

---

### ISS-114 — Frontend: contact page /[locale]/contact/

**Agent:** Ash (Frontend Developer)
**Component:** frontend | **Urgency:** high
**Blocked by:** ISS-113 (must be at least `in-testing`)
**G5 reviewer:** Alessandro | **G6 reviewer:** Chris
**Desk:** Ash-Desk (`presepi-site`)
**Branch:** `feature/iss-114-contact-page` from `develop`

**What this delivers:**
The contact page at `/[locale]/contact/` with a form that pre-fills `product_name` when navigated from a product CTA. Wires to the `POST /api/contact/` endpoint (ISS-113).

**Acceptance criteria:**
1. Route `/[locale]/contact/` renders in IT/EN/DE
2. Form fields: name, email, message, product (optional, pre-filled via query param `?product=slug`)
3. Form submits to `POST /api/contact/` and shows success/error state
4. Uses copy from ISS-115 locale files
5. Consistent with design tokens (Tailwind config from ISS-108)
6. At least 2 tests: page renders; product pre-fill populates field

**Watch for:** At G1, Ash confirms the contact API base URL (via `NEXT_PUBLIC_API_URL` env var) and the query param convention for product pre-fill.

**Do not start until ISS-113 is `in-testing` and its branch is pushed.**

---

## Reviewer Assignments Summary

| Ticket | Agent | G5 | G6 |
|---|---|---|---|
| ISS-171 | Claire | Alessandro | Chris |
| ISS-170 | Claire | Alessandro | Chris |
| ISS-113 | Claire | Alessandro | Chris |
| ISS-115 | Helios | Alessandro | Chris |
| ISS-168 | Martina | Alessandro | Chris |
| ISS-169 | Martina | Alessandro | Chris |
| ISS-114 | Ash | Alessandro | Chris |

---

## Gate Output Paths

| Ticket | G1 | G2 | G3 |
|---|---|---|---|
| ISS-171 | `backoffice/gate-output/ISS-171-G1.md` | `backoffice/gate-output/ISS-171-G2.md` | `backoffice/gate-output/ISS-171-G3.md` |
| ISS-170 | `backoffice/gate-output/ISS-170-G1.md` | `backoffice/gate-output/ISS-170-G2.md` | `backoffice/gate-output/ISS-170-G3.md` |
| ISS-113 | `backoffice/gate-output/ISS-113-G1.md` | `backoffice/gate-output/ISS-113-G2.md` | `backoffice/gate-output/ISS-113-G3.md` |
| ISS-115 | `backoffice/gate-output/ISS-115-G1.md` | *(n/a — content task)* | `backoffice/gate-output/ISS-115-G3.md` |
| ISS-168 | `backoffice/gate-output/ISS-168-G1.md` | *(n/a — research task)* | `backoffice/gate-output/ISS-168-G3.md` |
| ISS-169 | `backoffice/gate-output/ISS-169-G1.md` | *(n/a — content task)* | `backoffice/gate-output/ISS-169-G3.md` |
| ISS-114 | `backoffice/gate-output/ISS-114-G1.md` | `backoffice/gate-output/ISS-114-G2.md` | `backoffice/gate-output/ISS-114-G3.md` |

## G5 Routing Note Paths

| Ticket | Path |
|---|---|
| ISS-171 | `backoffice/g5-queue/ISS-171-routing.md` |
| ISS-170 | `backoffice/g5-queue/ISS-170-routing.md` |
| ISS-113 | `backoffice/g5-queue/ISS-113-routing.md` |
| ISS-115 | `backoffice/g5-queue/ISS-115-routing.md` |
| ISS-168 | `backoffice/g5-queue/ISS-168-routing.md` |
| ISS-169 | `backoffice/g5-queue/ISS-169-routing.md` |
| ISS-114 | `backoffice/g5-queue/ISS-114-routing.md` |

---

## Ticket CLI Reference

```bash
# Activate tools venv first
source C:/temp/ClaudeProjects/development/tools/.venv/Scripts/activate

# View a ticket
python C:/temp/ClaudeProjects/development/tools/ticket.py view ISS-NNN

# Record a gate note
python C:/temp/ClaudeProjects/development/tools/ticket.py update ISS-NNN \
  --field notes --value "2026-03-28 Bea: [G1 APPROVED] ..." --caller team-leader

# Advance to in-testing (agent does this at G3)
python C:/temp/ClaudeProjects/development/tools/ticket.py update ISS-NNN \
  --field status --value in-testing --caller backend   # or frontend / content / seo
```
