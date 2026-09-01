# Ruck On — Brand Guide

## Brand Identity
- **Name:** Ruck On
- **Tagline:** "Carry the Weight"
- **Description:** Gear for the long carry — made for people who never put the ruck down.
- **Vibe:** Field kit on paper. Olive green and worn tan on light grounds, big
  photography, condensed athletic caps. Editorial layout, rugged voice.

## Color Palette
```
--color-green:       #4a5c2a  (PRIMARY — buttons, links, accents)
--color-green-deep:  #1f290f
--color-green-light: #6b7f3e
--color-ink:         #2d3a18  (dark olive — headings and body ink)
--color-ink-60:      #5f6653  (secondary copy, meta)
--color-tan:         #7a6634  (eyebrows, stroke links, headline accents)
--color-tan-light:   #c8a96e  (accents over photography only — 2.2:1 on white)
--color-paper:       #ffffff  (main background)
--color-paper-warm:  #f2f1ea  (pull quotes, newsletter)
--color-paper-cool:  #f6f6f3  (product-card media tile)
--color-rule:        #e3e3dc  (hairline borders — the only border colour)
```

**Orange is retired.** The old `--color-orange` alias resolves to green so legacy
section CSS keeps working; do not reintroduce it.

Legacy aliases (`--color-olive`, `--color-black`, `--color-white`, `--color-charcoal`,
`--color-cream`, …) live in `snippets/css-variables.liquid` and resolve to the palette
above. **Prefer the real token names in new code**; the aliases are a compatibility
layer, not an API.

Contrast: green 7.35:1, ink 12.1:1, ink-60 6.0:1, tan 5.6:1 — all on white, all AA.
`--color-tan-light` fails on paper; use it only over photography.

## Typography
- **Oswald** (`--font-heading` and `--font-label`) — headlines, wordmark, eyebrows,
  nav, buttons, labels, prices, stat numbers. Display headings are **uppercase,
  weight 600, `letter-spacing: 0.01em`**. Labels are ~11px at `0.16em`.
- **Inter** (`--font-body`) — body copy, product-card titles, form fields.
- `--label-stretch` is `normal` (Oswald has no width axis); the variable is kept so
  existing label rules stay valid.

### The house accent
Headlines pair a phrase with an accent word shifted to **tan** — "THE RUCK / IS SACRED",
"BETTER / TOGETHER". Sections express this as a second settings field rendered in a
`<span>` (or `<em>` where the section has one text field). Oswald has no italic, so the
accent is always a colour shift, never a style change.

**Do not append an accent colour to a rule that already sets a settings-driven colour** —
it silently overrides the merchant's choice. Drive the accent from the setting instead.

## Design Tokens
- Buttons are rectangular — no radius, shadow or skew. ~11px uppercase,
  `letter-spacing: 0.18em`, `min-width: 9rem`, hover `opacity: 0.8`.
  Variants: `.btn--primary` (solid green — the default CTA everywhere),
  `.btn--secondary` (ink outline, fills on hover), `.btn--white` /
  `.btn--outline-light` (over photography).
- `.link-stroke` is the house text CTA: uppercase micro-label with a hairline
  underline that solidifies on hover. Variants `--navy` (ink), `--light`.
- Borders are hairlines in `--color-rule`. Do not tint borders tan.
- Scroll-triggered `fade-up` animations on cards and stats.
- Product cards are centred and borderless: media on `--color-paper-cool`,
  Inter title, Oswald price, hover `scale(1.035)`.

## Header Layout
3-column CSS grid (`1fr auto 1fr`) so the logo is always truly centered:

| Zone | Desktop (≥1025px) | Mobile (≤1024px) |
|------|-------------------|------------------|
| Left `.site-header__left` | First half of nav | Hamburger |
| Center `.site-header__logo` | Logo | Logo |
| Right `.site-header__right` | Second half of nav + Account + Cart | Account + Cart |

- White bar, hairline bottom rule, no blur or gradient.
- `nav_alignment` (schema) is set to **`split`** — links divide evenly either side of
  the wordmark, odd counts putting the extra on the left. Set it to `left` to stack
  them all before the logo.
- Cart is an **icon link** (shopping bag SVG); the count is a small plain tan numeral,
  not a filled pip.
- Mobile slide-out drawer is triggered by the hamburger button.

## Known Gaps
- Nav is a flat link list; there is no mega-menu.
- The stored logo image was drawn for the old dark header and is not wired up —
  the header falls back to the Oswald text wordmark.

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
