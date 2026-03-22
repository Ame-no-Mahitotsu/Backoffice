# Expedition Dispatches
Chronological journal of the SitoPresepi2 project. Append-only — corrections are new entries, not edits.

**ID format:** `EXP-NNN` (sequential)
**Labels:** see `glossary.md` for the full label vocabulary

---

## [EXP-001] 2026-03-11 — Constitutional Foundations
labels: #landmark #governance

Ratified the project constitution. Established governing principles, core epistemic rules, and the amendment process. Key decisions: the AI Orchestrator may raise Registered Concerns but the Project Owner's decision is always final. Honesty, intellectual humility, and evidence-based reasoning are binding on all participants.

## [EXP-006] 2026-03-12 — Constitutional Amendment: Governing Principles 6–9 + Amendment Scope
labels: #landmark #governance #constitution

Four new governing principles enshrined: TDD mandatory (no exceptions), No Monolith (parts must be independently replaceable), No Credentials in Code (environment variables only), GDPR Compliance Before Launch (Italy, UK, Germany). Amendment process scope clarified: Decisions Log updates are routine record-keeping and do not require the amendment process. Constitution last amended 2026-03-12. Approved by Owner + Orchestrator.

## [EXP-005] 2026-03-12 — Individual Sessions: Full Stack Decisions
labels: #landmark #architecture #stack #security #devops #testing

Four individual sessions completed: Dominick (security baseline), Patrick (architecture plan), Lota (UX direction), Alessandro (deployment + testing). Key decisions: DigitalOcean Frankfurt hosting, TDD from day one (pytest + Jest), Docker Compose deployment, SSG with revalidation for Next.js, local filesystem for images with django-storages abstraction. Lota pending Gabriele's answers on visual identity. All session notes in memory/workspace/.

## [EXP-004] 2026-03-12 — Architecture Debate: Monolith Rejected, New Recommendation
labels: #landmark #architecture #stack #frontend #security

Full team met: Patrick, Alessandro, Lota, Ash, Fran, Dominick. Original Django + Wagtail decision challenged on replaceability grounds. Owner confirmed: no monolith, parts must be independently replaceable. Three paths debated. Dominick validated Path 2 from security standpoint. Final recommendation: Django REST Framework + Next.js + custom admin + Nginx + Docker on VPS. Owner to hold individual discussions with each team member before final approval. Transcript: memory/workspace/meeting_full_team_001.md.

## [EXP-003] 2026-03-12 — Stack Decision: Django + Wagtail
labels: #landmark #architecture #stack

Patrick (Architect) and Alessandro (Tech Lead) met to decide the technical stack. After evaluating three options — headless CMS, Django + Wagtail, and custom stack — plus a third contender (Laravel + Filament), both aligned on Django + Wagtail self-hosted on a VPS. Key drivers: open source, integrated admin, localisation built-in, extensible, €5–20/month hosting, no vendor lock-in. Approved by Project Owner. Full transcript in memory/workspace/meeting_architect_techlead_001.md.

## [EXP-002] 2026-03-11 — Project Document System Established
labels: #landmark #governance

Created the three core project documents: Expedition Dispatches (this file), Issue Backlog, and Glossary. Agreed on EXP-NNN sequential IDs for dispatches, Type + Tags structure for the backlog, and the Glossary as the single source of term definitions. The Expedition Dispatches theme was chosen over Star Trek / Captain's Log alternatives.
