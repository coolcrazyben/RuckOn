# Architecture Research

**Domain:** Shopify Liquid theme — About Us page and reusable block toolkit
**Researched:** 2026-03-06
**Confidence:** HIGH — all patterns verified against official Shopify documentation and existing codebase

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     LAYOUT LAYER                            │
│  layout/theme.liquid — HTML wrapper, fonts, head, groups   │
├─────────────────────────────────────────────────────────────┤
│                   TEMPLATE LAYER (JSON)                     │
│  templates/page.about-us.json — section IDs + order array  │
├──────────┬──────────┬───────────────────────────────────────┤
│ SECTION  │ SECTION  │  SECTION  (full-width page slots)    │
│  story   │ mission  │  team     <-- each in sections/       │
│ (Liquid) │ (Liquid) │ (Liquid)                              │
├──────────┴──────────┴───────────────────────────────────────┤
│                    BLOCK LAYER (reusable)                   │
│  blocks/image-text.liquid   blocks/multi-column.liquid      │
│  blocks/quote.liquid        blocks/collection-grid.liquid   │
│  blocks/group.liquid (exists)  blocks/text.liquid (exists)  │
├─────────────────────────────────────────────────────────────┤
│                   SNIPPET LAYER (no editor UI)              │
│  snippets/css-variables.liquid  snippets/scroll-animations  │
│  snippets/meta-tags.liquid      snippets/image.liquid       │
├─────────────────────────────────────────────────────────────┤
│                    ASSET LAYER                              │
│  assets/critical.css — shared .product-card, .btn, .fade-up│
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| `layout/theme.liquid` | HTML doc wrapper, font loading, global snippets | Renders header-group, main, footer-group |
| `templates/page.about-us.json` | Defines which sections appear on About Us and in what order | JSON file with `sections` + `order` arrays |
| `sections/about-origin.liquid` | Brand origin story section — full-width, self-contained | `{% style %}` + markup + `{% schema %}` |
| `sections/about-mission.liquid` | Mission/ethos section — philosophy content | Same pattern as above |
| `sections/about-team.liquid` | Team/people section — repeating person cards via blocks | Section hosts blocks via `{% content_for 'blocks' %}` |
| `blocks/image-text.liquid` | Reusable image + text side-by-side, any section | `blocks/` folder — `@theme` type, usable anywhere |
| `blocks/multi-column.liquid` | 2–3 column text layout for values/features/specs | `blocks/` folder — `@theme` type |
| `blocks/quote.liquid` | Customer testimonial or pull-quote display | `blocks/` folder — `@theme` type |
| `blocks/collection-grid.liquid` | Drag-onto-any-page product grid, configurable columns | `blocks/` folder — `@theme` type, uses collection setting |
| `blocks/group.liquid` | Layout wrapper for horizontal/vertical block nesting (exists) | Already in codebase, accepts `@theme` blocks |
| `blocks/text.liquid` | Generic styled text block (exists) | Already in codebase |
| `snippets/css-variables.liquid` | All brand CSS custom properties at `:root` | Rendered in `<head>` by layout |
| `assets/critical.css` | Global utilities loaded on every page | `.product-card`, `.btn`, `.fade-up`, `.pagination` |

## Recommended Project Structure

```
blocks/
├── group.liquid          # EXISTS — layout wrapper (row/column)
├── text.liquid           # EXISTS — generic styled text
├── image-text.liquid     # NEW — side-by-side image + text, configurable side
├── multi-column.liquid   # NEW — 2–3 column text block
├── quote.liquid          # NEW — testimonial or pull-quote
└── collection-grid.liquid # NEW — product grid, droppable onto any page

sections/
├── about-origin.liquid   # NEW — brand origin story (can use image-text block internally or theme blocks)
├── about-mission.liquid  # NEW — mission/ethos section
├── about-team.liquid     # NEW — team section with person blocks
├── custom-section.liquid # EXISTS — generic container that accepts @theme blocks
├── [all existing sections]

templates/
├── page.about-us.json    # NEW — alternate page template, references 3 about sections
├── [all existing templates]

locales/
├── en.default.json       # UPDATE — add keys for all new sections/blocks
```

### Structure Rationale

- **`blocks/` for reusable components:** The four new content blocks (image-text, multi-column, quote, collection-grid) go in `blocks/` because they need to be draggable onto any page via the theme editor, not locked to a single section.
- **`sections/` for About Us story containers:** The three About Us sections (origin, mission, team) are page-specific containers. They can either render fixed branded content with their own schema settings, or host `@theme` blocks inside them for maximum flexibility.
- **`templates/page.about-us.json`:** Shopify alternate page template naming convention — `page.{suffix}.json`. Merchants assign this template to the About Us page via the admin.
- **Existing `custom-section.liquid`:** Already accepts `@theme` blocks. The About Us page can use it as a flexible container alongside dedicated story sections.

## Architectural Patterns

### Pattern 1: Theme Blocks vs Section-Defined Blocks

**What:** Shopify has two types of blocks. Section-defined blocks exist only inside one section's schema (like tabs inside a product section). Theme blocks live in `/blocks/` and are reusable across any section that declares `"blocks": [{ "type": "@theme" }]`.

**CRITICAL RULE:** A section must choose one approach — it cannot mix both. If a section uses `"blocks": [{ "type": "@theme" }]`, it cannot also define local block types in the same schema.

**When to use theme blocks:** When the component needs to be draggable across multiple sections or page types. All four new blocks (image-text, multi-column, quote, collection-grid) should be theme blocks.

**When to use section-defined blocks:** When blocks are semantically tied to one section and make no sense elsewhere (example: individual slides inside a carousel section). The existing `shop-by-category.liquid` uses section-defined `category` blocks this way.

**Example — theme block declaration in a section schema:**
```json
{
  "name": "About Origin",
  "blocks": [{ "type": "@theme" }],
  "settings": [...]
}
```

**Example — theme block file with `@theme` in its own schema for nesting:**
```json
{
  "name": "Image + Text",
  "blocks": [{ "type": "@theme" }],
  "settings": [...]
}
```

### Pattern 2: Section as Branded Container, Blocks as Content

**What:** For the About Us page, each story section (origin, mission, team) owns the branded frame — background color, section heading, padding, eyebrow labels. The section schema exposes these as editor settings. Content detail lives inside blocks (or the section itself for simpler copy).

**When to use:** When a section has a distinct branded purpose and visual frame, but needs flexible internal content. The `about-team.liquid` section is the clearest example: it sets up the team section header and grid, then renders `{% content_for 'blocks' %}` for individual person cards as blocks.

**Example structure for about-team.liquid:**
```liquid
<section class="about-team" style="--team-bg: {{ section.settings.background_color }}">
  <div class="about-team__header">
    <span class="eyebrow">{{ section.settings.eyebrow_text }}</span>
    <h2>{{ section.settings.heading }}</h2>
  </div>
  <div class="about-team__grid">
    {% content_for 'blocks' %}
  </div>
</section>

{% schema %}
{
  "name": "Team",
  "blocks": [{ "type": "@theme" }],
  "settings": [...]
}
{% endschema %}
```

### Pattern 3: `{% style %}` for Editor Live-Preview, `{% stylesheet %}` for Structure

**What:** This pattern already exists in all Ruck On sections. Settings that a merchant changes in the editor (colors, spacing amounts) go in `{% style %}` blocks that write scoped CSS custom properties. Static structural CSS (layout, typography scale, media queries) goes in `{% stylesheet %}`.

**Why this matters for new blocks:** Every new block must follow this pattern. If a merchant can change the image position (left/right) in the editor, that drives a CSS class via `{{ block.settings.layout }}`. If they can set a background color, that goes in `{% style %}` as a CSS variable.

**Example for image-text block:**
```liquid
{% style %}
  #{{ block.id }} {
    --it-gap: {{ block.settings.gap }}px;
    --it-img-ratio: {{ block.settings.image_ratio }};
  }
{% endstyle %}

<div
  id="{{ block.id }}"
  class="image-text {{ block.settings.image_side }}"
  {{ block.shopify_attributes }}
>
  ...
</div>

{% stylesheet %}
  .image-text {
    display: grid;
    gap: var(--it-gap);
  }
  .image-text--left { grid-template-columns: 1fr 1fr; }
  .image-text--right { grid-template-columns: 1fr 1fr; direction: rtl; }
{% endstylesheet %}
```

### Pattern 4: Alternate Page Template for About Us

**What:** Shopify allows alternate JSON templates per page type using the naming convention `page.{suffix}.json`. Creating `templates/page.about-us.json` creates an alternate template that merchants assign to their About Us page in the Shopify admin (Pages > select page > Template dropdown).

**When to use:** Any time a specific page needs a distinct section layout that differs from the generic `page.json` template (which currently just renders `sections/page.liquid`).

**Template file structure:**
```json
{
  "sections": {
    "about-origin": {
      "type": "about-origin",
      "settings": {
        "background_color": "#111009",
        "eyebrow_text": "// Our Story"
      }
    },
    "about-mission": {
      "type": "about-mission",
      "settings": {}
    },
    "about-team": {
      "type": "about-team",
      "settings": {}
    }
  },
  "order": ["about-origin", "about-mission", "about-team"]
}
```

### Pattern 5: Block Nesting via group.liquid

**What:** The existing `blocks/group.liquid` is a layout wrapper that accepts `@theme` blocks inside it. This enables a two-level nesting pattern: a section contains group blocks, and group blocks contain content blocks (text, image-text, etc.).

**When to use:** When you need to arrange multiple blocks in a row or column layout within a section. For About Us, the mission section could use groups to place an image-text block alongside a stats block.

**Depth limit:** There is no documented hard depth limit, but the Shopify editor becomes unwieldy beyond 2–3 levels. Keep nesting shallow in practice.

**Nesting chain:**
```
about-mission section
  └── group block (horizontal layout)
        ├── image-text block
        └── multi-column block
```

### Pattern 6: Collection Grid Block Using collection Setting

**What:** The `collection-grid` block needs a `collection` picker setting to pull products. It renders a product grid using the same `.product-card` pattern from `critical.css`. Since blocks don't have access to the Liquid `paginate` tag (that's for sections), the collection grid block limits products via the `limit` filter.

**Key constraint:** Blocks cannot use `{% paginate %}`. The collection grid block must use `| limit: N` on the products array. Pagination requires a section-level wrapper.

**Example schema setting:**
```json
{
  "type": "collection",
  "id": "collection",
  "label": "Collection to show"
}
```

**Example render:**
```liquid
{% assign grid_collection = collections[block.settings.collection] %}
{% for product in grid_collection.products limit: block.settings.product_limit %}
  <article class="product-card">...</article>
{% endfor %}
```

## Data Flow

### About Us Page Request Flow

```
Merchant creates page in Shopify admin
    ↓
Assigns "page.about-us" template to the page
    ↓
User visits /pages/about-us
    ↓
Shopify reads templates/page.about-us.json
    ↓
Renders sections in order: about-origin → about-mission → about-team
    ↓
Each section reads section.settings.* from JSON template
    ↓
Sections with blocks call {% content_for 'blocks' %}
    ↓
Shopify renders each block in its stored order (from template JSON)
    ↓
Block reads block.settings.* — image, text, layout, etc.
    ↓
{% style %} outputs scoped CSS vars for color/spacing settings
    ↓
{% stylesheet %} structural CSS is bundled by Shopify (deduped)
    ↓
Final HTML returned to browser
```

### Block Settings Update Flow (Theme Editor)

```
Merchant changes block setting in editor sidebar
    ↓
{% style %} block re-executes with new value → CSS var updates live
    (No page reload needed for color/spacing settings)
    ↓
CSS class changes (layout, columns) trigger DOM class swap
    ↓
Shopify writes updated settings back to template JSON on save
```

### Collection Grid Block Data Flow

```
block.settings.collection (handle string)
    ↓
{% assign grid_collection = collections[block.settings.collection] %}
    ↓
{% for product in grid_collection.products limit: block.settings.product_limit %}
    ↓
Renders .product-card markup (reuses critical.css styles)
    ↓
No JS required — server-side rendered
```

## Build Order and Dependencies

Build in this order because each layer depends on the previous:

```
1. blocks/image-text.liquid
   └── No dependencies. Pure Liquid + CSS.

2. blocks/multi-column.liquid
   └── No dependencies. Pure Liquid + CSS.

3. blocks/quote.liquid
   └── No dependencies. Pure Liquid + CSS.

4. blocks/collection-grid.liquid
   └── Depends on: assets/critical.css (.product-card styles already exist)

5. sections/about-origin.liquid
   └── Depends on: blocks being available if using @theme blocks
       Can also be built as standalone section with own image-text markup

6. sections/about-mission.liquid
   └── Same as above

7. sections/about-team.liquid
   └── Depends on: blocks/image-text.liquid OR a dedicated person block
       (team members rendered as blocks inside this section)

8. templates/page.about-us.json
   └── Depends on: sections 5, 6, 7 existing

9. locales/en.default.json updates
   └── Depends on: all new sections/blocks having final key names

10. sections/collection.liquid schema additions
    └── Depends on: existing collection.liquid (just adding schema settings)
```

**Why blocks before sections:** Sections that use `{% content_for 'blocks' %}` with `@theme` type will be empty in the editor until blocks exist. Build blocks first so sections are immediately testable.

**Why collection.liquid last:** It's an isolated schema addition (image ratio, card style settings). No dependency on the new blocks or template.

## Anti-Patterns

### Anti-Pattern 1: Mixing Section-Defined and Theme Blocks

**What people do:** Add `"blocks": [{ "type": "@theme" }, { "type": "slide" }]` to mix theme blocks with locally-defined block types.

**Why it's wrong:** Shopify explicitly prohibits this — "sections can either define blocks locally or opt-in to supporting theme blocks, but they can't support both simultaneously." It will cause a schema validation error.

**Do this instead:** If a section needs both reusable blocks and custom block types, convert all block types to theme blocks (put them in `/blocks/` folder).

### Anti-Pattern 2: Putting Complex Shared Logic in Sections

**What people do:** Build a featured-products section with custom layout, then build it again in the About Us page, and again in a landing page.

**Why it's wrong:** CSS and Liquid logic duplicated across three sections. Any change needs to happen in three places.

**Do this instead:** Put the reusable component in `/blocks/collection-grid.liquid`. Any section that declares `"blocks": [{ "type": "@theme" }]` can now host it. One file to maintain.

### Anti-Pattern 3: Hardcoding Brand Copy in Section Markup

**What people do:** Write `<h2>Our Story</h2>` directly in the section's Liquid markup.

**Why it's wrong:** Violates the CLAUDE.md requirement that all text must be editor-configurable via `section.settings.*`. Merchants cannot change it without code access.

**Do this instead:** Every string — headings, body copy, button labels, eyebrow text — must be a `text` or `richtext` setting in the schema, defaulted to the brand copy.

### Anti-Pattern 4: Using `{% paginate %}` Inside a Block

**What people do:** Try to paginate a product list inside a `collection-grid` block.

**Why it's wrong:** The `{% paginate %}` tag is only valid at the template/section level where Shopify can intercept the URL parameters. Inside a block it will throw a Liquid error.

**Do this instead:** Use `limit` filter on the products array inside the block. If pagination is needed, wrap the collection grid in a dedicated section (not a block).

### Anti-Pattern 5: Skipping `{% doc %}` Tags on Blocks

**What people do:** Write block files without the LiquidDoc header.

**Why it's wrong:** CLAUDE.md mandates `{% doc %}` on all blocks (and snippets). Missing it breaks tooling conventions and makes the block ineligible for static `{% content_for 'block', type: '...', id: '...' %}` rendering.

**Do this instead:** Every block file starts with a `{% doc %}...{% enddoc %}` block documenting its purpose and `@example` usage.

### Anti-Pattern 6: Section-Level CSS for Block Styles

**What people do:** Put `.image-text` styles inside the parent section's `{% stylesheet %}` tag.

**Why it's wrong:** When the block is used in a different section, those styles don't exist. The block renders without styling.

**Do this instead:** Put ALL styling for a block inside the block's own `{% stylesheet %}` tag. Shopify deduplicates these across renders, so there's no penalty for the same styles appearing multiple times.

## Integration Points

### Shopify Theme Editor Integration

| Setting Type | Block Context | Notes |
|--------------|---------------|-------|
| `image_picker` | `block.settings.image` | Returns Shopify image object; use `\| image_url: width: N \| image_tag` |
| `collection` | `block.settings.collection` | Returns handle; access as `collections[block.settings.collection]` |
| `richtext` | `block.settings.body` | Returns HTML string; output with `{{ block.settings.body }}` (no escape) |
| `select` | `block.settings.image_side` | Values should be CSS class names or direct CSS values |
| `range` | `block.settings.gap` | Integer; use in `{% style %}` as CSS var with `px` unit suffix |
| `color` | `block.settings.bg_color` | Hex string; use in `{% style %}` directly |

### Critical CSS Shared Styles

The `blocks/collection-grid.liquid` block must reuse existing shared styles from `assets/critical.css` without redefining them:

| Class | Defined In | Block Can Use |
|-------|-----------|---------------|
| `.product-card` | `critical.css` | Yes — use directly in block markup |
| `.product-card__media` | `critical.css` | Yes — reuse DOM structure |
| `.product-card__info` | `critical.css` | Yes — reuse DOM structure |
| `.btn .btn--primary` | `critical.css` | Yes — use for CTAs in blocks |
| `.fade-up` | `critical.css` | Yes — add to block elements for scroll animations |

### Translation Keys Needed

New locale keys to add in `locales/en.default.json`:

```
sections.about_origin.*    — heading, eyebrow, body, cta
sections.about_mission.*   — heading, eyebrow, body
sections.about_team.*      — heading, eyebrow
blocks.image_text.*        — image_alt, cta_text
blocks.multi_column.*      — (all text settings have defaults in schema)
blocks.quote.*             — attribution_label
blocks.collection_grid.*   — heading, view_all_text, empty_text
```

## Sources

- [Shopify Theme Blocks — Official Docs](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks) — HIGH confidence
- [Shopify JSON Templates — Official Docs](https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates) — HIGH confidence
- [Sections vs Blocks Best Practices — Official Docs](https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks) — HIGH confidence
- [Shopify Sections — Official Docs](https://shopify.dev/docs/storefronts/themes/architecture/sections) — HIGH confidence
- [Existing codebase — .planning/codebase/ARCHITECTURE.md](../codebase/ARCHITECTURE.md) — HIGH confidence (primary source for existing patterns)
- [Existing blocks/group.liquid and blocks/text.liquid](../../blocks/) — HIGH confidence (live pattern examples)
- [Existing sections/collection.liquid](../../sections/collection.liquid) — HIGH confidence (confirms CSS var + schema pattern)

---
*Architecture research for: Ruck On — About Us page and reusable block toolkit*
*Researched: 2026-03-06*
