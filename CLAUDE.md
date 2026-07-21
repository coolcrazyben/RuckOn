# Ruck On — Brand Guide

## Brand Identity
- **Name:** Ruck On
- **Tagline:** "Carry the Weight"
- **Description:** Tactical gear and apparel for those who never put the ruck down.
- **Vibe:** Dark, tactical, military-inspired but modern and premium. Rugged meets clean design.

## Color Palette
```
--color-olive:       #4a5c2a  (primary brand green)
--color-olive-dark:  #2d3a18  (dark olive, backgrounds)
--color-olive-light: #6b7f3e  (light olive, accents)
--color-tan:         #c8a96e  (accent, highlights, logo)
--color-tan-light:   #e8d5b0  (light tan, body text)
--color-cream:       #f5f0e8  (very light, occasional bg)
--color-black:       #111009  (main background)
--color-charcoal:    #1e1e1a  (card/section backgrounds)
--color-white:       #faf8f4  (primary text color)
--color-orange:      #d4622a  (CTAs, eyebrow text, badges)
```

## Typography
- **Black Ops One** — headings, logo, product names, stat numbers
- **Barlow Condensed** — eyebrow text, nav links, labels, uppercase text
- **Barlow** — body text, descriptions

## Design Tokens
- Buttons use `clip-path: polygon(0 0, 95% 0, 100% 100%, 5% 100%)` for tactical skew shape
- All uppercase text: `letter-spacing: 2–5px` depending on size
- Scroll-triggered `fade-up` animations on cards and stats

## Header Layout
3-column CSS grid (`1fr auto 1fr`) so the logo is always truly centered:

| Zone | Desktop (≥1025px) | Mobile (≤1024px) |
|------|-------------------|------------------|
| Left `.site-header__left` | Nav links | Hamburger + Search icon |
| Center `.site-header__logo` | Logo | Logo |
| Right `.site-header__right` | Search + Account + Cart icons | Account + Cart icons |

- Cart is an **icon link** (shopping bag SVG) with an orange count badge — not a styled button
- Desktop search icon: `.site-header__search-desktop`; mobile (left zone) search: `.site-header__search-link` (each hidden on the other breakpoint via CSS)
- Mobile slide-out drawer is triggered by the hamburger button

## Product Page Architecture
The product page is **block-based**. `sections/product.liquid` renders the gallery (left
column) and `{% content_for 'blocks' %}` for the info column (right); it also keeps the
related-products and patch cross-sell rows, plus the shared product `{% stylesheet %}` /
`{% javascript %}` (keyed off stable ids/classes like `#product-form`, `#product-price`,
`.product-pill`, `.product-tabs__btn`).

Each info-column piece is a theme block in `blocks/product-*.liquid`: `product-eyebrow`,
`product-title`, `product-price`, `product-variant-picker`, `product-community-selector`,
`product-patch-text`, `product-community-name`, `product-quantity`, `product-buy-buttons`,
`product-tabs`. The section schema allows `@theme` + `@app` blocks, so merchants and apps
can add/remove/reorder content anywhere in the info column.

**Form rule:** `product-buy-buttons` owns the `<form id="product-form">`. All other blocks
with form inputs (quantity, community selector, patch text, community name) associate via
the HTML `form="product-form"` attribute — never nest a second `<form>`. The community
selector and patch-text blocks self-guard by product tag, so they're safe on the shared
default template.

## Theme Editor Rules
Every section/block must expose all text, colors, images, links, and visibility through
`{% schema %}` settings so merchants can edit from the Customize editor without touching
code. No hardcoded brand copy in Liquid output — use `section.settings.*` / `block.settings.*`.

---

# Shopify Theme Notes

🚨 **MANDATORY: call `learn_shopify_api` once before working with Liquid themes** (if the
tool is available in the session). For any Liquid filter/tag/object reference, use that
tool or the Shopify docs rather than relying on memory.

## Directory structure
- `assets` — static files (CSS/JS/images/fonts). Keep only `critical.css` + globally-needed
  assets here; prefer `{% stylesheet %}` / `{% javascript %}` tags in components.
- `blocks` — reusable, nestable components with `{% schema %}` (validate vs `schemas/theme_block.json`).
- `config` — `settings_schema.json` (global settings shape) + `settings_data.json` (values).
- `layout` — top-level HTML wrappers; must include `{{ content_for_header }}` and `{{ content_for_layout }}`.
- `locales` — translation JSON (`en.default.json` required); use `{{ 'key' | t }}`.
- `sections` — full-width page modules with `{% schema %}` (validate vs `schemas/section.json`).
- `snippets` — reusable fragments rendered via `{% render %}`.
- `templates` — JSON files mapping sections/blocks to each page type (theme-editor data).

## Component rules
- CSS/JS live in `{% stylesheet %}` / `{% javascript %}` tags (only in `snippets/`,
  `blocks/`, `sections/`). Liquid is NOT rendered inside these tags.
- Snippets, and statically-rendered blocks (`{% content_for 'block', ... %}`), must start
  with a `{% doc %}` header documenting purpose + `@param`s.

## Schema good practices
- **Single CSS property** → drive it with a CSS variable, e.g. `style="--gap: {{ block.settings.gap }}px"`.
- **Multiple CSS properties** → use a `select` whose values are CSS class names.
- **Mobile columns** → a `select` with options `1` / `2` (id `columns_mobile`).
- Parentheses and ternaries are NOT supported in Liquid — use nested `{% if %}`.

## Translations
- All user-facing text uses `{{ 'key' | t }}`; add new keys to `locales/en.default.json` only.
- Hierarchical snake_case keys, max 3 levels; interpolate variables; sentence case.
- Note: the existing product section/blocks use plain-English schema labels (not `t:` keys)
  for consistency — match the surrounding file's convention when editing.
