# Meeting Minutes — ARQ-001

ID: ARQ-001
Type: architecture-review
Tags: data-model, customers-app, payment-stub, audit-log, application-logging, page-body, css, tailwind
Date: 2026-03-13
Chair: Orchestrator
Participants: Patrick, Dominick, Alessandro, Ash, Owner
Ticket: —
Sprint: —
Status: closed
Project: site

---

## Decisions Taken

| Ref | Decision | Approved |
|---|---|---|
| D1 | `customers/` as dedicated Django app from day one; boundary rules documented in §5 | Yes |
| D2 | `Payment` stub added to data model in `commerce/`; fields: id, order FK, provider (nullable enum), status, amount, created_at | Yes |
| D3 | `AuditLog` model in `admin_app/`; append-only enforced at model + DB level; fields: id, actor, action, target_model, target_id, ip_address, timestamp | Yes |
| D3b | Application logging: Python `logging` module; modular namespaces (`sitopresepi.*`); level from `LOG_LEVEL` env var; ERROR→DEBUG range | Yes |
| D4 | `Page.body` = structured JSON blocks; typed block type enum; no freeform HTML | Yes (revised from Markdown during post-meeting owner review) |
| D5 | CSS: Tailwind CSS; `tailwind.config.ts` as design token source | Yes |
| D6 | Architecture constraint C3 revised: owner is sole technical operator for lifetime of project; no non-technical operator assumed; artisan is content subject only | Yes |

## Rejected

| Item | Reason |
|---|---|
| B2B / ShopOwner stub | No requirement in BA documents; no roadmap entry; rejected as speculative |

## Action Items

| Action | Owner | Triggered by |
|---|---|---|
| Review `ba_requirements.md` for all requirements that assumed a non-technical operator | Fran (BA) | D6 — C3 was a BA requirement; other assumptions may be contaminated |
| Review architecture §2, §13 for remaining C3-influenced decisions | Patrick | D6 |
| Review admin UX direction for non-technical-user assumptions | Ash | D6 |
| Group check-in after individual reviews to confirm nothing missed | All | D6 |
| Update architecture document — all D1–D6 changes | Orchestrator | This meeting — done |
| Log all decisions in constitution.md | Orchestrator | This meeting — done |

## Unprompted Finding (Chair)

Architecture constraint C7 required a payment seam, but no `Payment` model existed in the data model. Caught during chair's pre-meeting document review. Added as D2.

---

*Minutes recorded by Orchestrator. Approved by Owner 2026-03-13.*
