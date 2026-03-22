# Meeting Minutes — BAR-001

ID: BAR-001
Type: ba-review
Tags: admin-roles, rbac, artisan-access, session-timeout, authentication, totp, c3-correction
Date: 2026-03-13
Chair: Orchestrator
Participants: Fran, Dominick, Owner
Ticket: —
Sprint: —
Status: closed
Project: site

---

## Background

Architecture constraint C3 was written as "Admin operable by non-technical user" based on Fran's BA requirement NFR-02. Post-ARQ-001, the Owner confirmed this was a false assumption: the site is operated by the Owner (technical professional) permanently. No handover to a non-technical party is planned.

This meeting was called to:
1. Confirm Gabriele's actual access needs (he is the artisan, not the operator)
2. Define the correct admin role model
3. Update BA requirements to remove the contaminated assumption

---

## Decisions Taken

| Ref | Decision | Approved |
|---|---|---|
| BAR-D1 | Two admin roles: Django superuser (Owner) + `artisan` Django Group (Gabriele). No third role. | Yes |
| BAR-D2 | Gabriele's access scope: upload images, edit text content, view orders and sales figures. He cannot manage products, users, languages, or site configuration. | Yes |
| BAR-D3 | Gabriele acts independently for his permitted actions — no owner approval step required for content uploads and text edits. | Yes |
| BAR-D4 | TOTP (django-otp) required for ALL admin users, including Gabriele. No exceptions. | Yes |
| BAR-D5 | Session inactivity timeouts: Owner = 1h (`ADMIN_SESSION_TIMEOUT_OWNER=3600`); Gabriele = 2h (`ADMIN_SESSION_TIMEOUT_ARTISAN=7200`). Both env-configurable via `.env`. | Yes |
| BAR-D6 | C3 already revised in ARQ-001. This meeting confirms the downstream BA documents must now be updated to reflect the actual operator model. | Yes |

---

## Rationale — Session Timeouts (Dominick)

- Both timeouts are **inactivity-based** — they reset on every HTTP request. Active sessions are not interrupted.
- Owner (superuser) at 1h: shortest window justified by highest privilege level. 1h is a standard enterprise baseline for privileged accounts.
- Gabriele (artisan group) at 2h: lower privilege, content-only access; slightly longer window is acceptable.
- Both are env-var driven — trivial to reduce further in production if desired.

---

## Rationale — TOTP for Gabriele (Dominick)

- Any login to the admin panel is a privileged action regardless of the group's permission scope.
- Credential compromise at Gabriele's account could enable content defacement or data visibility of orders.
- TOTP friction is acceptable — Gabriele's sessions are not frequent; setup is a one-time operation.
- Consistent policy is simpler to maintain and audit than a two-tier auth policy.

---

## BA Action Items

| Action | Owner | Status |
|---|---|---|
| Rewrite U3 in `ba_requirements.md` | Fran | Done — see updated file |
| Rewrite NFR-02 in `ba_requirements.md` | Fran | Done — see updated file |

---

## Open Items — Closed (2026-03-13)

| Item | Owner | Resolution |
|---|---|---|
| Patrick: review architecture §2, §13 for remaining C3-influenced decisions | Patrick | Done — §2 and §13 clean; AI TDD constraint added to §13 |
| Ash: review admin UX direction for non-technical-user assumptions | Ash | Done — no non-technical-user assumptions found |
| Group check-in after individual reviews | All | Done — clean close confirmed 2026-03-13 |

All action items resolved. BAR-001 fully closed.

---

*Minutes recorded by Orchestrator. Approved by Owner 2026-03-13.*
