# Ruck On — Brand Guide

## Brand Identity
- **Name:** Ruck On
- **Tagline:** "Carry the Weight"
- **Description:** Gear for the long carry — made for people who never put the ruck down.
- **Vibe:** Editorial running-heritage. Paper-white grounds, deep navy ink, antique
  gold accents, big photography and a display serif. Restrained, premium, magazine-like.
  Modelled on tracksmith.com.

## Color Palette
```
--color-navy:        #0a1e32  (primary ink, dark panels, solid buttons)
--color-navy-deep:   #061524  (deepest navy)
--color-navy-mid:    #2c4763  (mid navy)
--color-navy-60:     #5c6469  (secondary body copy, meta)
--color-gold:        #9a825c  (accent, dividers)
--color-gold-dark:   #857151  (eyebrows, stroke links, solid gold buttons)
--color-gold-light:  #c3ac83
--color-paper:       #ffffff  (main background)
--color-paper-warm:  #f4f0e9  (cream — pull quotes, newsletter)
--color-paper-cool:  #f8f8f9  (product-card media tile)
--color-rule:        #e7e9eb  (hairline borders — the only border colour)
```

Legacy aliases (`--color-olive`, `--color-tan`, `--color-black`, `--color-white`,
`--color-orange`, …) still exist in `snippets/css-variables.liquid` and now resolve to
the palette above, so older section CSS keeps working. **Prefer the new token names in
new code**; treat the aliases as a compatibility layer, not an API.

## Typography
- **Playfair Display** (`--font-heading`) — headlines, product/collection titles,
  prices, stat numbers, the wordmark. Always `font-weight: 400`, tight leading,
  slightly negative tracking, Title Case (never uppercase except the wordmark).
- **Archivo** (`--font-label` and `--font-body`) — nav, eyebrows, buttons, labels,
  body copy. Loaded as a variable font; label/uppercase text sets
  `font-stretch: var(--label-stretch)` (112%) to read as an extended grotesque.

### The house italic
Headlines pair a roman phrase with an italic accent — "The Ruck *is Sacred*",
"Join Our *Newsletter*", "Better *Together*". Sections express this as a second
settings field rendered in a `<span>` (or `<em>` where the section has one text field);
the accent is **italic in the same ink**, never a colour change.

## Design Tokens
- Buttons are rectangular — no radius, no shadow, no skew. ~11px uppercase,
  `letter-spacing: 0.18em`, `min-width: 9rem`, hover `opacity: 0.8`.
  Variants: `.btn--primary` (solid gold-dark), `.btn--navy` (solid navy — use for
  commerce actions: add to cart, checkout), `.btn--secondary` (navy outline on paper),
  `.btn--white` / `.btn--outline-light` (over photography).
- `.link-stroke` is the house text CTA: uppercase micro-label with a 40%-opacity
  hairline underline that solidifies on hover. Variants `--navy`, `--light`.
- Borders are hairlines in `--color-rule`. Do not tint borders gold.
- All uppercase text: `letter-spacing: 0.16em` (0.18–0.24em for buttons and eyebrows).
- Scroll-triggered `fade-up` animations on cards and stats.
- Product cards are centred and borderless: media on `--color-paper-cool`, sans title,
  **serif price**, hover `scale(1.035)`.

## Header Layout
3-column CSS grid (`1fr auto 1fr`) so the logo is always truly centered:

| Zone | Desktop (≥1025px) | Mobile (≤1024px) |
|------|-------------------|------------------|
| Left `.site-header__left` | Nav links | Hamburger |
| Center `.site-header__logo` | Logo | Logo |
| Right `.site-header__right` | Account + Cart icons | Account + Cart icons |

- White bar, hairline bottom rule, no blur or gradient.
- `nav_alignment` (schema) defaults to `left` — all links sit left of the centred
  wordmark, as on the reference site. Set it to `split` to balance them either side.
- Cart is an **icon link** (shopping bag SVG); the count is a small plain gold numeral,
  not a filled pip.
- Mobile slide-out drawer is triggered by the hamburger button.

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
