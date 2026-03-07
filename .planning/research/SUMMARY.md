# Project Research Summary

**Project:** Ruck On — Theme Enhancement Milestone (About Us page + drag-and-drop block toolkit)
**Domain:** Shopify Liquid theme development — reusable blocks, page templates, collection editor controls
**Researched:** 2026-03-06
**Confidence:** HIGH

## Executive Summary

This milestone delivers two interconnected deliverables on a Shopify OS 2.0 Liquid theme with zero build tooling: (1) a fully editor-configurable About Us page built from dedicated sections and a template file, and (2) a reusable drag-and-drop block toolkit (image+text, multi-column, testimonial/quote, collection grid) available on any page in the store. The recommended approach is entirely native Shopify — theme blocks in `blocks/`, sections in `sections/`, a JSON alternate page template `templates/page.about-us.json`, and schema settings for every piece of merchant-editable content. No npm packages, no JavaScript frameworks, no build pipeline changes are needed or acceptable.

The core architectural choice is to build all four new content components as theme blocks (`blocks/` directory, type `@theme`) rather than section-local blocks. This is the only way to achieve reusability across the About Us page, any future custom page, and the existing generic `custom-section.liquid` container. The existing `blocks/group.liquid` and `blocks/text.liquid` already establish this pattern in the codebase. The About Us page itself uses three purpose-built sections (origin story, mission/ethos, team) composed in a `page.about-us.json` template, each accepting `@theme` blocks for flexible content within a branded frame.

The dominant risks are editor-integration pitfalls: missing `{{ block.shopify_attributes }}` on block root elements (makes blocks invisible to the editor), missing presets in block schemas (makes blocks absent from the "Add block" picker), and Liquid code inside `{% javascript %}`/`{% stylesheet %}` tags (silently prevents dynamic settings from applying). These are all preventable with a consistent checklist applied to every block file before pushing. All four research areas have HIGH confidence grounded in official Shopify documentation and verified codebase patterns.

## Key Findings

### Recommended Stack

This is a pure Liquid stack with no deviations. Shopify provides all bundling, rendering, and deployment infrastructure. The only recommended tooling is the Shopify CLI for local development with live-reload. The milestone requires no new technology decisions — the stack is fixed by the platform.

**Core technologies:**
- Liquid (Shopify-managed): template rendering, schema definitions, block and section composition — the only server-side language available; no alternative exists
- `{% stylesheet %}` per component: structural, static CSS loaded only when the block renders; Shopify bundles and deduplicates automatically — no build step
- `{% style %}` per block/section: dynamic CSS custom properties derived from `section.settings.*` or `block.settings.*`; re-evaluates live when merchant changes editor settings
- `assets/critical.css`: shared utilities that must exist on every page load (`.btn`, `.product-card`, `.fade-up`, `.pagination`) — do not add block-specific styles here
- Vanilla JS via `{% javascript %}`: per-component behavior; Shopify bundles; no frameworks
- JSON templates (`templates/*.json`): OS 2.0 page composition; merchant-assignable in admin

**Critical version requirements:**
- `visible_if` conditional settings: stable release as of May 21, 2025 — safe to use
- Theme blocks and `content_for 'blocks'`: stable, already in use in this codebase
- All other features: standard OS 2.0, available on all Shopify stores

### Expected Features

**Must have (table stakes) — all marked P1:**
- About Us: brand origin story section (heading, richtext body, optional image, CTA)
- About Us: mission/ethos section (eyebrow, headline, supporting text)
- About Us: team/people section with repeating person blocks (photo, name, title, bio)
- Image + text block: left/right image toggle, adjustable image width, heading + body + optional CTA button
- Multi-column block: 2 or 3 columns, per-column optional icon/image + heading + body text
- Testimonial/quote block: quote text, author name, 1-5 star rating, optional reviewer photo, grid layout
- Collection grid block: collection picker, column count (2/3/4), product limit, view-all link toggle
- Collection page schema additions: `columns_desktop`, `columns_mobile`, `image_ratio`, `card_style` settings

**Should have (competitive differentiators) — P2:**
- Reviewer photo on testimonial block (credibility boost at low cost)
- Vertical alignment control on image+text block
- Secondary image on hover for collection page product cards (adds moderate JS complexity)

**Defer (v2+):**
- Testimonial carousel layout: high JS complexity, low value when grid of 3 is sufficient
- Video embed in brand story section: high impact storytelling but text-first launch is sufficient
- Press/media logos bar: only relevant once Ruck On has press coverage
- Full-width option on image+text block: easy add but not blocking launch

**Anti-features confirmed — do not build:**
- Hardcoded brand copy in any Liquid output
- Tab-switching About Us (hides content, adds JS, zero conversion benefit)
- Social media feed embeds (third-party scripts, API fragility)
- Animated counting numbers (existing `stats-bar.liquid` already serves this purpose)
- Carousel/slider for image+text rows (JS complexity with no advantage over left/right alternation)
- Too-granular block design (separate heading/body/button blocks per Shopify anti-pattern guidance)

### Architecture Approach

The architecture follows a strict layered model: layout wraps templates, templates compose sections, sections host theme blocks, blocks and sections use snippets for shared logic, and assets hold global utilities. The four new content blocks are entirely self-contained — each block owns its markup, CSS (in `{% stylesheet %}`), dynamic styles (in `{% style %}`), and any JS (in `{% javascript %}`). The About Us page is an alternate JSON template (`templates/page.about-us.json`) that references three dedicated sections, which merchants assign to their About Us page via the Shopify admin Pages UI.

**Major components and build order:**
1. `blocks/image-text.liquid` — reusable image+text block, no dependencies, builds first
2. `blocks/multi-column.liquid` — reusable column layout block, no dependencies
3. `blocks/quote.liquid` — testimonial/pull-quote block, no dependencies
4. `blocks/collection-grid.liquid` — product grid block; depends on existing `.product-card` styles in `critical.css`
5. `sections/about-origin.liquid` — branded origin story container; can use `@theme` blocks internally
6. `sections/about-mission.liquid` — branded mission container; same pattern
7. `sections/about-team.liquid` — team section container; hosts person blocks via `content_for 'blocks'`
8. `templates/page.about-us.json` — alternate template; depends on sections 5–7 existing
9. `locales/en.default.json` updates — depends on all new files having final key names
10. `sections/collection.liquid` schema additions — isolated; no dependencies on new blocks

**Key patterns enforced throughout:**
- Theme blocks (`@theme`) only — no mixing with section-local block definitions
- All block CSS namespaced with block name prefix (e.g., `.block-image-text__content`) to prevent cross-block style bleed in Shopify's bundled `block-styles.css`
- Every block has `{{ block.shopify_attributes }}` on its root element and at least one preset in schema
- CSS custom properties via `{% style %}` for all merchant-configurable values; `{% stylesheet %}` for structural rules only

### Critical Pitfalls

1. **Missing `{{ block.shopify_attributes }}` on block root element** — blocks render on storefront but are invisible to the theme editor (no click selection, no settings panel). Add this attribute as the first thing after the opening tag in every block file. Verify by clicking the block in the editor and confirming the blue selection border appears.

2. **Missing preset in block schema** — block file exists but does not appear in the editor "Add block" picker. No error is thrown; it silently doesn't appear. Every block schema must include `"presets": [{ "name": "..." }]`. Verify by opening the picker in the editor.

3. **Liquid code inside `{% javascript %}` or `{% stylesheet %}` tags** — styles or scripts silently fail; dynamic settings appear as literal Liquid strings in DevTools. Dynamic values (colors, spacing) must use `{% style %}` or inline `style=""` attributes with CSS custom properties, then referenced in `{% stylesheet %}`. Establish this pattern in the first block file.

4. **Mixing section-local block definitions with `@theme` theme blocks** — Shopify prohibits this combination; schema validation errors, blocks fail to render. All new sections use `"blocks": [{ "type": "@theme" }]` exclusively.

5. **Generic CSS class names causing cross-block style conflicts** — Shopify bundles all block stylesheets together; `.content`, `.image-wrap` etc. collide across block types. Use block-name-prefixed BEM classes (e.g., `.block-quote__author`) for every CSS rule inside block `{% stylesheet %}` tags.

6. **JS not re-initializing after editor section reload** — interactive elements work on page load but break whenever a merchant changes a setting. Wrap all JS init logic in a named function called from both initial load and `document.addEventListener('shopify:section:load', ...)`.

## Implications for Roadmap

Research points clearly toward a 3-phase delivery. The dependency graph from ARCHITECTURE.md drives the order: blocks must precede sections that host them, sections must precede the template, and collection page schema changes are entirely independent. The PITFALLS research adds a cross-cutting concern: the pattern-establishing pitfall mitigations (CSS namespacing, `shopify_attributes`, presets, `{% style %}` vs `{% stylesheet %}`) must be resolved in Phase 1 before subsequent blocks inherit incorrect patterns.

### Phase 1: Reusable Block Toolkit Foundation

**Rationale:** All four reusable blocks have zero external dependencies (only `critical.css` for the collection grid, which already exists). They are the foundation everything else builds on. Building them first means About Us sections can immediately reference and test them. Critically, this phase establishes the block CSS naming convention, the `{% style %}` pattern, and the editor integration checklist — patterns that all subsequent work inherits. Fixing pitfall patterns here prevents compound errors in Phases 2 and 3.

**Delivers:** Four drag-and-drop blocks usable on any page in the store; immediate merchant value for any page using the existing `custom-section.liquid` container.

**Addresses features:** Image+text block (left/right, width control, heading/body/CTA), multi-column block (2-3 cols, icon/image + heading + text), testimonial/quote block (quote + name + stars + optional photo), collection grid block (collection picker + columns + product limit + view-all link).

**Avoids:** Cross-block CSS conflicts (establish namespacing convention here), Liquid-in-stylesheet pitfall (establish `{% style %}` pattern here), missing `shopify_attributes` and presets (enforce checklist from first block).

### Phase 2: About Us Page

**Rationale:** Depends on Phase 1 blocks being available for sections to host. The three dedicated sections (origin, mission, team) use `content_for 'blocks'` and `"blocks": [{ "type": "@theme" }]` — they need the block toolkit to be meaningful. The alternate template ties them together. Locale keys are finalized here since section key names are determined.

**Delivers:** A fully merchant-configurable About Us page with branded sections, team photos, and block-powered flexible content — assignable to the About Us page via Shopify admin template picker.

**Addresses features:** Brand origin story section, mission/ethos section, team/people section with repeating person blocks (photo + name + title + bio), scroll-triggered fade-up animations (reuse existing `snippets/scroll-animations.liquid`), all images via editor with placeholder SVG fallbacks.

**Avoids:** Hardcoded text strings (all copy through schema settings + `| t` filter + locale keys), missing image placeholder fallbacks (team photos especially), theme block/section block mixing (about-team section uses `@theme` exclusively).

### Phase 3: Collection Page Controls

**Rationale:** Entirely independent from Phases 1 and 2 — it's an additive schema change to the existing `sections/collection.liquid`. No new files are created. It can ship in any order but is placed last because it is self-contained and carries the lowest dependency risk. The pitfall to watch here is backwards compatibility: new schema settings must have safe defaults that reproduce the current behavior for existing store configurations.

**Delivers:** Column count control (desktop 2-4, mobile 1-2), image ratio control (adapt/portrait/square), and card style control (standard/minimal) on the collection page — matching Dawn's baseline feature set that merchants expect.

**Addresses features:** `columns_desktop`, `columns_mobile`, `image_ratio`, `card_style` settings on `sections/collection.liquid`; schema defaults must reproduce current behavior.

**Avoids:** Breaking existing collection pagination, sort bar, and banner (new settings are purely additive); CLS issues from images without dimensions (verify `image_tag` filter usage after aspect-ratio CSS changes).

### Phase Ordering Rationale

- Blocks first because sections hosting `content_for 'blocks'` are untestable in the editor until blocks exist
- Pattern establishment (CSS namespacing, `{% style %}` vs `{% stylesheet %}`, checklist) must happen in Phase 1 because all subsequent blocks and sections inherit whatever convention is set first
- About Us template can only be created after its three sections exist (JSON template references section types by name)
- Collection page schema changes have no dependency on anything new — they are genuinely isolated and safe to sequence last or parallelize if two developers are available
- Locale updates (`locales/en.default.json`) are an output of each phase, not a separate phase — add keys alongside the Liquid files that use them

### Research Flags

Phases with standard, well-documented patterns — skip additional research:
- **Phase 1 (Block toolkit):** Shopify block architecture is fully documented with official examples. Existing `blocks/group.liquid` and `blocks/text.liquid` provide live in-codebase patterns. No research-phase needed.
- **Phase 2 (About Us page):** Alternate JSON templates and section architecture are well-documented. No novel patterns required.
- **Phase 3 (Collection controls):** Purely additive schema changes. `sections/collection.liquid` already exists as reference. No research-phase needed.

No phase requires a `/gsd:research-phase` during planning for this milestone. All technical patterns are established and verified.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Official Shopify docs + verified against existing codebase; zero ambiguity — platform is fixed |
| Features | MEDIUM | P1 features from official Shopify docs and Dawn source; P2/P3 differentiators partially from third-party analysis and competitor observation |
| Architecture | HIGH | Official Shopify docs + existing codebase verified; build order confirmed by dependency graph |
| Pitfalls | HIGH | All critical pitfalls sourced from official Shopify docs; community sources used only to confirm known issues |

**Overall confidence:** HIGH

### Gaps to Address

- **Team section person blocks vs. dedicated block type:** ARCHITECTURE.md suggests `sections/about-team.liquid` could use the generic `blocks/image-text.liquid` for person cards or a dedicated person block. A dedicated `blocks/person.liquid` (photo + name + title + bio) is more semantically correct and provides better merchant UX. Decision should be made before Phase 2 begins — confirm whether person block is a theme block or a section-local block inside `about-team.liquid`.

- **Collection page `card_style` CSS scope:** The `card_style` select setting will need CSS classes applied to the product card grid wrapper. FEATURES.md specifies `card` (border + shadow), `standard` (no border), `minimal` (image only) options. The exact CSS implementation against the existing `.product-card` structure in `critical.css` needs validation during implementation — the card markup structure must accommodate all three variants without DOM changes.

- **`visible_if` constraint on `collection` type:** PITFALLS.md notes that the `collection` setting type does not support `visible_if`. If the collection grid block needs to conditionally show/hide other settings based on collection presence, an alternative approach (showing/hiding via a `checkbox` intermediate) must be used. Low probability of impacting this milestone but worth confirming during schema design.

## Sources

### Primary (HIGH confidence)
- Shopify Developer Docs — Blocks: https://shopify.dev/docs/storefronts/themes/architecture/blocks
- Shopify Developer Docs — Theme Blocks: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks
- Shopify Developer Docs — Theme Block Schema: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/schema
- Shopify Developer Docs — JSON Templates: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates
- Shopify Developer Docs — Input Settings (including `visible_if`): https://shopify.dev/docs/storefronts/themes/architecture/settings/input-settings
- Shopify Developer Docs — Blocks Best Practices: https://shopify.dev/docs/storefronts/themes/architecture/blocks/best-practices
- Shopify Developer Docs — Sections and Blocks Best Practices: https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks
- Shopify Developer Docs — JavaScript and Stylesheet Tags: https://shopify.dev/docs/storefronts/themes/best-practices/javascript-and-stylesheet-tags
- Shopify Developer Docs — Editor Integration: https://shopify.dev/docs/storefronts/themes/best-practices/editor/integrate-sections-and-blocks
- Shopify Changelog — `visible_if` stable release (May 21, 2025): https://shopify.dev/changelog/conditional-settings-in-the-theme-editor
- Dawn source — main-collection-product-grid.liquid: https://github.com/Shopify/dawn/blob/main/sections/main-collection-product-grid.liquid
- Existing codebase — `blocks/group.liquid`, `blocks/text.liquid`, `sections/collection.liquid`: verified live patterns

### Secondary (MEDIUM confidence)
- Shopify official blog — How to Write an About Us Page: https://www.shopify.com/blog/how-to-write-an-about-us-page
- Shopify official blog — Meet the Team Page Examples: https://www.shopify.com/blog/meet-the-team-page-examples
- Shopify community — `shopify:section:load` event listener stacking: https://community.shopify.com/t/whats-up-with-shopifyload-stacking-event-listeners/572531

### Tertiary (LOW confidence)
- Posstack — Dawn image+text and multicolumn enhancements (third-party analysis, useful for gap identification only)
- Moxie Sozo — `@split` pattern (community blog; edge-case pattern not needed for this milestone)
- Tactical brand About Us analysis: Propper, Rothco, Vertx, FirstTactical websites (observation only)

---
*Research completed: 2026-03-06*
*Ready for roadmap: yes*
