# Stack Research

**Domain:** Shopify Liquid theme — reusable blocks and rich page sections
**Researched:** 2026-03-06
**Confidence:** HIGH (Shopify official docs + codebase verification)

---

## Recommended Stack

This is a Liquid-only stack. No npm, no build tools. The project constraint is enforced and correct — Shopify's own skeleton starter and modern themes like Dawn use vanilla Liquid, vanilla CSS (per-component via `{% stylesheet %}`), and vanilla JS. Any deviation from this introduces deployment complexity with zero upside for a Shopify theme.

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Liquid (Shopify) | Current (Shopify-managed) | Template rendering, schema, Liquid objects | The only server-side language available in Shopify themes — no alternative |
| CSS via `{% stylesheet %}` | N/A | Per-component scoped styles, bundled by Shopify | Shopify bundles these automatically; no build step needed. Use this for structural CSS in blocks and sections |
| CSS via `{% style %}` | N/A | Dynamic editor-reactive styles mapped from schema settings | `{% style %}` re-evaluates when theme editor changes a setting — required for live preview. Use only for CSS custom properties derived from `section.settings.*` or `block.settings.*` |
| CSS via `assets/critical.css` | N/A | Shared utilities loaded on every page | Only put styles here that must exist on page load and are reused across multiple sections. Do NOT put block-specific styles here |
| Vanilla JS via `{% javascript %}` | ES2020+ | Per-component behavior | Shopify bundles these. No frameworks — the project has zero framework dependency and should stay that way |
| JSON templates (`templates/*.json`) | Shopify OS 2.0 | Page composition without code | Modern Shopify page structure. Assign to pages in admin. `page.about.json` is the correct file for an About Us page template |

### Theme Block System (the core pattern for this milestone)

The milestone requires reusable blocks. Here is the definitive pattern:

**Theme blocks** (`blocks/*.liquid`) vs **section blocks** (defined inside a section's `{% schema %}`) are mutually exclusive. You must choose one per section. Because the goal is reusability across the About Us template AND any future pages, use theme blocks exclusively.

| Concept | Pattern | Why |
|---------|---------|-----|
| Theme blocks | `blocks/image-with-text.liquid`, `blocks/multi-column.liquid`, etc. | Reusable across any section that opts in with `"blocks": [{ "type": "@theme" }]` |
| Section blocks | Defined inline in the section `{% schema %}` | Section-local only — cannot be reused. Do NOT use for this milestone |
| Container section | A generic `sections/page-builder.liquid` with `{% content_for 'blocks' %}` | The host that accepts any theme block. One section, unlimited flexibility |
| Nested blocks | A block with `"blocks": [{ "type": "@theme" }]` and `{% content_for 'blocks' %}` | Allows blocks inside blocks — the `group.liquid` pattern already in this codebase |

### Schema Setting Types — What to Use for Each Block

| Setting Type | Use Case | Returns | Notes |
|--------------|----------|---------|-------|
| `image_picker` | Hero image, left/right image, team photo | image object | Supports focal point, alt text. Use `image_url` + `image_tag` filters |
| `select` | Image position (left/right), column count, card style, image ratio | string | Renders as SegmentedControl (2-5 options) or Dropdown (6+). Use CSS class values directly |
| `range` | Padding, gap, font size | number | Requires `min`, `max`, `default`. Use as CSS variable |
| `inline_richtext` | Headings, short labels | string (HTML) | Supports bold, italic, link. No line breaks. Better than `text` for headings |
| `richtext` | Body copy, long descriptions | string (HTML) | Full paragraph/list support. Use with `.rte` class for styled output |
| `text` | Labels, bylines, short strings | string | Plain text — no formatting |
| `url` | CTA links | string | Includes page/collection/product picker |
| `color` | Override colors per-block | color object | Only needed if blocks need per-instance color overrides. Default to CSS variables |
| `checkbox` | Toggle features (e.g., show image, reverse layout) | boolean | Use with `visible_if` to hide dependent settings |
| `collection` | Collection grid block | collection object | Returns live collection. No default value support |
| `text_alignment` | Content alignment | string (left/center/right) | Renders as icon SegmentedControl — better UX than a select |
| `visible_if` | Conditional setting visibility | n/a | Released May 2025. Stable. Syntax: `"visible_if": "{{ block.settings.show_image }}"`. Use to hide image-specific settings when no image is shown |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Shopify CLI | Local dev server, theme sync, push/pull | `shopify theme dev` for live-reload local dev against a dev store. Required for efficient iteration |
| Shopify Theme Editor | Visual validation of schema settings | Test every block immediately after writing schema. Settings that aren't visible in the editor are broken |
| GitHub integration | Theme version control | Already in use (main branch syncs to Shopify) |

---

## Installation

No npm packages. No dependencies to install. All tools are Shopify CLI:

```bash
# Shopify CLI (if not already installed)
npm install -g @shopify/cli @shopify/theme

# Start local dev server
shopify theme dev --store=your-store.myshopify.com

# Push theme to store
shopify theme push
```

---

## Key Patterns — Concrete Implementation

### Pattern 1: Container Section for the About Us Template

The About Us page needs a generic container section that accepts any theme block. This is the "page builder" pattern.

```liquid
{{! sections/about-us.liquid (or use a generic page-builder.liquid) }}
<section class="about-page" {{ section.shopify_attributes }}>
  <div class="page-width">
    {% content_for 'blocks' %}
  </div>
</section>

{% schema %}
{
  "name": "About Us",
  "blocks": [{ "type": "@theme" }, { "type": "@app" }],
  "presets": [{ "name": "About Us" }]
}
{% endschema %}
```

Then `templates/page.about.json` references this section:

```json
{
  "sections": {
    "main": {
      "type": "about-us",
      "settings": {}
    }
  },
  "order": ["main"]
}
```

Merchants assign this template to the About Us page in Shopify admin (Pages > About Us > Theme template > about-us).

### Pattern 2: Image + Text Block

Use a theme block with `image_picker`, `inline_richtext` for heading, `richtext` for body, `select` for layout direction, and `visible_if` for the image-specific alt text field.

```liquid
{{! blocks/image-with-text.liquid }}
{% doc %}
  Renders a two-column image + text layout, image on left or right.

  @example
  {% content_for 'block', type: 'image-with-text', id: 'about-story' %}
{% enddoc %}

<div
  class="image-with-text {{ block.settings.layout }}"
  style="--img-width: {{ block.settings.image_width }}%"
  {{ block.shopify_attributes }}
>
  <div class="image-with-text__media">
    {%- if block.settings.image -%}
      {{ block.settings.image | image_url: width: 1200 | image_tag: loading: 'lazy' }}
    {%- else -%}
      {{ 'image' | placeholder_svg_tag: 'image-with-text__placeholder' }}
    {%- endif -%}
  </div>
  <div class="image-with-text__content">
    {%- if block.settings.eyebrow != blank -%}
      <span class="eyebrow">{{ block.settings.eyebrow }}</span>
    {%- endif -%}
    {%- if block.settings.heading != blank -%}
      <h2 class="image-with-text__heading">{{ block.settings.heading }}</h2>
    {%- endif -%}
    {%- if block.settings.body != blank -%}
      <div class="rte image-with-text__body">{{ block.settings.body }}</div>
    {%- endif -%}
    {%- if block.settings.cta_label != blank and block.settings.cta_url != blank -%}
      <a href="{{ block.settings.cta_url }}" class="btn btn--primary">
        {{ block.settings.cta_label }}
      </a>
    {%- endif -%}
  </div>
</div>

{% schema %}
{
  "name": "Image with text",
  "settings": [
    {
      "type": "select",
      "id": "layout",
      "label": "Image position",
      "options": [
        { "value": "image-with-text--image-left", "label": "Image left" },
        { "value": "image-with-text--image-right", "label": "Image right" }
      ],
      "default": "image-with-text--image-left"
    },
    {
      "type": "range",
      "id": "image_width",
      "label": "Image width %",
      "min": 30,
      "max": 70,
      "step": 5,
      "default": 50
    },
    {
      "type": "image_picker",
      "id": "image",
      "label": "Image"
    },
    {
      "type": "text",
      "id": "eyebrow",
      "label": "Eyebrow text",
      "placeholder": "Our Story"
    },
    {
      "type": "inline_richtext",
      "id": "heading",
      "label": "Heading",
      "default": "Built for Those Who Never Stop"
    },
    {
      "type": "richtext",
      "id": "body",
      "label": "Body text"
    },
    {
      "type": "text",
      "id": "cta_label",
      "label": "Button label"
    },
    {
      "type": "url",
      "id": "cta_url",
      "label": "Button link",
      "visible_if": "{{ block.settings.cta_label != blank }}"
    }
  ],
  "presets": [{ "name": "Image with text" }]
}
{% endschema %}
```

### Pattern 3: Multi-Column Text Block

Use section-level `select` for column count, render columns as nested blocks OR as a direct multi-column layout with block settings. The nested block approach (group block + child text blocks) is more flexible but complex. For 2-3 fixed columns with identical structure, a simpler flat design using `forloop.index` modulo logic is better.

**Decision: Use the flat block approach.** Each `multi-column` block IS the column group. Settings control the number of columns (2 or 3). Child content uses `{% content_for 'blocks' %}` to nest text/image sub-blocks inside each column.

This uses the existing `group.liquid` + `text.liquid` nesting pattern already established in this codebase.

### Pattern 4: Testimonial / Quote Block

Single block, no nesting needed. Settings: `richtext` for the quote, `text` for author name, `text` for role/company, `image_picker` for optional avatar.

```json
{
  "name": "Testimonial",
  "settings": [
    { "type": "richtext", "id": "quote", "label": "Quote" },
    { "type": "text", "id": "author", "label": "Author name" },
    { "type": "text", "id": "author_title", "label": "Role or location" },
    { "type": "image_picker", "id": "avatar", "label": "Author photo (optional)" }
  ],
  "presets": [{ "name": "Testimonial" }]
}
```

### Pattern 5: Collection Grid Block

A theme block that renders a product grid from a `collection` setting. This is a standalone block used on non-collection pages (like About Us, or a landing page) to surface products.

```json
{
  "name": "Collection grid",
  "settings": [
    { "type": "collection", "id": "collection", "label": "Collection" },
    {
      "type": "select",
      "id": "columns",
      "label": "Columns",
      "options": [
        { "value": "collection-grid--2", "label": "2" },
        { "value": "collection-grid--3", "label": "3" },
        { "value": "collection-grid--4", "label": "4" }
      ],
      "default": "collection-grid--3"
    },
    {
      "type": "select",
      "id": "image_ratio",
      "label": "Image ratio",
      "options": [
        { "value": "ratio--square", "label": "Square" },
        { "value": "ratio--portrait", "label": "Portrait (3:4)" },
        { "value": "ratio--landscape", "label": "Landscape (4:3)" }
      ],
      "default": "ratio--portrait"
    },
    {
      "type": "range",
      "id": "products_to_show",
      "label": "Products to show",
      "min": 2,
      "max": 12,
      "step": 2,
      "default": 6
    }
  ],
  "presets": [{ "name": "Collection grid" }]
}
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Theme blocks in `blocks/` directory | Section blocks defined in section schema | Only when the block will NEVER be reused in another section. Appropriate for highly specific blocks like a product-page-only feature block |
| `{% stylesheet %}` per block | Single global CSS file in `assets/` | Only for truly global utilities (resets, grid, `.btn`, `.product-card`). Never for block-specific layout |
| `inline_richtext` for headings | Plain `text` for headings | Use `text` only for labels, badges, short strings where formatting is never needed |
| `content_for 'blocks'` for nested layout | Capturing block output and splitting on `<!--@split-->` | The `@split` pattern exists for edge cases where you need to wrap each block output differently (e.g., a slider where each block becomes a slide). Too clever for this milestone — avoid |
| `page.about.json` template | Adding About Us to `page.json` | `page.json` is the default — modifying it would affect all pages. Always create a named alternate template for custom pages |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Section blocks (inline in `{% schema %}`) for the new blocks | Cannot be reused across sections; locked to one parent. The `image-with-text` block would be stuck in `about-us.liquid` and unavailable elsewhere | Theme blocks in `blocks/` directory |
| Mixing section blocks and theme blocks in one section | Shopify explicitly prohibits this — it causes schema validation errors and breaks the editor | Choose one: `"blocks": [{ "type": "@theme" }]` for theme blocks, OR local block type definitions, never both |
| Hardcoded brand copy in block Liquid output | Violates the project constraint and breaks i18n. The theme editor cannot override hardcoded strings | All user-visible text through `section.settings.*`, `block.settings.*`, or `{{ 't:key' | t }}` |
| CSS in `assets/critical.css` for block-specific styles | Critical CSS is loaded on every page. Block CSS should only load when the block is used | `{% stylesheet %}` inside the block file — Shopify only loads it when the block renders |
| Static section rendering (`{% section 'name' %}`) for About Us content | Static sections cannot be reordered, added, or removed by merchants in the editor — defeats the purpose of a flexible About Us template | JSON template + dynamic section rendering |
| JavaScript frameworks (React, Alpine.js, Vue) | No build step available, adds KB with no benefit for simple DOM interactions | Vanilla JS in `{% javascript %}` tag |

---

## Stack Patterns by Variant

**If a block needs different visual layouts (image left / image right):**
- Use a `select` setting whose values ARE the CSS class names (e.g., `"value": "image-with-text--image-left"`)
- Apply the class directly: `<div class="{{ block.settings.layout }}">`
- Write both layout variants in `{% stylesheet %}`
- Do NOT use `{% if %}` blocks to swap HTML structure — it duplicates markup and creates maintenance burden

**If a block has a setting that only makes sense when another setting is active:**
- Use `visible_if` (stable as of May 2025): `"visible_if": "{{ block.settings.show_image }}"`
- Example: hide `image_width` range when no image is set
- This is purely a UX improvement in the editor — the Liquid output should still handle both states gracefully with `{%- if block.settings.image -%}`

**If a block needs to show a collection's products:**
- Read `block.settings.collection.products` directly in Liquid
- Limit with `| limit: block.settings.products_to_show`
- Use the shared `.product-card` classes from `critical.css` — this is what they're there for

**If the merchant needs to control image ratio on the collection page:**
- Add `image_ratio` select to the `collection.liquid` section schema
- Pass the ratio value as a CSS class on the product grid wrapper
- CSS: `.collection-grid.ratio--square .product-card__media { aspect-ratio: 1 / 1; }`
- This follows the existing `columns_desktop`/`columns_mobile` pattern already in that section

---

## Version Compatibility

| Feature | Availability | Notes |
|---------|-------------|-------|
| Theme blocks (`blocks/` directory) | Shopify OS 2.0+ (all stores) | Stable. Not a beta feature |
| `content_for 'blocks'` | Shopify OS 2.0+ | Stable. Used in this codebase already |
| `content_for 'block', type: ..., id: ...` (static blocks) | Shopify OS 2.0+ | For embedding a specific block statically. Requires `{% doc %}` in the block file |
| `visible_if` conditional settings | Released May 21, 2025 | Stable release per Shopify changelog. Safe to use |
| Nested blocks (`"blocks": [{ "type": "@theme" }]` in a block schema) | Shopify OS 2.0+ | Stable. The existing `group.liquid` block already uses this |
| `@app` block type in schema | Shopify OS 2.0+ | Allows Shopify apps to inject blocks into sections. Include alongside `@theme` as a courtesy |
| `enabled_on`/`disabled_on` in section schema | Shopify OS 2.0+ | Use to restrict sections to specific template types (e.g., only on page templates) |

---

## Sources

- Shopify Developer Docs — Blocks: https://shopify.dev/docs/storefronts/themes/architecture/blocks (HIGH confidence — official)
- Shopify Developer Docs — Theme Blocks: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks (HIGH confidence — official)
- Shopify Developer Docs — Theme Block Schema: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/schema (HIGH confidence — official)
- Shopify Developer Docs — JSON Templates: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates (HIGH confidence — official)
- Shopify Developer Docs — Input Settings: https://shopify.dev/docs/storefronts/themes/architecture/settings/input-settings (HIGH confidence — official)
- Shopify Changelog — Conditional Settings (visible_if): https://shopify.dev/changelog/conditional-settings-in-the-theme-editor (HIGH confidence — official changelog, released May 21, 2025)
- Shopify Developer Docs — Sections Best Practices: https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks (HIGH confidence — official)
- Dawn Theme — image-with-text.liquid (schema reference): https://github.com/Shopify/dawn (MEDIUM confidence — reference implementation, not prescriptive)
- Dawn Theme — multicolumn.liquid (schema reference): https://raw.githubusercontent.com/Shopify/dawn/main/sections/multicolumn.liquid (MEDIUM confidence — reference implementation)
- Moxie Sozo — @split pattern: https://moxiesozo.com/ideas/building-shopify-themes-with-theme-blocks-and-using-the-split-pattern (LOW confidence — community blog, single source, describes an edge-case pattern not needed for this milestone)
- Existing codebase: `blocks/group.liquid`, `blocks/text.liquid`, `sections/collection.liquid` — verified patterns in use (HIGH confidence — source of truth for this project)

---

*Stack research for: Ruck On — Theme Enhancement Milestone (Shopify Liquid blocks and sections)*
*Researched: 2026-03-06*
