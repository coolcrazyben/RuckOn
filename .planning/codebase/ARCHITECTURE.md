# Architecture

**Analysis Date:** 2025-02-06

## Pattern Overview

**Overall:** Shopify Liquid theme using a component-based section architecture with shared layout primitives. The theme separates content (sections, blocks, templates) from presentation (critical CSS) and global functionality (snippets), following Shopify's theme conventions.

**Key Characteristics:**
- Sections-first composition — entire pages built from configurable, nestable sections
- Centralized design system via CSS variables in `css-variables.liquid`
- Global scroll animation system using IntersectionObserver
- Template-driven page composition via JSON files that combine sections
- All content editable from Shopify theme editor without code changes

## Layers

**Layout Layer:**
- Purpose: HTML document wrapper, fonts, head metadata, global CSS
- Location: `layout/theme.liquid`, `layout/password.liquid`
- Contains: DOCTYPE, stylesheet/font imports, section group rendering, main content area
- Depends on: `snippets/css-variables.liquid`, `snippets/meta-tags.liquid`, `assets/critical.css`
- Used by: All pages (via Shopify's implicit `{% layout %}` tag)

**Page Template Layer:**
- Purpose: Define which sections appear on each page type and in what order
- Location: `templates/*.json` (e.g., `templates/index.json`, `templates/product.json`, `templates/collection.json`)
- Contains: Section IDs, settings objects, render order
- Depends on: Section definitions in `sections/`
- Used by: Shopify rendering engine to compose pages
- Key files: `index.json` (homepage), `product.json` (product detail), `collection.json` (collection listing), `article.json` (blog post)

**Section Layer:**
- Purpose: Full-width page components that can be reordered and edited in theme editor
- Location: `sections/*.liquid` (also `sections/*-group.json` for nested section groups)
- Contains: Styling, Liquid logic, schema definitions for editor settings
- Depends on: Global CSS variables, snippets for rendering, sometimes blocks
- Used by: Templates and section groups
- Pattern: Each section has three main parts:
  1. `{% style %}` block mapping Liquid variables to CSS custom properties
  2. Main HTML/Liquid markup
  3. `{% schema %}` defining editor fields and settings

**Block Layer:**
- Purpose: Reusable sub-components within sections (nested, addable, removable)
- Location: `blocks/*.liquid`
- Contains: Small, focused components with their own schema
- Depends on: CSS variables, parent section for data
- Used by: Sections via `{% content_for 'blocks' %}`
- Examples: `text.liquid` (generic text block), `group.liquid` (layout wrapper for nested blocks)

**Snippet Layer:**
- Purpose: Reusable code fragments without editor interface
- Location: `snippets/*.liquid`
- Contains: Helpers, repeated markup, initialization scripts
- Depends on: CSS variables, global objects
- Used by: Layout, sections, other snippets
- Key files:
  - `css-variables.liquid` — defines all brand colors, fonts, spacing as CSS custom properties
  - `scroll-animations.liquid` — global IntersectionObserver for `.fade-up` animations
  - `meta-tags.liquid` — social, OG tags, structured data, title/description
  - `image.liquid` — responsive image wrapper
  - `community-list.liquid` — dropdown options for membership community selector

**Asset Layer:**
- Purpose: Critical CSS loaded on every page, SVG icons, global styles
- Location: `assets/`
- Contains: Utilities, resets, shared component styles, global animations
- Key file: `assets/critical.css`
  - Global reset and box model
  - `.shopify-section` grid layout system (3-col with center content area)
  - Global button classes (`.btn--primary`, `.btn--secondary` with clip-path skew)
  - Scroll animation classes (`.fade-up`, `.fade-up.in-view`, delay helpers)
  - Shared product card styles (`.product-card`, `.product-card--tall`)
  - Pagination styles (`.pagination`)
  - Rich text editor styles (`.rte`)
  - Form input styles (`.form-input`)

**Configuration Layer:**
- Purpose: Global theme settings and metadata
- Location: `config/`
- Contains: Theme-wide customization schema and current settings
- Key files:
  - `settings_schema.json` — defines global settings for Typography, Layout, Colors (favicon, max page width, margin, brand colors)
  - `settings_data.json` — current values for global settings

**Localization Layer:**
- Purpose: Translation keys and schema descriptions for multi-language support
- Location: `locales/`
- Contains: JSON files with translation keys
- Key files:
  - `en.default.json` — all user-facing text strings
  - `en.default.schema.json` — translation keys for editor field labels

## Data Flow

**Homepage Composition:**

1. Shopify renders `templates/index.json`
2. Template defines 7 sections: hero → stats-bar → best-sellers → manifesto → shop-by-category → community → newsletter
3. Each section reads from `templates/index.json` settings (colors, text, images)
4. Section renders Liquid markup + CSS variables via `{% style %}`
5. Products in `best-sellers` section pull from collection setting, loop through max 6 products
6. Scroll animation system observes `.fade-up` elements, adds `.in-view` class on intersection
7. Layout footer is rendered via `{% sections 'footer-group' %}`

**Product Page Data Flow:**

1. Shopify renders `templates/product.json` with `sections/product.liquid`
2. Section receives `product` object from Shopify
3. Product variant data passed to JavaScript via `<script type="application/json" id="product-variants-json">`
4. JavaScript listens to variant pill clicks, updates hidden input, fetches corresponding variant
5. Price, availability, images update in DOM based on variant selection
6. Related products section queries product recommendations

**State Management:**

- **Global state:** CSS variables in `:root` set by `snippets/css-variables.liquid` (colors, fonts, spacing)
- **Section state:** Each section reads `section.settings.*` from its schema, passed to `{% style %}`
- **Client state:** Product page manages variant selection via JavaScript DOM manipulation (no frameworks)
- **Scroll state:** IntersectionObserver tracks which `.fade-up` elements are in viewport

## Key Abstractions

**Grid System:**
- Purpose: Consistent layout with responsive side margins
- Examples: `assets/critical.css` lines 52–67
- Pattern: `.shopify-section` uses `--content-grid: margin min-width margin` CSS grid
  - All children go to column 2 (the narrow content column)
  - `.full-width` class makes children span columns 1–3 (edge-to-edge)
  - Responsive via calc() — respects `--page-width` and `--page-margin` settings

**CSS Variable Cascading:**
- Purpose: Centralized design system, merchant-editable colors
- Example: `css-variables.liquid` defines brand colors as `:root` variables
- Pattern: Sections map `section.settings.color` to CSS custom properties, then use them in `{% style %}`
  - Hero section: `--hero-bg: {{ section.settings.background_color }}`
  - Button: `--btn-bg: var(--color-orange)`
  - Allows theme editor to change colors without touching code

**Scroll Animations:**
- Purpose: Lazy-load animations on viewport entry
- Example: `snippets/scroll-animations.liquid`
- Pattern: Global IntersectionObserver watches `.fade-up` class
  - On intersection, adds `.in-view` class, unobserves
  - CSS defines transition from `opacity: 0; transform: translateY(28px)` to default
  - `[data-delay]` attributes stagger animations

**Product Card Archetype:**
- Purpose: Reusable layout across best-sellers, collection, search results
- Examples: `assets/critical.css` lines 173–290
- Pattern: `.product-card` base class with modifiers
  - `.product-card--tall` for featured card (takes 2 rows, image fills height)
  - Consistent DOM structure: `.product-card__link` > `.product-card__media` + `.product-card__info`
  - Image hover scales, CTA fades in on hover
  - Used in: `best-sellers.liquid`, `collection.liquid`, `search.liquid`

**Form Input System:**
- Purpose: Consistent dark-themed form styling
- Example: `assets/critical.css` lines 368–387
- Pattern: `.form-input` class applied to `<input>`, `<select>`, `<textarea>`
  - Transparent dark background with tan border
  - Focus state lightens border
  - Used in: newsletter signup, product variant selector, login forms

## Entry Points

**Home Page:**
- Location: `layout/theme.liquid` → `templates/index.json` → 7 sections in order
- Triggers: User visits `/` or site root
- Responsibilities: Render header group, hero, stats, products, manifesto, category grid, community section, newsletter, footer group

**Product Page:**
- Location: `layout/theme.liquid` → `templates/product.json` → `sections/product.liquid`
- Triggers: User visits `/products/{handle}`
- Responsibilities: Display product gallery, variant options, price, quantity selector, add-to-cart form, reviews/tabs, related products

**Collection Page:**
- Location: `layout/theme.liquid` → `templates/collection.json` → `sections/collection.liquid`
- Triggers: User visits `/collections/{handle}`
- Responsibilities: Display collection hero/banner, sort dropdown, product grid with pagination

**Search Page:**
- Location: `layout/theme.liquid` → `templates/search.json` → `sections/search.liquid`
- Triggers: User submits search query
- Responsibilities: Display search input, product results grid, no-results state

**Cart Page:**
- Location: `layout/theme.liquid` → `templates/cart.json` → `sections/cart.liquid`
- Triggers: User navigates to `/cart`
- Responsibilities: Display cart items with qty steppers, remove buttons, subtotal, checkout button, empty state

**Password Page:**
- Location: `layout/password.liquid` → `templates/password.json` → `sections/password.liquid`
- Triggers: Store is password-protected
- Responsibilities: Display password form (Shopify automatic), brand styling via layout CSS

## Error Handling

**Strategy:** Graceful degradation with fallbacks

**Patterns:**
- Missing product images: Use `{{ 'product-1' | placeholder_svg_tag }}` placeholder
- Missing collection: Default to `collections.frontpage` (best-sellers section)
- Missing variant: Use `product.selected_or_first_available_variant`
- Missing settings: Use `section.settings.color | default: '#111009'` pattern
- Missing sections: Page templates can be edited in Shopify editor to add missing sections

## Cross-Cutting Concerns

**Logging:** No custom logging. Shopify liquid errors are logged by platform.

**Validation:**
- Product form uses Shopify's built-in `{% form 'product' %}` validation
- Variant selection validated client-side in product.liquid JavaScript section
- Newsletter form uses Shopify customer form validation

**Authentication:**
- Handled by Shopify's built-in customer accounts
- Login/logout links rendered via `{{ routes.account_login_url }}` and similar global objects
- Password protection template includes Shopify's automatic password form

**Accessibility:**
- Semantic HTML: `<header role="banner">`, `<main id="main-content">`, `<nav aria-label="...">`
- ARIA attributes on interactive elements: `aria-pressed`, `aria-expanded`, `aria-current="page"`
- Alt text on all images via Liquid filters
- Skip-to-main-content pattern (implicit in Shopify layouts)
- Color contrast follows WCAG guidelines (tan on black, white on dark backgrounds)

---

*Architecture analysis: 2025-02-06*
