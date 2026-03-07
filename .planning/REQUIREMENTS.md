# Requirements: Ruck On — Theme Enhancement Milestone

**Defined:** 2026-03-06
**Core Value:** Merchants can tell the Ruck On story and build new pages entirely through the Shopify theme editor — no code required.

## v1 Requirements

### About Us Page

- [ ] **ABOUT-01**: User can view a brand origin story section with heading, richtext body, and optional image
- [ ] **ABOUT-02**: User can view a mission/ethos section with eyebrow text, headline, and supporting body text ("Carry the Weight" philosophy)
- [ ] **ABOUT-03**: User can view a team/people section with repeating person cards (photo, name, title, bio)
- [ ] **ABOUT-04**: Merchant can edit all About Us content from the Shopify theme editor (no hardcoded text)
- [ ] **ABOUT-05**: All About Us images are editor-uploadable with placeholder fallbacks (no hardcoded assets)
- [ ] **ABOUT-06**: Merchant can assign the About Us template to any page via Shopify admin template picker

### Image + Text Block

- [ ] **ITEXT-01**: Merchant can drag an image+text block onto any page using the theme editor
- [ ] **ITEXT-02**: Merchant can toggle image position (left or right of text content)
- [ ] **ITEXT-03**: Merchant can control image width proportion (narrow / half / wide)
- [ ] **ITEXT-04**: Block supports heading, body text, and an optional CTA button with configurable label and URL
- [ ] **ITEXT-05**: Block stacks vertically on mobile (image above text)

### Multi-Column Text Block

- [ ] **MCOL-01**: Merchant can drag a multi-column block onto any page using the theme editor
- [ ] **MCOL-02**: Merchant can set column count (2 or 3 columns)
- [ ] **MCOL-03**: Each column supports an optional icon/image, heading, and body text
- [ ] **MCOL-04**: Block collapses to single-column on mobile

### Testimonial / Quote Block

- [ ] **QUOTE-01**: Merchant can drag a testimonial block onto any page using the theme editor
- [ ] **QUOTE-02**: Each testimonial displays quote text, author name, and a star rating (1–5 stars)
- [ ] **QUOTE-03**: Each testimonial supports an optional reviewer photo
- [ ] **QUOTE-04**: Multiple testimonials display in a grid layout (no carousel)

### Collection Grid Block

- [ ] **CGRID-01**: Merchant can drag a collection grid block onto any page using the theme editor
- [ ] **CGRID-02**: Merchant can pick which Shopify collection to display
- [ ] **CGRID-03**: Merchant can set column count (2, 3, or 4 columns)
- [ ] **CGRID-04**: Merchant can set a product limit (how many products to show)
- [ ] **CGRID-05**: Block uses existing `.product-card` styles from `critical.css` for visual consistency

### Collection Page Controls

- [ ] **COLL-01**: Merchant can set desktop column count (2–4 columns) from the theme editor on the collection page
- [ ] **COLL-02**: Merchant can set mobile column count (1–2 columns) from the theme editor on the collection page
- [ ] **COLL-03**: Merchant can set product image ratio (adapt / portrait / square) from the theme editor
- [ ] **COLL-04**: Merchant can set card style (standard / minimal) from the theme editor
- [ ] **COLL-05**: Default settings reproduce the current collection page appearance (zero visual regression for existing store)

## v2 Requirements

### About Us Enhancements

- **ABOUT-V2-01**: Video embed in brand origin story section (storytelling upgrade)
- **ABOUT-V2-02**: Press/media logos bar ("As seen in" section)

### Block Enhancements

- **ITEXT-V2-01**: Full-width image option on image+text block
- **ITEXT-V2-02**: Vertical alignment control (top / center / bottom) on image+text block
- **QUOTE-V2-01**: Optional carousel/slider layout for testimonials

### Collection Enhancements

- **COLL-V2-01**: Secondary image on hover for product cards on collection page
- **COLL-V2-02**: Filtering sidebar (by size, color, etc.) on collection page

## Out of Scope

| Feature | Reason |
|---------|--------|
| Testimonial carousel | High JS complexity, accessibility burden; grid of 3 is sufficient for v1 |
| Animated counting numbers | Already handled by existing `stats-bar.liquid` section |
| Social media feed embeds | Third-party API fragility, script performance cost |
| Tab-switching About Us layout | Hides content; zero conversion benefit |
| Checkout customization | Outside Shopify theme scope |
| Customer account redesign | Not requested |
| Mobile app or headless | Web Shopify theme only |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| ITEXT-01 | Phase 1 | Pending |
| ITEXT-02 | Phase 1 | Pending |
| ITEXT-03 | Phase 1 | Pending |
| ITEXT-04 | Phase 1 | Pending |
| ITEXT-05 | Phase 1 | Pending |
| MCOL-01 | Phase 1 | Pending |
| MCOL-02 | Phase 1 | Pending |
| MCOL-03 | Phase 1 | Pending |
| MCOL-04 | Phase 1 | Pending |
| QUOTE-01 | Phase 1 | Pending |
| QUOTE-02 | Phase 1 | Pending |
| QUOTE-03 | Phase 1 | Pending |
| QUOTE-04 | Phase 1 | Pending |
| CGRID-01 | Phase 1 | Pending |
| CGRID-02 | Phase 1 | Pending |
| CGRID-03 | Phase 1 | Pending |
| CGRID-04 | Phase 1 | Pending |
| CGRID-05 | Phase 1 | Pending |
| ABOUT-01 | Phase 2 | Pending |
| ABOUT-02 | Phase 2 | Pending |
| ABOUT-03 | Phase 2 | Pending |
| ABOUT-04 | Phase 2 | Pending |
| ABOUT-05 | Phase 2 | Pending |
| ABOUT-06 | Phase 2 | Pending |
| COLL-01 | Phase 3 | Pending |
| COLL-02 | Phase 3 | Pending |
| COLL-03 | Phase 3 | Pending |
| COLL-04 | Phase 3 | Pending |
| COLL-05 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 29 total
- Mapped to phases: 29
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-06*
*Last updated: 2026-03-06 after initial definition*
