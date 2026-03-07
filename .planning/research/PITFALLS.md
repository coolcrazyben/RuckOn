# Pitfalls Research

**Domain:** Shopify theme development — reusable blocks, About Us page, collection editor controls
**Researched:** 2026-03-06
**Confidence:** HIGH (all findings verified against official Shopify documentation + community evidence)

---

## Critical Pitfalls

### Pitfall 1: Missing `{{ block.shopify_attributes }}` on Block Root Elements

**What goes wrong:**
New blocks render correctly on the storefront but are invisible to the theme editor. Merchants cannot click to select them, see their settings panel, or get the blue highlight border. The editor appears broken even though the storefront is fine.

**Why it happens:**
Sections get their editor-detection attributes injected automatically by Shopify's wrapper element. Blocks do NOT — developers must add `{{ block.shopify_attributes }}` manually to the block's outermost HTML element. This asymmetry is a common trap when moving from section-based development to block-based.

**How to avoid:**
Every block's outermost element must include `{{ block.shopify_attributes }}`. This is non-negotiable:
```liquid
<div class="image-text-block" {{ block.shopify_attributes }}>
  ...
</div>
```
The existing `blocks/group.liquid` and `blocks/text.liquid` already do this correctly — use them as the pattern for every new block.

**Warning signs:**
- Clicking on block content in the editor does nothing (no selection highlight)
- Block settings panel doesn't appear in the editor sidebar
- Editor console shows no block-related Shopify events firing

**Phase to address:**
Any phase that creates new blocks. Make it the first line added after the opening tag in every block file.

---

### Pitfall 2: No Preset Defined — Block Doesn't Appear in "Add Block" Picker

**What goes wrong:**
The block file exists in `/blocks/`, but merchants cannot add it via the theme editor "Add block" button. The block is simply absent from the picker.

**Why it happens:**
Shopify requires at least one `preset` entry in the block schema for the block to appear in the editor's block picker. Without presets, the block can only be added by editing JSON template files manually — something merchants will never do.

**How to avoid:**
Every new block schema must include a `presets` array with at least one entry. The existing `blocks/text.liquid` is a correct example:
```json
"presets": [{ "name": "t:general.text" }]
```
For blocks with multiple configurations (like image+text with image-left vs image-right), define multiple presets so merchants get helpful starting points.

**Warning signs:**
- Block file exists but isn't visible in the editor "Add block" picker
- No errors thrown — it silently doesn't appear

**Phase to address:**
Every block creation phase. Add presets last, after schema settings are finalized, so preset defaults match valid setting values.

---

### Pitfall 3: Liquid Code Inside `{% javascript %}` or `{% stylesheet %}` Tags

**What goes wrong:**
Styles or scripts silently fail. CSS variables that need block-specific values (colors, dimensions from settings) don't apply. JS that needs to reference section IDs breaks without clear errors.

**Why it happens:**
Shopify processes `{% javascript %}` and `{% stylesheet %}` content as static bundles — Liquid is NOT rendered inside them. Developers coming from other templating systems assume they can interpolate values from settings.

**How to avoid:**
Use `{% style %}` (not `{% stylesheet %}`) for dynamic values that require Liquid. Use CSS custom properties passed via inline `style=""` attributes, then reference them in `{% stylesheet %}`. The project already does this correctly in `sections/collection.liquid`:
```liquid
{% style %}
  #shopify-section-{{ section.id }} .collection-page {
    --col-cols-desktop: {{ section.settings.columns_desktop }};
  }
{% endstyle %}
```
All new blocks must follow the same pattern: dynamic values via `{% style %}` or inline `style` attributes, structural CSS via `{% stylesheet %}`.

**Warning signs:**
- Settings changes in editor have no visual effect
- CSS property values appear as literal Liquid strings (e.g., `{{ block.settings.color }}`) in DevTools
- JS errors referencing undefined variables that were supposed to come from Liquid

**Phase to address:**
Foundational — establish pattern before building any blocks. Every block that has color, size, or layout settings must use the `{% style %}` pattern.

---

### Pitfall 4: JavaScript Not Re-Initializing After Editor Section Reload

**What goes wrong:**
JavaScript runs on page load but breaks the moment a merchant edits a setting in the theme editor. Carousels, interactive galleries, tab widgets, and scroll animations stop working in the editor preview. Merchants report "it broke when I changed X setting."

**Why it happens:**
When a merchant changes a section setting, Shopify re-renders that section and fires `shopify:section:load`. Any `{% javascript %}` bundled code that ran on `DOMContentLoaded` will NOT re-run automatically. Interactive elements in the re-rendered markup have no event listeners.

**How to avoid:**
Wrap initialization logic in a function callable from both initial load and `shopify:section:load`. For blocks, also handle `shopify:block:select`:
```javascript
(function() {
  function initBlock(container) {
    // initialization targeting container
  }

  // Initial load
  document.querySelectorAll('[data-my-block]').forEach(initBlock);

  // Editor re-renders
  document.addEventListener('shopify:section:load', function(e) {
    e.target.querySelectorAll('[data-my-block]').forEach(initBlock);
  });
})();
```
The collection page's sort select is simple enough to survive a re-render without this, but any block with interactive state (galleries, testimonial sliders, tab switching) must implement this pattern.

**Warning signs:**
- Interactive elements work on page load but stop working after any editor change
- Animations or scroll effects don't trigger in editor preview after settings changes
- Merchant reports "worked during setup but broke after customizing"

**Phase to address:**
Any phase building blocks with JavaScript (image+text layout toggles, testimonial interactions, collection grid JS). Not needed for purely CSS-driven blocks.

---

### Pitfall 5: Mixing Section Blocks and Theme Blocks in the Same Section

**What goes wrong:**
Schema validation errors, editor behavior breaks, blocks don't render or nest as expected.

**Why it happens:**
Shopify enforces that a section accepts either section blocks OR theme blocks — not both. Section blocks are the old inline approach (defined within the section's `blocks` array in schema). Theme blocks live in the `/blocks/` directory and are referenced via `{ "type": "@theme" }`. The distinction is easy to confuse when extending existing sections.

**How to avoid:**
For all new sections in this milestone, use theme blocks exclusively: define `"blocks": [{ "type": "@theme" }]` in the schema and render with `{% content_for 'blocks' %}`. Do not mix inline block type definitions with `@theme`. The About Us sections and any section hosting the reusable blocks must all use the theme block approach.

**Warning signs:**
- Schema validation errors when uploading to Shopify
- Blocks appear in schema but don't render in the editor
- `content_for 'blocks'` renders nothing despite blocks being added

**Phase to address:**
About Us page sections and any section designed to host the new reusable blocks.

---

### Pitfall 6: Block CSS Using Generic Class Names That Conflict Across Blocks

**What goes wrong:**
Styles from one block bleed into another. An `.image-text` class defined in `blocks/image-text.liquid` overrides the same class used differently in a snippet or another section. Layout breaks on pages with multiple block types.

**Why it happens:**
Shopify bundles all `{% stylesheet %}` content from blocks into a single `block-styles.css` file. Class names from different blocks live in the same global CSS scope. Developers forget this and use short, generic class names.

**How to avoid:**
Namespace all block CSS classes with the block name as a prefix:
```css
/* Good */
.block-image-text { ... }
.block-image-text__image { ... }
.block-image-text__content { ... }

/* Bad — will conflict */
.image-wrap { ... }
.content { ... }
```
Alternatively, use CSS custom properties scoped via the block's `shopify_attributes` data attribute if needed for per-instance differentiation.

**Warning signs:**
- Adding a second block type causes layout shifts in the first block type
- Styles inconsistently applied across pages with different block combinations
- DevTools shows the same class name with conflicting rules from different source files

**Phase to address:**
Before writing any block CSS. Establish the naming convention in the first block, enforce it in all subsequent blocks.

---

## Moderate Pitfalls

### Pitfall 7: `image_picker` Settings Without Placeholder Fallback

**What goes wrong:**
New blocks or About Us sections look broken in the editor before images are uploaded. Empty image containers show with no size, collapsing to 0px height. The block appears unusable before content is added.

**Why it happens:**
Developers check `{% if section.settings.image %}` and render nothing in the `else` branch, assuming images will always be present. But new pages start empty, and merchants need to see the block's structure to understand where to upload.

**How to avoid:**
Always provide a fallback using Shopify's `placeholder_svg_tag` filter. The `onboarding` Liquid variable pattern is also helpful:
```liquid
{% if block.settings.image %}
  {{ block.settings.image | image_url: width: 1200 | image_tag: loading: 'lazy', alt: block.settings.image.alt }}
{% else %}
  {{ 'lifestyle-1' | placeholder_svg_tag: 'block-image__placeholder' }}
{% endif %}
```
For About Us team member photos, use `'image'` as the placeholder type. For product collection grids, use `'product-1'` (already used in `sections/collection.liquid`).

**Warning signs:**
- Block appears collapsed (zero height) before image is set
- Merchant asks "how do I add an image? I can't see where to click"
- New section in editor looks like an empty div

**Phase to address:**
About Us page phase (team photos, hero images) and collection grid block phase.

---

### Pitfall 8: Hardcoded Text in Liquid Output Instead of `t` Filter

**What goes wrong:**
Text strings in new blocks and sections are not editable through the Shopify Language Editor. Merchants who sell internationally or want to customize labels cannot change text without editing code. The project's own standard (per CLAUDE.md) is violated.

**Why it happens:**
Developers write `<span>Learn More</span>` instead of `{{ 'blocks.image_text.cta_label' | t }}` + the corresponding locale key. It's faster during development and easy to forget.

**How to avoid:**
Every user-facing string must use the `t` filter and have a corresponding key in `locales/en.default.json`. Schema label references (for the editor sidebar) use `t:` prefix in the schema JSON. The project already has this pattern in `blocks/group.liquid` (`t:labels.alignment`, etc.) — replicate this for all new blocks.

For new blocks, add keys under a `blocks.` namespace:
```json
{
  "blocks": {
    "image_text": {
      "heading_placeholder": "Our Story",
      "cta_label": "Learn more"
    }
  }
}
```

**Warning signs:**
- Merchant can't find text to edit in the Language Editor
- Static English strings visible in rendered HTML even when store language is changed

**Phase to address:**
All phases. Add locale keys alongside Liquid output in the same commit to prevent drift.

---

### Pitfall 9: Over-Granular Block Design Creates Merchant Confusion

**What goes wrong:**
Merchants are overwhelmed by too many block types in the picker. Simple pages require adding 6-8 blocks to achieve what should be one component. The "Add block" list becomes unwieldy.

**Why it happens:**
Developers mirror their mental model of components ("I'll make a heading block, a body text block, a button block, an image block...") rather than thinking in merchant-facing units of content. Each block that can stand alone as a useful component.

**How to avoid:**
For the About Us page, think in story units: an "Image + Text" block is one block, not four separate blocks. A "Team Member" block contains photo + name + title + bio — not separate blocks for each field. The existing `blocks/group.liquid` is a layout primitive; new content blocks should be semantic and self-contained.

Per Shopify's official guidance: "group related attributes like author, date, and comments into a single block rather than introducing them as three separate blocks."

**Warning signs:**
- The block picker has more than 8-10 options for a single page type
- Merchants need to add 3+ blocks to produce one visible content unit
- Block count for a typical About Us page exceeds 20

**Phase to address:**
Design phase before implementation. Plan block types on paper first.

---

### Pitfall 10: Collection Grid Block Triggering Full Page Reloads for Sort Changes

**What goes wrong:**
The reusable collection grid block placed on non-collection pages (like an About Us or landing page) has no access to `collection.sort_options` or `collection.sort_by` Liquid objects. Attempting to replicate the sort bar behavior breaks silently or causes Liquid errors.

**Why it happens:**
The `collection` object is only available on collection page templates. On a generic `/pages/about-us` template, it doesn't exist. Developers copy the sort bar from `sections/collection.liquid` into a block without realizing the Liquid objects won't resolve.

**How to avoid:**
The reusable collection grid block should accept a `collection` setting (type `collection`) from the editor, then access products via `collections[block.settings.collection_handle].products`. Sort controls must be omitted or replaced with a static featured-product display since dynamic sort requires URL parameters that only work on collection templates.

**Warning signs:**
- Liquid error: "undefined method 'sort_options' for NilClass" on non-collection pages
- Sort dropdown appears but has no options
- `collection.products` returns empty on a page template

**Phase to address:**
Collection grid block phase. Clarify in the block schema description that sort is not supported when used outside collection templates.

---

### Pitfall 11: `visible_if` Used on Unsupported Setting Types

**What goes wrong:**
Schema validation errors or settings that silently ignore their `visible_if` condition, always showing regardless of the condition's value.

**Why it happens:**
`visible_if` is a beta feature with a limited list of supported setting types. The setting types `collection`, `collection_list`, `product`, `product_list`, `page`, `article`, `blog`, and `color_scheme_group` do NOT support `visible_if`. Developers add conditions to these without checking the constraint.

**How to avoid:**
Only use `visible_if` on basic setting types: `text`, `textarea`, `richtext`, `inline_richtext`, `checkbox`, `select`, `radio`, `range`, `color`, `image_picker`, `url`, `video_url`, `font_picker`, `html`, `liquid`, and `number`. The `group.liquid` block uses it correctly on `alignment` (a `select` type). Do not attempt to conditionally show/hide a `collection` picker using `visible_if`.

**Warning signs:**
- Schema upload succeeds but `visible_if` condition appears to be ignored
- Setting always shows even when condition should hide it
- Shopify CLI validation error mentioning unsupported conditional setting type

**Phase to address:**
Any phase adding conditional schema settings. Check the setting type before adding `visible_if`.

---

## Minor Pitfalls

### Pitfall 12: Duplicate Setting IDs Within a Block

**What goes wrong:**
Shopify throws a hard error during theme upload or CLI push. Development stops until the duplicate is resolved.

**Why it happens:**
Copy-pasting block schema between blocks and forgetting to rename IDs. Especially common when creating the multi-column text block from a copy of the image+text block.

**How to avoid:**
Each setting `id` must be unique within a block. When copying schema, immediately audit all IDs. Schema IDs like `heading`, `body`, `cta_label` are safe reuse across different blocks (they're scoped to the block), but within one block they must be unique.

**Warning signs:**
- Shopify CLI push error: "duplicate setting ID"
- Theme upload fails with schema validation error

**Phase to address:**
Any block creation. Caught immediately by Shopify's validator — low risk if caught during development.

---

### Pitfall 13: Block Has More Than One `{% schema %}`, `{% stylesheet %}`, or `{% javascript %}` Tag

**What goes wrong:**
Shopify throws a syntax error. The block file becomes unpublishable.

**Why it happens:**
Developers add a second `{% stylesheet %}` to handle a media query addition they forgot to include in the first one, or copy a block and leave a duplicate tag.

**How to avoid:**
One `{% schema %}` tag, one `{% stylesheet %}` tag, and one `{% javascript %}` tag per file — always. Group all CSS into the single `{% stylesheet %}` block, including media queries.

**Warning signs:**
- Shopify CLI error: "only one {% stylesheet %} tag allowed per file"
- File saves but theme editor shows broken rendering

**Phase to address:**
All phases. Trivial to avoid if you lint with Shopify CLI before pushing.

---

### Pitfall 14: Image Tag Missing Width/Height Causing Layout Shift (CLS)

**What goes wrong:**
Images load and cause surrounding content to jump downward (Cumulative Layout Shift). Core Web Vitals score degrades. This affects SEO and the merchant's Shopify speed score.

**Why it happens:**
Rendering images without explicit `width` and `height` attributes means the browser doesn't reserve space before the image downloads. The browser sets height to 0px until the image loads.

**How to avoid:**
Use Shopify's `image_tag` filter — it automatically adds `width` and `height` attributes from the image object's dimensions. The existing `sections/collection.liquid` does this correctly:
```liquid
{{ product.featured_image
   | image_url: width: 700
   | image_tag: loading: 'lazy', sizes: '...' }}
```
For the About Us team section where images are cropped to specific ratios, use CSS `aspect-ratio` as an additional guarantee.

**Warning signs:**
- Lighthouse CLS score is non-zero
- Page content jumps when scrolling during image load
- Shopify Theme Check `ImgWidthAndHeight` check fails

**Phase to address:**
About Us page (team photos especially) and collection grid block.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoded text strings | Faster to write | Can't be edited in Language Editor; blocks merchant translation | Never — always use `t` filter |
| Generic CSS class names in blocks | Less typing | Cross-block style conflicts once multiple blocks are active on same page | Never — always namespace with block name |
| No `placeholder_svg_tag` fallback for images | Simpler code | Block looks broken to merchants before images are uploaded | Never for production |
| Skipping `shopify:section:load` re-init | Simpler JS | Interactive elements break in editor after any setting change | Acceptable only for purely static blocks with zero JavaScript |
| Copying schema IDs without renaming | Faster copy-paste | Hard schema validation error on push | Never |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Shopify Language Editor | Using `section.settings.some_text` directly as label text | Use `t:` references in schema and `| t` filter in output |
| Theme editor JS events | Initializing JS once on `DOMContentLoaded` only | Also handle `shopify:section:load` for editor re-renders |
| Collection object in blocks | Accessing `collection.sort_options` in a block on a page template | Accept `collection` setting; only sort on collection templates |
| Image picker settings | Rendering nothing when setting is empty | Always provide `placeholder_svg_tag` fallback |
| Block nesting | Using section block inline definitions alongside `@theme` | Choose one system per section — cannot mix |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Liquid in `{% javascript %}` or `{% stylesheet %}` | Values don't apply; no error | Use `{% style %}` for dynamic values | Immediately on any page load |
| Images without lazy loading on About Us sections | Slow initial page load, high LCP | Use `loading: 'lazy'` for below-fold images; `loading: 'eager'` only for hero | Always (especially mobile) |
| Missing image dimensions (no width/height) | Layout shift (CLS) on every page load | Use `image_tag` filter which auto-adds dimensions | Always |
| Too many products per page on collection grid block | Long Liquid render time + large HTML | Cap at 24 products default; allow merchant to reduce | At 24+ products on slower stores |
| JS event listener stacking in editor | After 10+ editor changes, events fire multiple times | Guard initialization with a check or remove listeners before re-adding | In editor after repeated changes |

---

## UX Pitfalls

| Pitfall | User (Merchant) Impact | Better Approach |
|---------|------------------------|-----------------|
| Block has no preset | Block invisible in "Add block" picker — merchant can't use it | Always define at least one preset per block |
| Missing `shopify_attributes` on block element | Can't click to select block in editor | Add `{{ block.shopify_attributes }}` to every block's root element |
| Image block shows empty/collapsed before upload | Merchant can't see structure; thinks block is broken | Provide `placeholder_svg_tag` fallback |
| Too many block settings without grouping | Editor sidebar is overwhelming | Use `header` settings to group related settings with clear labels |
| About Us sections with no schema settings at all | Merchant must edit code for any text change | Every piece of text, every image, every color must be a schema setting |

---

## "Looks Done But Isn't" Checklist

- [ ] **Every new block:** Has `{{ block.shopify_attributes }}` on its outermost element — verify by clicking the block in the editor and checking for blue selection border
- [ ] **Every new block:** Has at least one `preset` in schema — verify by opening "Add block" picker in editor and confirming block appears
- [ ] **Every image setting:** Has a `placeholder_svg_tag` fallback — verify by previewing the block before uploading any images
- [ ] **Every string in Liquid output:** Uses `| t` filter and has a key in `locales/en.default.json` — verify by opening the Language Editor and searching for the text
- [ ] **Every block with JS:** Handles `shopify:section:load` event for re-initialization — verify by making a setting change in the editor and confirming interactive elements still work
- [ ] **Every block's CSS classes:** Are namespaced with the block name prefix — verify by checking DevTools on a page with multiple block types
- [ ] **Collection grid block:** Does not reference `collection.sort_options` or `collection.sort_by` — verify by previewing on a standard page template (not a collection page)

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Missing `shopify_attributes` discovered post-launch | LOW | Add one line to block's root element; re-push |
| Missing presets discovered post-launch | LOW | Add presets array to block schema; re-push; merchant can now use picker |
| Liquid in stylesheet (styles not applying) | LOW-MEDIUM | Move dynamic values to `{% style %}` tag; restructure CSS to use CSS custom properties |
| CSS class name conflicts causing cross-block style bleed | MEDIUM | Rename all CSS classes in one block to namespaced versions; test all pages with mixed blocks |
| Hardcoded text strings throughout many blocks | MEDIUM-HIGH | Extract all strings to locale keys; update all Liquid references; re-test Language Editor |
| JS not re-initializing in editor (discovered during QA) | LOW | Wrap init logic in function; add `shopify:section:load` listener |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Missing `shopify_attributes` | Every block creation phase | Click block in editor — blue border must appear |
| Missing presets | Every block creation phase | Open "Add block" picker — block must be visible |
| Liquid in stylesheet/JS tags | First block phase (establish pattern) | Inspect rendered CSS/JS source — no Liquid syntax should appear |
| JS not re-initializing in editor | Any phase adding interactive JS | Change a setting in editor — interactive elements must still work |
| Section vs. theme block mixing | About Us sections phase | Push to Shopify CLI with no schema errors |
| Generic CSS class names | First block phase (establish convention) | Review DevTools on page with all blocks active |
| Missing image placeholders | About Us page phase (team photos) | Preview block with no images set in editor |
| Hardcoded text strings | All phases | Open Language Editor — all strings must appear editable |
| Over-granular block design | Design phase before any coding | Count block types needed for a typical About Us page — should be under 6 |
| Collection block on page template | Collection grid block phase | Preview block on a `/pages/` template — no Liquid errors |
| `visible_if` on unsupported types | Schema settings phase | Push to Shopify CLI — no validation errors |
| Duplicate setting IDs | Every block creation phase | Shopify CLI push must succeed with zero schema errors |
| Image dimensions/CLS | About Us page phase | Lighthouse audit — CLS score must be under 0.1 |

---

## Sources

- [Shopify blocks best practices (official)](https://shopify.dev/docs/storefronts/themes/architecture/blocks/best-practices) — HIGH confidence
- [Building with sections and blocks (official)](https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks) — HIGH confidence
- [Block schema documentation (official)](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/schema) — HIGH confidence
- [JavaScript and stylesheet tags (official)](https://shopify.dev/docs/storefronts/themes/best-practices/javascript-and-stylesheet-tags) — HIGH confidence
- [Integrate sections and blocks with the theme editor (official)](https://shopify.dev/docs/storefronts/themes/best-practices/editor/integrate-sections-and-blocks) — HIGH confidence
- [Performance best practices (official)](https://shopify.dev/docs/storefronts/themes/best-practices/performance) — HIGH confidence
- [ImgWidthAndHeight Theme Check (official)](https://shopify.dev/docs/storefronts/themes/tools/theme-check/checks/img-width-and-height) — HIGH confidence
- [Input settings including `visible_if` (official)](https://shopify.dev/docs/storefronts/themes/architecture/settings) — HIGH confidence
- [CLS optimization for Shopify (official blog)](https://performance.shopify.com/blogs/blog/how-to-optimize-cumulative-layout-shift-cls-on-shopify-sites) — HIGH confidence
- [Shopify community: shopify:section:load event listener stacking](https://community.shopify.com/t/whats-up-with-shopifyload-stacking-event-listeners/572531) — MEDIUM confidence
- [Shopify community: section JS not re-running after edit](https://community.shopify.com/c/technical-q-a/section-doesn-t-run-javascript-after-edit/td-p/706460) — MEDIUM confidence
- [inline_richtext vs richtext (Shopify community)](https://community.shopify.dev/t/input-type-richtext-and-inline-richtext/28155) — MEDIUM confidence

---
*Pitfalls research for: Shopify theme — blocks, About Us page, collection editor controls*
*Researched: 2026-03-06*
