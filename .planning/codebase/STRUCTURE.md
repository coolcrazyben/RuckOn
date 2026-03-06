# Codebase Structure

**Analysis Date:** 2025-02-06

## Directory Layout

```
ruck-on/
├── assets/                 # Critical CSS, icons, images loaded on every page
├── blocks/                 # Reusable sub-components (text, group, etc.)
├── config/                 # Global theme settings schema and data
├── layout/                 # HTML document wrappers (theme.liquid, password.liquid)
├── locales/                # Translation strings for all languages
├── sections/               # Full-width page sections and section groups
├── snippets/               # Reusable code fragments (no editor interface)
├── templates/              # Page type templates (JSON files composing sections)
├── .planning/              # Planning and analysis documents (created by GSD)
└── CLAUDE.md               # Brand guide and development standards
```

## Directory Purposes

**`assets/`:**
- Purpose: Static files loaded on every page (critical CSS, global icons)
- Contains: CSS reset, global utilities, brand colors, shared component styles, SVG icons
- Key files:
  - `critical.css` — loaded via `preload` tag in `<head>`, contains reset, section grid, buttons, animations, product cards, pagination, RTE styles, form inputs
  - `icon-cart.svg` — shopping bag icon for cart button
  - `icon-account.svg` — user icon for account link
  - `shoppy-x-ray.svg` — unused asset (legacy)
- Generated: No
- Committed: Yes

**`blocks/`:**
- Purpose: Reusable nested components that merchants can add/remove/reorder within sections
- Contains: Small UI components with their own schema for editor customization
- Key files:
  - `text.liquid` — generic text block with style options (title, subtitle, normal)
  - `group.liquid` — layout wrapper for nesting other blocks (flex row/col, gap, padding control)
- Generated: No
- Committed: Yes

**`config/`:**
- Purpose: Global theme customization schema and current settings
- Contains: JSON configuration for what merchants can edit at theme level
- Key files:
  - `settings_schema.json` — defines all global settings: Favicon, Typography (fallback font), Layout (max width, margin), Colors (bg, fg, accent)
  - `settings_data.json` — current values (dark theme defaults: black bg, white fg, tan accent)
- Generated: Partially — `settings_data.json` auto-updated by Shopify admin
- Committed: Yes (schema yes, data mostly auto-generated)

**`layout/`:**
- Purpose: Top-level HTML templates that wrap all other page content
- Contains: Document structure, font imports, global scripts, head metadata
- Key files:
  - `theme.liquid` — standard page layout: renders `header-group` + `main` + `footer-group` + scroll-animations snippet
  - `password.liquid` — password protection page layout (no header/footer, full width)
- Generated: No
- Committed: Yes

**`locales/`:**
- Purpose: Translation strings for multi-language support and editor labels
- Contains: JSON files mapping keys to localized text
- Key files:
  - `en.default.json` — all user-facing text (section titles, labels, button text, etc.)
  - `en.default.schema.json` — translation keys for editor field names and descriptions
- Pattern: Keys use hierarchical dot notation: `general.text`, `sections.hero.headline`, `labels.alignment`
- Generated: No (maintained manually)
- Committed: Yes

**`sections/`:**
- Purpose: Full-width page components, section groups, and modular content blocks
- Contains: Liquid files (`.liquid`) with embedded schema, plus JSON files for grouping sections
- Page sections (25 total):
  - `hero.liquid` — full-viewport hero with background image, overlay, eyebrow, two-line headline, subtext, dual buttons
  - `announcement-bar.liquid` — sticky top bar with promo text
  - `header.liquid` — sticky navigation (3-col grid: left nav / center logo / right search+account+cart)
  - `stats-bar.liquid` — 4-stat display (10K+ ruckers, 500+ miles, 100% guarantee, 5★ rating)
  - `best-sellers.liquid` — product grid from collection setting, 6 products max, with scroll animations
  - `manifesto.liquid` — centered quote section with two-line quote, body text, CTA
  - `shop-by-category.liquid` — 4-column category grid with `category` blocks
  - `community.liquid` — two-column layout with overlapping images, stats, badge
  - `newsletter.liquid` — email signup form with Shopify customer form
  - `footer.liquid` — 4-column footer (brand info + 3 link menus)
  - `product.liquid` — full product detail: gallery with thumbnails, variant pills, qty stepper, tabs, related products
  - `collection.liquid` — collection page: banner, sort dropdown, product grid, pagination
  - `cart.liquid` — cart page: line items with qty stepper, remove buttons, subtotal, checkout
  - `search.liquid` — search page: input, product results grid, no-results state
  - `blog.liquid` — blog listing: article cards with image, date, excerpt
  - `article.liquid` — article page: hero image, content (RTE), comment form
  - `page.liquid` — generic page: centered title, RTE content area
  - `404.liquid` — 404 page: large code, "Mission Not Found" copy, return home CTA
  - `collections.liquid` — collections list: card grid of all collections
  - `password.liquid` — password form section (Shopify handles markup, we style)
  - `custom-section.liquid` — example template for custom sections
  - `hello-world.liquid` — example/test section
  - `community-creation-page.liquid` — custom page for community creation feature
- Section groups (JSON files grouping sections):
  - `header-group.json` — combines `announcement-bar` + `header` in fixed order
  - `footer-group.json` — combines footer sections
- Generated: Partially — JSON files auto-updated by Shopify editor
- Committed: Yes

**`snippets/`:**
- Purpose: Reusable code fragments rendered via `{% render %}` tag (no editor interface)
- Contains: Helpers, partial markup, initialization scripts
- Key files:
  - `css-variables.liquid` — renders `<style>` block with `:root` CSS custom properties (brand colors, fonts, layout vars, header heights)
  - `scroll-animations.liquid` — renders `<script>` with global IntersectionObserver for `.fade-up` animation class
  - `meta-tags.liquid` — renders `<meta>` tags in `<head>`: OG (open graph), Twitter cards, title, canonical, structured data
  - `image.liquid` — responsive image component with optional link wrapper, width/height/crop control
  - `community-list.liquid` — renders `<option>` elements for community selector dropdown (on basic membership products)
- Generated: No
- Committed: Yes

**`templates/`:**
- Purpose: Define page structure by composing sections and blocks
- Contains: JSON files (one per page type) that list which sections appear and in what order
- Key files:
  - `index.json` — homepage: hero → stats-bar → best-sellers → manifesto → shop-by-category → community → newsletter (7 sections)
  - `product.json` — product page: just the `product` section as main
  - `collection.json` — collection page: `collection` section
  - `cart.json` — cart page: `cart` section
  - `search.json` — search page: `search` section
  - `article.json` — blog post page: `article` section
  - `blog.json` — blog listing page: `blog` section
  - `page.json` — generic page: `page` section
  - `list-collections.json` — collections listing: `collections` section
  - `404.json` — not found page: `404` section
  - `password.json` — password page: rendered by layout, not sections
  - `page.community-creation.json` — custom page for community creation
  - `product.community-creation.json` — custom product template for community creation
  - `gift_card.liquid` — Shopify gift card template (Liquid, not JSON)
- Pattern: Each JSON has `"sections": { "id": { "type": "section-name", "settings": {...} } }` and `"order": ["id1", "id2"]`
- Generated: Partially — auto-updated by Shopify editor when merchants reorder/add sections
- Committed: Yes

## Key File Locations

**Entry Points:**
- `layout/theme.liquid` — primary HTML wrapper, loads fonts, CSS, renders page content
- `layout/password.liquid` — password page wrapper (minimal, no header/footer)
- `templates/index.json` — homepage template (7 sections in order)
- `templates/product.json` — product detail template

**Configuration:**
- `config/settings_schema.json` — global theme settings (favicon, typography, layout, colors)
- `config/settings_data.json` — current global settings values
- `sections/header-group.json` — header/announcement bar configuration
- `sections/footer-group.json` — footer configuration

**Core Logic:**
- `sections/hero.liquid` — full-screen hero section with animations
- `sections/product.liquid` — product page logic with variant pills, qty stepper, tabs, JS
- `sections/best-sellers.liquid` — product grid from collection
- `sections/collection.liquid` — collection page with sort and pagination
- `snippets/css-variables.liquid` — all design system CSS variables

**Styling & Animation:**
- `assets/critical.css` — all global styles, loaded on every page
- `snippets/scroll-animations.liquid` — IntersectionObserver for `.fade-up` animations
- Individual section `{% style %}` blocks — section-specific CSS bound to `#shopify-section-{{ section.id }}`

**Testing:**
- No test files (Shopify Liquid themes don't have standard test infrastructure)
- Manual testing via Shopify theme preview

## Naming Conventions

**Files:**
- Sections: `kebab-case.liquid` — `hero.liquid`, `best-sellers.liquid`, `announcement-bar.liquid`
- Blocks: `kebab-case.liquid` — `text.liquid`, `group.liquid`
- Snippets: `kebab-case.liquid` — `css-variables.liquid`, `scroll-animations.liquid`
- Templates: `kebab-case.json` (page type) or `kebab-case.liquid` (gift card) — `index.json`, `product.json`, `article.json`
- Assets: lowercase with extensions — `critical.css`, `icon-cart.svg`
- Locales: language code + optional region + extension — `en.default.json`, `en.default.schema.json`

**Directories:**
- Always lowercase plural — `sections/`, `snippets/`, `blocks/`, `templates/`, `assets/`, `locales/`, `config/`, `layout/`

**CSS Classes & Variables:**
- BEM naming: `.block__element--modifier` — `.product-card__info`, `.product-card--tall`, `.site-header__logo`
- CSS custom properties: `--kebab-case` with sections scoped — `--bs-bg` (best-sellers background), `--header-link` (header link color)
- Utility classes: `.btn`, `.fade-up`, `.eyebrow`, `.rte`, `.form-input`
- JavaScript hooks: `id="product-variant-id"`, `id="main-content"`, `data-delay="2"`

**Liquid Variables:**
- section.settings: `kebab_case_id` — `section.settings.background_color`, `section.settings.heading_text_1`
- Local assigns: `snake_case` — `assign featured_collection = ...`
- Loop variables: `singular` — `for product in products`, `for link in nav_links`

## Where to Add New Code

**New Section:**
- Create: `sections/my-section.liquid`
- Structure:
  ```liquid
  {% style %}
    #shopify-section-{{ section.id }} .my-section {
      --color: {{ section.settings.color }};
    }
  {% endstyle %}

  <div class="my-section">Content here</div>

  {% schema %}
  { "name": "My Section", "settings": [ ... ] }
  {% endschema %}
  ```
- Register in template: Add to `templates/page-type.json`

**New Block:**
- Create: `blocks/my-block.liquid`
- Structure: Same as section but smaller scope, use `block.settings.*` instead of `section.settings.*`
- Register in section: Add `"blocks": [{ "type": "@theme" }]` or `"type": "my-block"` to section schema

**New Snippet:**
- Create: `snippets/my-helper.liquid`
- Include `{% doc %}` header with purpose and parameters
- Render via: `{% render 'my-helper', param1: value1 %}`

**New Template:**
- Create: `templates/page-type.json` if new page type
- Structure: `{ "sections": {...}, "order": [...] }`
- Use existing sections or create new ones

**Utilities / Shared CSS:**
- Add to `assets/critical.css` if used on multiple pages
- Add to section `{% stylesheet %}` if used only in that section
- Prefer CSS custom properties over hardcoded values
- Reference design system variables: `var(--color-orange)`, `var(--font-heading)`

**Global Styling Updates:**
- Update: `snippets/css-variables.liquid` for brand color/font changes
- Or: `config/settings_schema.json` to expose new global settings

## Special Directories

**`.planning/`:**
- Purpose: Architecture and codebase documentation generated by GSD
- Generated: Yes (by GSD mapper)
- Committed: Yes (documentation in source control)
- Contains: ARCHITECTURE.md, STRUCTURE.md, CONVENTIONS.md, TESTING.md, CONCERNS.md, STACK.md, INTEGRATIONS.md

**`.claude/`:**
- Purpose: Claude Code project settings and memory
- Generated: Yes (by Claude Code IDE)
- Committed: Possibly (if checked in)
- Contains: Editor configuration, project memory (MEMORY.md)

**Root files:**
- `CLAUDE.md` — Brand guide, development standards, Liquid reference
- `package.json` — (if present) for dependencies like Prettier, ESLint (not applicable to Liquid themes)

## File Count Summary

- **Sections:** 25 (22 page sections + 2 section groups + 1 custom)
- **Blocks:** 2 (text, group)
- **Snippets:** 5 (css-variables, scroll-animations, meta-tags, image, community-list)
- **Templates:** 14 (12 JSON page templates + 1 gift card Liquid + 1 test)
- **Assets:** 4 (critical.css + 3 SVG icons)
- **Config:** 2 (settings_schema.json, settings_data.json)
- **Locales:** 2 (en.default.json, en.default.schema.json)
- **Layouts:** 2 (theme.liquid, password.liquid)

---

*Structure analysis: 2025-02-06*
