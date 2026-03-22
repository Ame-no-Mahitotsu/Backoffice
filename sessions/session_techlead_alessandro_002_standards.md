---
ID: session_techlead_alessandro_002_standards
Type: session
Role: tech-lead
Topic: Development standards — repo layout, TypeScript, branching, Python toolchain, Conventional Commits, code review
Date: 2026-03-13
Participants: Owner, Alessandro
Ticket: —
Sprint: —
Status: closed
Project: site
Outputs: ai-standards.md updated (coding standards, test standards, branch model, code review), 6 decisions in constitution.md
---

# Session: Alessandro (Tech Lead) — Development Standards
**Date:** 2026-03-13
**Status:** Approved by Project Owner

---

## Decisions Made This Session

| # | Decision | Choice | Approved |
|---|---|---|---|
| 1 | Repository | Local bare repo — `SitoPresepi2/remote.git` as origin, `SitoPresepi2-src/` as working clone | Owner |
| 2 | Frontend language | TypeScript, strict mode | Owner |
| 3 | Branching model | Lightweight GitFlow: `main` / `develop` / `feature/*` / `hotfix/*` | Owner |
| 4 | Python toolchain | Black + isort + Ruff + pre-commit hooks | Owner |
| 5 | Commit format | Conventional Commits | Owner |
| 6 | Code review | Two-reviewer model: Tech Lead + domain specialist (Backend agent for Django PRs, Frontend agent for Next.js PRs) | Owner |

---

## Repository Setup

### Structure
```
c:\temp\ClaudeProjects\
├── SitoPresepi2\              ← project management (constitution, memory, AI standards)
│   └── remote.git\            ← bare git repository — the authoritative "remote"
│
└── SitoPresepi2-src\          ← working clone — all code written here
    ├── django\
    ├── nextjs\
    ├── docker-compose.yml      (symlink or copy from SitoPresepi2/)
    ├── nginx\
    └── .env.example
```

### Initialisation commands (to be run when ready to start coding)
```bash
# 1. Create the bare repo (the "remote")
git init --bare c:/temp/ClaudeProjects/SitoPresepi2/remote.git

# 2. Clone into the working directory
git clone c:/temp/ClaudeProjects/SitoPresepi2/remote.git c:/temp/ClaudeProjects/SitoPresepi2-src

# 3. Set default branch to main
cd c:/temp/ClaudeProjects/SitoPresepi2-src
git checkout -b main
git checkout -b develop

# 4. Push both branches to remote
git push -u origin main
git push -u origin develop
```

### Migrating to GitHub (when CI/CD is added in v2)
```bash
# Add GitHub as a second remote — keeps local bare repo intact
git remote add github git@github.com:<username>/sitopresepi2.git
git push github --all
git push github --tags
# From that point, use "github" as the primary remote
```

---

## Branching Strategy

```
main        Production-only. Never committed to directly.
            Tagged on every deployment: v0.1.0, v0.2.0 etc.

develop     Integration branch. All tests must pass here at all times.
            Red tests on develop = team's top priority.

feature/*   One branch per task. Branched from develop. Merged to develop via PR.
            Short-lived: hours to days, never weeks.

hotfix/*    Production fixes only. Branched from main.
            Merged to BOTH main AND develop after fix.
```

### Branch naming
```
feature/catalogue-models
feature/api-products-endpoint
feature/nextjs-product-list-page
fix/product-slug-collision
test/order-flow-e2e
chore/update-dependencies
hotfix/nginx-ssl-renewal
```

### Rules
- No direct commits to `main` or `develop`
- PRs from `feature/*` → `develop` only
- Tests must be green before any merge — no exceptions (Constitution §6)
- One logical unit of work per branch — no scope creep mid-branch

---

## Coding Standards

### Python / Django

**Toolchain**
| Tool | Purpose |
|---|---|
| `black` | Formatting — zero config, no arguments |
| `isort` | Import ordering (Black-compatible profile) |
| `ruff` | Linting — replaces flake8, fast, comprehensive |

**pre-commit config (`.pre-commit-config.yaml` in repo root)**
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.x.x
    hooks:
      - id: black
        language_version: python3.12
  - repo: https://github.com/pycqa/isort
    rev: 5.x.x
    hooks:
      - id: isort
        args: ["--profile", "black"]
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.x.x
    hooks:
      - id: ruff
        args: ["--fix"]
```

**Naming conventions**
```python
# Files/modules:   snake_case.py
# Classes:         PascalCase
# Functions/vars:  snake_case
# Constants:       SCREAMING_SNAKE_CASE
# Private:         _leading_underscore
# URL patterns:    kebab-case strings
```

**App structure — every Django app**
```
app_name/
├── __init__.py
├── models.py          # Models only. No business logic.
├── services.py        # All business logic. Tested independently.
├── serializers.py     # DRF serializers.
├── views.py           # Thin. Calls services. No logic.
├── urls.py
└── tests/
    ├── __init__.py
    ├── test_models.py
    ├── test_services.py
    └── test_views.py
```

**No business logic in views — enforced in every review**
```python
# WRONG
class ProductViewSet(viewsets.ReadOnlyModelViewSet):
    def get_queryset(self):
        return Product.objects.filter(is_active=True, ...)  # logic in view

# CORRECT
class ProductViewSet(viewsets.ReadOnlyModelViewSet):
    def get_queryset(self):
        return ProductService.get_active_products(
            language_code=self.request.LANGUAGE_CODE
        )
```

---

### TypeScript / Next.js

**Toolchain**
| Tool | Purpose |
|---|---|
| `typescript` | Strict mode (`"strict": true` in tsconfig) |
| `prettier` | Formatting |
| `eslint` | Linting — Next.js recommended + custom rules |

**Naming conventions**
```typescript
// Files:
ProductCard.tsx          // Components: PascalCase
useProductList.ts        // Hooks: camelCase, "use" prefix
productService.ts        // Utilities: camelCase
types.ts                 // Shared types

// Types/interfaces: PascalCase, no I/T prefix
interface Product { ... }
type ProductStatus = 'available' | 'made_to_order' | 'unavailable'

// Variables/functions: camelCase
const activeProducts = ...
function fetchProducts() {}

// Constants: SCREAMING_SNAKE_CASE
const MAX_CART_ITEMS = 10
```

**Folder structure**
```
nextjs/
├── app/
│   └── [locale]/
│       ├── layout.tsx
│       ├── page.tsx
│       └── catalogo/page.tsx   # language-specific URL slugs
├── components/
│   ├── catalogue/              # domain-grouped
│   ├── cart/
│   ├── gallery/
│   └── ui/                     # generic primitives only
├── lib/
│   ├── api.ts                  # Django API client
│   ├── i18n.ts                 # localisation helpers
│   └── types.ts                # TypeScript types mirroring Django models
└── public/
```

**`lib/types.ts` is the API contract file.** Every type here must match the Django serializer output. Divergence between them = a bug. The Backend and Frontend agents are both responsible for keeping these in sync.

---

## Testing Standards

### Django — test naming
```python
# Pattern: test_<subject>_<condition>_<expected_result>
def test_product_without_translation_falls_back_to_italian(): ...
def test_order_submit_with_invalid_email_returns_400(): ...
def test_admin_login_after_5_failures_returns_locked(): ...
```

### Django — test structure (AAA)
```python
def test_active_products_excludes_inactive():
    # Arrange
    Product.objects.create(is_active=True, ...)
    Product.objects.create(is_active=False, ...)
    # Act
    result = ProductService.get_active_products(language_code="it")
    # Assert
    assert result.count() == 1
```

### Django — fixtures
- Use pytest fixtures in `conftest.py`, not Django XML fixtures
- App-level `conftest.py` for app-specific fixtures
- Root-level `conftest.py` for shared fixtures (language objects, etc.)
- DB tests decorated with `@pytest.mark.django_db`

### Next.js — test location
Co-located with component:
```
components/catalogue/
├── ProductCard.tsx
└── ProductCard.test.tsx     # same folder
```

### Next.js — what gets tested
- Renders without crashing
- Displays correct content given props
- User interactions (clicks, form inputs, language switch)
- API data: correct rendering with mocked responses
- NOT: CSS, visual appearance, pixel-level layout

---

## Code Review Process

### Two-reviewer model
Every PR receives two independent reviews before merge:

| Reviewer | Scope |
|---|---|
| **Tech Lead** (`/tech-lead`) | Architecture alignment, app boundary violations, coding standards, security rules (Dominick's baseline), commit message format, PR description quality |
| **Domain specialist** | Django PRs → `/backend` agent. Next.js PRs → `/frontend` agent. Deep framework patterns, ORM correctness, React hook rules, ISR edge cases |

Reviews are **independent** — the domain specialist reads the code fresh, without seeing the Tech Lead's comments first. This prevents anchoring bias.

### PR description template
```
## What this does
[One paragraph — what behaviour does this add or fix?]

## Tests written
[List test function names and what each one proves]

## Architecture alignment
[Which section of the architecture document does this implement?]

## Anything unusual
[Any deviation from standards, and why]
```

### Merge checklist (Tech Lead enforces)
- [ ] PR description complete
- [ ] Tests written first (TDD — can be verified by commit order)
- [ ] All tests passing
- [ ] No business logic in views (Django)
- [ ] No secrets or credentials anywhere in diff
- [ ] No violations of app boundary rules (architecture document §5)
- [ ] Commit messages follow Conventional Commits format
- [ ] `lib/types.ts` updated if API response shapes changed

---

## Commit Message Format

**Conventional Commits**
```
<type>(<scope>): <imperative description, max 72 chars>

[optional body — explains WHY, not WHAT]

[optional footer — breaking changes, issue refs]
```

**Types:** `feat`, `fix`, `test`, `refactor`, `chore`, `docs`, `style`, `perf`

**Examples**
```
feat(catalogue): add Product model with localisation support
test(catalogue): add translation fallback test for missing language
fix(api): return 404 not 500 for unknown product slug
chore(deps): add black isort ruff to dev dependencies
feat(nextjs): add ProductCard component with price and size display
```

**Rules**
- Imperative mood: "add" not "added", "fix" not "fixed"
- No full stop at end of first line
- Body explains why, not what
