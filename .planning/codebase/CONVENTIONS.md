# Coding Conventions

**Analysis Date:** 2025-02-14

## Naming Patterns

**Files:**
- Sections: kebab-case (e.g., `best-sellers.liquid`, `announcement-bar.liquid`)
- Blocks: kebab-case (e.g., `group.liquid`, `text.liquid`)
- Snippets: kebab-case (e.g., `css-variables.liquid`, `scroll-animations.liquid`)
- Assets: kebab-case (e.g., `critical.css`)
- All lowercase, no spaces

**CSS Classes:**
- Block Element Modifier (BEM) pattern: `.component__element--modifier`
- Examples:
  - `.site-header__logo` (element within component)
  - `.site-header__nav-link` (nested elements use double underscore only once)
  - `.site-header__nav-link--active` (modifier state)
  - `.product-card__title` (element)
  - `.product-card--tall` (modifier)
- State classes: `.is-active`, `.is-hidden`, `.is-scrolled`, `.is-open` (prefixed with `is-`)
- Utility classes: `.full-width`, `.fade-up`, `.rte` (lowercase, descriptive)

**CSS Variables (Custom Properties):**
- Global brand colors: `--color-{name}` (e.g., `--color-olive`, `--color-tan`)
- Section-specific: `--{section-name}-{property}` (e.g., `--bs-heading`, `--hero-headline1-color`)
- Semantic: `--color-background`, `--color-foreground`
- Typography: `--font-heading`, `--font-label`, `--font-body`
- Layout: `--page-width`, `--page-margin`, `--nav-height`

**Liquid Variables & Attributes:**
- snake_case for assigned variables: `featured_collection`, `nav_link_count`, `left_count`
- snake_case for settings references: `section.settings.heading_text_1`
- camelCase for object property access where appropriate: `product.featured_image`, `product.selling_plan_groups`

**Schema Setting IDs:**
- snake_case: `logo_text_1`, `background_color`, `heading_text_1`, `min_height`
- Color settings: `{element}_color` (e.g., `background_color`, `link_color`, `badge_bg_color`)
- Boolean toggles: `show_{feature}` (e.g., `show_badge`, `show_payment_button`, `show_reviews_tab`)
- Text content: `{element}_text` (e.g., `eyebrow_text`, `heading_text_1`, `cart_label`)

**Data Attributes:**
- kebab-case: `data-delay`, `data-option`, `data-value`, `data-action`, `data-src`, `data-index`
- Used for JavaScript selectors and progressive enhancement

## Code Style

**Formatting:**
- No automated formatter configured; rely on manual consistency
- Liquid whitespace trimming:
  - Use `{%- -%}` to remove surrounding whitespace in control tags
  - Use `{{- -}}` to remove surrounding whitespace in output tags
  - Applied selectively to avoid extra blank lines
- Example from header.liquid:
  ```liquid
  {%- assign nav_links     = section.settings.menu.links -%}
  {%- assign nav_link_count = nav_links.size -%}
  ```

**Linting:**
- No ESLint or linting tool configured
- Manual code review expected

**HTML:**
- 2-space indentation (de facto standard in Shopify themes)
- Semantic HTML5 elements: `<header>`, `<footer>`, `<nav>`, `<article>`, `<main>`
- ARIA attributes for accessibility: `aria-label`, `aria-hidden`, `aria-expanded`, `aria-selected`, `aria-controls`
- Comments for major sections: `{%- comment -%} Mobile navigation drawer {%- endcomment -%}`

## Import Organization

**Not Applicable:** This is a Shopify Liquid theme, not a JavaScript/TypeScript codebase.

**Render Order in Layouts:**
- `layout/theme.liquid` order:
  1. Metadata and fonts (head)
  2. Critical CSS
  3. Meta tags
  4. Header group (sections 'header-group')
  5. Main content (content_for_layout)
  6. Footer group (sections 'footer-group')
  7. Global scripts (scroll-animations)

**Section Composition:**
- Group sections (header-group, footer-group) nest multiple sections
- Sections render blocks via `{% content_for 'blocks' %}`
- Snippets rendered with `{% render 'filename' %}` with parameters passed as key-value

## Error Handling

**Patterns:**
- Null/empty checks using Liquid conditions:
  ```liquid
  {%- if featured_collection == empty or featured_collection == nil -%}
    {%- assign featured_collection = collections.frontpage -%}
  {%- endif -%}
  ```
- Fallback assignments:
  ```liquid
  {%- assign product_limit = section.settings.product_limit | default: 6 -%}
  {%- assign og_description = page_description | default: shop.description | default: shop.name -%}
  ```
- No explicit error handling; Shopify renders nothing for nil/missing values
- Availability checks for product variants:
  ```liquid
  {%- unless current_variant.available -%}
    <button ... disabled>{{ section.settings.sold_out_text }}</button>
  {%- endunless -%}
  ```

**Accessibility:**
- Always provide fallback text for images: `alt: product.featured_image.alt | default: product.title`
- Escape user-generated content: `{{ product.title | escape }}`
- Use `aria-label` on icon-only buttons: `aria-label="Open menu"`
- Use `aria-live="polite"` for dynamic updates: `<span aria-live="polite">{{ cart.item_count }}</span>`

## Logging

**Framework:** No dedicated logging framework

**Patterns:**
- No console.log or debugging in production code
- Liquid has no native logging; errors surfaced in Shopify theme dev environment
- Comments used for documentation instead of debug statements
- JavaScript in `{% javascript %}` tags uses vanilla DOM APIs without logging

## Comments

**When to Comment:**
- Major section headers with visual markers:
  ```liquid
  {%- comment -%} === GALLERY === {%- endcomment -%}
  {%- comment -%} Left zone: Desktop=nav links, Mobile=hamburger+search {%- endcomment -%}
  ```
- Complex logic or non-obvious conditionals:
  ```liquid
  {%- comment -%}
    Split nav evenly around the logo.
    Odd totals get the extra link on the left.
  {%- endcomment -%}
  ```
- Feature flags and special conditions:
  ```liquid
  {%- comment -%}
    Basic membership product — show community selector.
    Tag your product with the value in section.settings.basic_membership_tag
    (default: "basic-membership") to enable this dropdown.
  {%- endcomment -%}
  ```

**LiquidDoc/DocBlocks:**
- All snippets must have `{% doc %}` header:
  ```liquid
  {% doc %}
    Renders a responsive image that might be wrapped in a link.

    @param {image} image - The image to be rendered
    @param {string} [url] - An optional destination URL for the image

    @example
    {% render 'image', image: product.featured_image %}
  {% enddoc %}
  ```
- All blocks must have `{% doc %}` header (especially static-rendered blocks):
  ```liquid
  {% doc %}
    Renders a text block.

    @example
    {% content_for 'block', type: 'text', id: 'text' %}
  {% enddoc %}
  ```
- No docblocks required for sections (schema provides documentation)

## Function Design

**Size:** No strict rules; prefer single responsibility

**Parameters (Snippet Renders):**
- Pass only required and optional parameters:
  ```liquid
  {% render 'image', image: product.featured_image, url: product.url, width: 1200, height: 800, crop: 'center' %}
  ```
- Optional parameters indicated in doc with square brackets: `[param]`

**Return Values:** Snippets output HTML directly; no return values

## Module Design

**Exports:**
- Snippets exported via `{% render 'snippet-name' %}`
- Sections defined by file presence; no explicit export syntax
- Blocks defined by file presence in `/blocks/` directory

**Barrel Files:**
- Not applicable to Liquid

**Section Schema Structure:**
- Always include `"name"` key (translatable with `t:` prefix)
- Always include `"settings"` array for full customization
- Optional `"blocks"` array for nested content
- Include `"presets"` with default configurations
- Example structure:
  ```json
  {
    "name": "t:section_name",
    "settings": [
      { "type": "header", "content": "t:labels.section" },
      { "type": "text", "id": "heading_text", "label": "t:labels.text" }
    ],
    "presets": [
      { "name": "t:section_name" }
    ]
  }
  ```

## CSS Architecture

**Organization:**
- Global styles in `assets/critical.css` (loaded on every page)
- Component-specific styles in `{% stylesheet %}` tags (scoped to section/block/snippet)
- Dynamic colors via `{% style %}` tag using CSS variables:
  ```liquid
  {% style %}
    #shopify-section-{{ section.id }} .hero {
      --hero-headline1-color: {{ section.settings.headline1_color }};
    }
  {% endstyle %}
  ```

**Responsive Design:**
- Mobile-first breakpoint: `@media (max-width: 1024px)` for desktop/mobile split
- Tablet breakpoint: `@media (max-width: 768px)` for smaller adjustments
- Examples:
  - Header: hides desktop nav at 1024px, shows mobile nav
  - Product card: adjusts image aspect ratio at 768px

**Animations:**
- Global fade-up on scroll: `.fade-up { opacity: 0; transform: translateY(28px); }`
- Driven by global IntersectionObserver in `scroll-animations.liquid`
- Stagger via `data-delay` attribute: `data-delay="1"` through `"6"` (100ms increments)
- Transitions used throughout: `.2s ease`, `.25s ease`, `.35s ease` for consistency

## Translation Keys

**Pattern:**
- Hierarchical with dot notation: `section.featured_collection.title`
- Always use `{{ 'key' | t }}` filter for user-facing text
- No hardcoded English copy
- Optional translations via square brackets: `{{ 'label' | t: min: value, max: value }}`

**Examples from codebase:**
- `t:general.text` (generic text block)
- `t:labels.text` (setting label)
- `t:options.text_style.title` (select option)
- `{{ 'general.newsletter.success' | t }}` renders "You're in. Welcome to the movement."

---

*Convention analysis: 2025-02-14*
