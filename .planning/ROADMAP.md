# Roadmap: Ruck On — Theme Enhancement Milestone

## Overview

This milestone delivers two interconnected capabilities on top of the existing custom Shopify theme: a drag-and-drop block toolkit that any merchant can use on any page, and a fully editor-configurable About Us page. The block toolkit is built first because the About Us page sections host those blocks. Collection page editor controls are self-contained and ship last. Every phase delivers a coherent, verifiable capability that a merchant can use immediately.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (e.g., 2.1): Urgent insertions, created via /gsd:insert-phase

- [ ] **Phase 1: Block Toolkit** - Four reusable drag-and-drop blocks available on any page in the store
- [ ] **Phase 2: About Us Page** - Fully merchant-configurable About Us page with branded sections
- [ ] **Phase 3: Collection Controls** - Column count, image ratio, and card style configurable from the collection page editor

## Phase Details

### Phase 1: Block Toolkit
**Goal**: Merchants can drag four reusable content blocks onto any page from the theme editor
**Depends on**: Nothing (first phase)
**Requirements**: ITEXT-01, ITEXT-02, ITEXT-03, ITEXT-04, ITEXT-05, MCOL-01, MCOL-02, MCOL-03, MCOL-04, QUOTE-01, QUOTE-02, QUOTE-03, QUOTE-04, CGRID-01, CGRID-02, CGRID-03, CGRID-04, CGRID-05
**Success Criteria** (what must be TRUE):
  1. Merchant can open the theme editor, find all four blocks in the "Add block" picker, and drag each onto a page
  2. The image+text block renders with image left or right, adjustable width proportion, and a heading + body + optional CTA button that stacks vertically on mobile
  3. The multi-column block renders 2 or 3 columns on desktop (each with optional icon/image, heading, and body text) and collapses to single-column on mobile
  4. The testimonial block renders quote text, author name, star rating (1-5 stars), and optional reviewer photo in a grid layout
  5. The collection grid block renders products from a merchant-selected collection at the configured column count and product limit, using the existing product card styles
**Plans**: TBD

### Phase 2: About Us Page
**Goal**: Merchants can build and edit a complete About Us page entirely from the theme editor
**Depends on**: Phase 1
**Requirements**: ABOUT-01, ABOUT-02, ABOUT-03, ABOUT-04, ABOUT-05, ABOUT-06
**Success Criteria** (what must be TRUE):
  1. Visiting /pages/about-us renders an origin story section with heading, richtext body, and an optional image — all editable from the theme editor
  2. The page shows a mission/ethos section with eyebrow text, headline, and supporting body text reflecting the "Carry the Weight" philosophy
  3. The page shows a team section with repeating person cards (photo, name, title, bio) where merchant can add/remove/reorder team members from the editor
  4. All image fields show Shopify placeholder SVGs when no image is uploaded; no broken image states exist
  5. Merchant can assign the About Us template to any page via the Shopify admin template picker without touching code
**Plans**: TBD

### Phase 3: Collection Controls
**Goal**: Merchants can configure the collection page grid layout and card appearance from the theme editor
**Depends on**: Nothing (independent — can run after Phase 1 or 2)
**Requirements**: COLL-01, COLL-02, COLL-03, COLL-04, COLL-05
**Success Criteria** (what must be TRUE):
  1. Merchant can change desktop column count (2, 3, or 4) from the collection page editor and see the grid update live
  2. Merchant can change mobile column count (1 or 2) and image ratio (adapt/portrait/square) from the editor
  3. Merchant can switch card style (standard/minimal) and see the product cards update without a code change
  4. A store that has never touched these settings renders the collection page identically to how it looked before this phase shipped (zero visual regression)
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Block Toolkit | 0/? | Not started | - |
| 2. About Us Page | 0/? | Not started | - |
| 3. Collection Controls | 0/? | Not started | - |
