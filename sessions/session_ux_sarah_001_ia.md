---
ID: session_ux_sarah_001_ia
Type: session
Role: ux
Topic: Information architecture and design direction — page list, nav, user flows, The Maker page
Date: 2026-03-13
Participants: Owner, Sarah
Ticket: ISS-006, ISS-007
Sprint: 1
Status: closed
Project: site
Outputs: IA decisions D1-D4 approved, page list, nav spec, B2B/B2C user flows, 10 Gabriele interview questions (IT), ISS-006 ready, ISS-007 in-development
---

# Session: Sarah (UX/UI) — Information Architecture & Design Direction
**Date:** 2026-03-12 (started) / 2026-03-13 (IA session)
**Status:** In progress — 4 decisions pending German assistant confirmation + owner formal sign-off
**Tickets:** ISS-006 (IA), ISS-007 (Maker page interview)

---

## Key Inputs Confirmed

- **Tone:** Warm, rustichic — earthy, handmade feel without being rough. Workshop aesthetic, beautifully lit.
- **Artisan:** Gabriele — name to build the identity around
- **Visual identity:** None existing — building from zero
- **Primary device:** Desktop (more website visits expected on desktop)
- **Mobile:** Still fully supported, designed mobile-first

---

## Decisions Made (2026-03-13) — Pending formal owner sign-off + German assistant review

### D1 — URL slugs: locale-specific ✅ OWNER CONFIRMED
Owner overrode Sarah's v1 recommendation. Locale-specific slugs approved.

### D2 — Remove Cart/Checkout/Order Confirmation from v1 ⏳ PENDING
Sarah proposed; owner inclined to agree. Contact-first model replaces B2C order flow.
- P-07 Cart, P-08 Checkout, P-09 Order Confirmation → **removed from v1**
- Product Detail CTA changes from "Add to Cart" → "Sono interessato / I'm interested" → routes to Contact pre-filled with product name
- Rationale: artisan product; personal conversation IS the sale; reduces GDPR surface area; matches how orders actually happen
- Requires formal PO decision to remove PB-06 from v1 scope

### D3 — Nav labels ⏳ PENDING German assistant review
- IT: **Il Lavoro**
- EN: **The Making** (owner liked; "The Craft" was considered overused)
- DE: **Das Handwerk** ← flagged for native German speaker review

### D4 — Add "Chi sono / The Maker" page ⏳ PENDING
Sarah strongly recommended; owner inclined to agree.
- Rationale: the product IS Gabriele; personal story justifies premium; Home page cannot carry both routing and brand story without conflict
- Content: portrait of Gabriele at work (hands visible), short personal text, workshop image
- Sarah's note: "every product page lands differently when you know who made it"

---

## Confirmed Page List (pending D2 and D4 approval)

| # | Page | Route pattern (IT / EN / DE) | Primary user | Status |
|---|---|---|---|---|
| P-01 | Home | `/it/` / `/en/` / `/de/` | All | ✅ Confirmed |
| P-02 | Catalogue | `/it/catalogo/` / `/en/catalogue/` / `/de/katalog/` | B2B + B2C | ✅ Confirmed |
| P-03 | Product Detail | `/it/catalogo/[slug]/` / `/en/catalogue/[slug]/` / `/de/katalog/[slug]/` | B2B + B2C | ✅ Confirmed |
| P-04 | Gallery | `/it/galleria/` / `/en/gallery/` / `/de/galerie/` | All | ✅ Confirmed |
| P-05 | The Work / Il Lavoro | `/it/il-lavoro/` / `/en/the-making/` / `/de/das-handwerk/` | All | ✅ Confirmed (labels pending D3) |
| P-06 | Contact | `/it/contatti/` / `/en/contact/` / `/de/kontakt/` | B2B primary | ✅ Confirmed |
| P-06b | The Maker / Chi sono | `/it/chi-sono/` / `/en/the-maker/` / `/de/der-meister/` | All (trust) | ⏳ Pending D4 |
| P-07 | Cart | `/it/carrello/` / `/en/cart/` / `/de/warenkorb/` | B2C | ❌ Removed pending D2 |
| P-08 | Checkout | `/it/ordine/` / `/en/order/` / `/de/bestellung/` | B2C | ❌ Removed pending D2 |
| P-09 | Order Confirm | `/it/ordine/conferma/` etc. | B2C | ❌ Removed pending D2 |
| P-10 | Privacy Policy | `/it/privacy/` / `/en/privacy/` / `/de/datenschutz/` | Legal | ✅ Confirmed |
| P-11 | 404 | `not-found.tsx` — locale-aware | All | ✅ Confirmed |

Product slugs (`[slug]`) are language-neutral in v1 — single slug per product used across all locales.

---

## Navigation Spec

### Desktop
```
[Logo / Gabriele Presepi]    Catalogo  Galleria  Il Lavoro  Chi sono  Contatti    [IT|EN|DE]  [🛒]
```
- Logo left, nav items centre/right, language switcher + cart icon rightmost
- Cart icon: hidden when empty, shows count badge when ≥1 item (only relevant if D2 reversed later)
- No dropdowns in v1 — flat nav
- If D4 approved: "Chi sono" added as 5th nav item

### Mobile
- Sticky header: Logo left | Cart icon | Hamburger (right)
- Hamburger → full-width slide-down drawer: all nav items + language switcher below divider
- Cart icon always in sticky header (not inside drawer)

### Footer
```
Privacy Policy  |  Contatti  |  IT · EN · DE
© [year] Gabriele [Cognome TBD]
```

---

## User Flows

### B2B (revised — contact-first)
```
Arrive → Home → Catalogue → Product Detail → Contact
                   ↓                ↓
                Gallery     "Hai domande? Contattami"
                   ↓
                Contact
```
Contact always one click from anywhere via primary nav.

### B2C (revised — contact-first if D2 approved)
```
Arrive → Home → Catalogue → Product Detail → "Sono interessato" → Contact (pre-filled)
                                  ↓
                              Chi sono (trust building)
                                  ↓
                               Gallery
```
No cart, no checkout. Interest expressed via Contact. Gabriele follows up personally.

---

## Open Questions for Gabriele (5 visual questions — still outstanding)

1. Is there a place — a workshop, a town, a landscape — that feels connected to how you work?
2. If you had to describe your houses in three words, what would they be?
3. Are there any websites, shops, or brands — not necessarily presepi — whose look and feel you admire or find beautiful?
4. What materials do you use most? What does a house smell and feel like when you're building it?
5. Is there a house you've built that you're most proud of? Why that one?

---

## "The Maker" Page — Gabriele Interview Questions (ISS-007)

*Cultural framing: open narrative questions, not a questionnaire. Ask for stories, not descriptions. The best answers come from the second follow-up question.*

**Sul luogo e le radici**
1. Dove lavori? Descrivimi il posto — non come ci arrivi, ma cosa vedi quando sei lì e stai lavorando.
2. Questo mestiere ha radici nella tua famiglia, nel tuo paese, in qualcuno che hai conosciuto — oppure è una storia tua, che hai costruito da solo?

**Sul lavoro**
3. Come si comincia un presepe? Raccontami i primi gesti — dal niente al primo pezzo in mano.
4. Qual è il materiale con cui hai più feeling? Quello che senti sotto le dita e sai già come si comporterà?
5. C'è una parte del lavoro che trovi difficile ancora oggi — qualcosa che non è mai diventato automatico?

**Sui pezzi**
6. C'è un presepe che hai fatto e che ancora ti viene in mente? Non necessariamente il più bello — ma quello a cui sei rimasto più legato. Perché quello?
7. Hai mai fatto un pezzo per qualcuno e poi, quando glielo hai consegnato, hai visto la sua reazione? Cosa è successo?

**Sul senso del lavoro**
8. Quando qualcuno guarda un tuo presepe, cosa vorresti che sentisse — anche solo per un secondo?
9. Il presepe è una tradizione molto precisa, con le sue regole, la sua iconografia. Tu la rispetti, la reinterpreti, la ignori? Come ti rapporti con la tradizione?

**La domanda finale**
10. Se dovessi spiegare a qualcuno che non ha mai visto un presepe artigianale perché vale la pena averne uno — e non potessi mostrarglielo, solo parlargli — cosa gli diresti?

*Questions 1, 6, 7, and 10 will almost certainly form the core of the final page text.*

---

## Sarah's Next Actions (blocked on D2/D3/D4 approval)

1. Once D2–D4 formally approved: finalise page list and write ISS-006 deliverable file
2. Produce 2–3 mood board directions in words ("warm rustichic" baseline + Gabriele's visual answers when received)
3. Once mood board direction chosen: define design tokens (colours, typography, spacing) for Ash's `tailwind.config.ts`
4. Produce component specs: Navigation, ProductCard, GalleryGrid, ContactForm — enough for Ash to implement without guessing
