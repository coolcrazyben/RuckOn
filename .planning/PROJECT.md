# Ruck On — Theme Enhancement Milestone

## What This Is

Ruck On is a custom Shopify theme for a tactical gear and apparel brand. The existing theme was built from scratch (not Dawn) with full dark/military aesthetic and all core pages. This milestone adds a brand storytelling About Us page and expands the theme editor with reusable drag-and-drop blocks so merchants can build and modify pages without touching code.

## Core Value

Merchants should be able to tell the Ruck On story and build new pages entirely through the Shopify theme editor — no code required.

## Requirements

### Validated

- ✓ Header with sticky nav, 3-col grid, mobile hamburger — existing
- ✓ Hero section (full-viewport, scroll animations) — existing
- ✓ Stats bar, best-sellers, manifesto, shop-by-category, community, newsletter sections — existing
- ✓ Footer with 4-col link menus — existing
- ✓ Product page (gallery, variant pills, qty, tabs, related) — existing
- ✓ Collection page (banner, sort bar, product grid, pagination) — existing
- ✓ Cart, search, blog, article, generic page, 404, password pages — existing

### Active

- [ ] About Us page: brand origin story section
- [ ] About Us page: mission/ethos section ("Carry the Weight" philosophy)
- [ ] About Us page: team/people section with photos
- [ ] All About Us sections use editor-uploadable images (no hardcoded assets)
- [ ] Reusable block: image + text side-by-side (image left or right, configurable)
- [ ] Reusable block: multi-column text (2–3 columns, good for values/features/specs)
- [ ] Reusable block: testimonials/quotes (customer reviews or pull-quote style)
- [ ] Reusable block: collection grid (drag onto any page, control columns + card style)
- [ ] Collection page: column count, image ratio, and card style configurable from editor

### Out of Scope

- Mobile app or headless storefront — web Shopify theme only
- Custom Liquid app blocks (only theme blocks + sections)
- Checkout customization — outside theme scope
- Customer account redesign — not requested

## Context

- Built on Shopify Skeleton starter, NOT Dawn — every section is custom
- All CSS in `{% stylesheet %}` tags per component; global utilities in `assets/critical.css`
- Shared `.product-card` styles are in `critical.css` (shared across collection, search, related products)
- Brand CSS variables in `snippets/css-variables.liquid`; scroll animations in `snippets/scroll-animations.liquid`
- Google Fonts (Black Ops One, Barlow Condensed, Barlow) loaded in `layout/theme.liquid`
- Schema settings use `{% style %}` for live-preview in theme editor; structural CSS uses `{% stylesheet %}`
- All user-facing text must use `{{ 'key' | t }}` translation filter with keys in `locales/en.default.json`
- About Us images will be placeholder-based — uploaded via editor (`image_picker` settings)

## Constraints

- **Tech stack**: Liquid, CSS, vanilla JS — no build tools, no npm
- **Shopify schema**: All sections/blocks must have complete `{% schema %}` so everything is editor-configurable
- **Brand consistency**: Must match existing dark tactical aesthetic (colors, fonts, clip-path buttons, fade-up animations)
- **Compatibility**: Existing sections and templates must not be broken by new blocks
- **Performance**: Stylesheet/javascript tags per component; only critical styles in `critical.css`

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build on Skeleton not Dawn | Project history — theme was built custom | — Pending |
| About Us as `/pages/about-us` Shopify page | Standard Shopify page template, editor-assignable | — Pending |
| New reusable blocks (not sections) | Blocks nest inside sections, giving merchants full flexibility | — Pending |

---
*Last updated: 2026-03-06 after initialization*
