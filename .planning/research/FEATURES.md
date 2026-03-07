# Feature Research

**Domain:** Shopify theme drag-and-drop block toolkit + About Us page
**Researched:** 2026-03-06
**Confidence:** MEDIUM (official Shopify docs + multiple verified sources; some findings from WebSearch only)

---

## Feature Landscape

### About Us Page

#### Table Stakes (Merchants Expect These)

Features merchants assume exist. Missing these = page feels like a placeholder.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Brand origin story section | 31% of shoppers consider About Us essential for purchase decisions — they want to know who they're buying from | LOW | Full-width section: heading, rich text body, optional image. Schema: heading text, body text (richtext), background image picker, background color |
| Mission/ethos statement | Tactical brands especially need this — "why we exist" differentiates from generic gear stores | LOW | Eyebrow label + headline + supporting text. For Ruck On: "Carry the Weight" philosophy center stage |
| Team/people section with photos | Builds human connection and trust. Customers want faces, not just a logo | MEDIUM | Repeating blocks: each block has image picker, name text, title text, bio textarea. Grid layout of cards |
| All images via editor (no hardcoded assets) | Every merchant expectation in Shopify — if it can't be changed in the editor, it may as well not exist | LOW | All `image_picker` schema settings, placeholder SVG fallbacks when no image is set |
| Call-to-action buttons linking to shop/collections | Page should convert — not just inform. Users expect at least one CTA | LOW | Standard `.btn .btn--primary` with link and label settings |
| Mobile-responsive layout | Mandatory — 60%+ of Shopify traffic is mobile | LOW | Stacked layout on mobile for all sections; team grid goes 1-col on smallest breakpoints |

#### Differentiators (Competitive Advantage)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Tactical/military brand credibility block | Veteran-owned messaging, field-tested claims, "not built in a boardroom" copy sections — what the best tactical brands use to build authority | LOW | A dedicated "credibility bar" block: icon + short stat/claim, repeating. E.g. "Founded by veterans", "Field tested gear" |
| Values/ethos grid with icons | Turns abstract brand values into scannable, visual content. Better than a wall of text | MEDIUM | Multi-column block (see block section below) used for 3-4 values with icons, heading, and short description per value |
| Video embed option | Motion content outperforms static for brand storytelling — founders talking camera > static page | MEDIUM | Optional video URL setting (YouTube/Vimeo) with poster image fallback. External video URL Liquid filter available |
| Press/media mention logos bar | Social proof for brands with press coverage — "As seen in" strip | LOW | Repeating image blocks in a horizontal strip; each block = logo image + optional URL |
| Scroll-triggered fade-up animations | Brand consistency: existing Ruck On sections all use fade-up animations. Not having it on About Us would feel disconnected | LOW | Already built in `snippets/scroll-animations.liquid` — just add `class="fade-up"` to elements |

#### Anti-Features (Do NOT Build)

| Anti-Feature | Why Avoid | Alternative |
|--------------|-----------|-------------|
| Hardcoded brand copy in Liquid output | Violates the project's core constraint. Any hardcoded text breaks editor editing and fails Shopify Theme Store submission requirements | Use `section.settings.*` for ALL copy. Never write literal brand text in `.liquid` output |
| Tab-switching "Our Story / Our Mission / Our Team" single-section design | Tabs hide content, reduce scannability, and add JS complexity with no conversion benefit | Separate scrollable sections — merchants can reorder them in the editor |
| Embedded Google Map on About Us | Adds GDPR complexity, performance weight, and rarely drives conversions for a tactical gear brand | Link to a contact page if location info is needed |
| Social media feed embeds | Third-party scripts, CORS issues, performance hit, frequent API breakage | Static imagery with a manual "Follow us" CTA link |
| Animated counter numbers ("10,000+ customers") | Requires JS intersection observer, adds complexity, and counts break on slow connections | Use the existing `stats-bar.liquid` section instead — it already exists and fits the brand |

---

### Drag-and-Drop Block Toolkit

#### Table Stakes (Merchants Expect These)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Image + text block: image left or right toggle | The most basic layout control — every page builder has this. Missing it feels broken | LOW | Schema: `select` setting with values `image-left` / `image-right`. CSS flex-direction swap |
| Image + text block: image size control | Merchants need to balance visual weight between image and text | LOW | Schema: `select` with 1/3, 1/2, 2/3 width options. CSS custom property drives the column split |
| Image + text block: heading, body text, optional button | Standard content structure for any marketing section | LOW | Separate schema settings: heading text, body richtext, button label + URL |
| Multi-column block: 2 or 3 column selection | Standard layout for values, features, specs — any "things in a row" content | LOW | Schema: `select` — 2 columns or 3 columns. Mobile always stacks to 1 column |
| Multi-column block: column icon or image (optional) | Merchants need visual anchors above column text — pure text columns feel sparse | LOW | Per-column blocks: optional `image_picker` setting. If empty, skip image rendering |
| Multi-column block: column heading + body text | Core content for each column | LOW | Per-column block settings: heading text, body textarea |
| Testimonial block: quote text | The actual review content — no quote = no testimonial | LOW | `textarea` setting for quote body |
| Testimonial block: reviewer name | Attribution is required — anonymous quotes have no credibility | LOW | `text` setting for reviewer name |
| Testimonial block: star rating (1-5) | Users expect stars. A testimonial without stars looks unpolished | LOW | `range` setting min 1 max 5, render as filled/empty star SVGs in Liquid |
| Collection grid block: collection picker | Merchants need to choose which collection to show | LOW | `collection` setting type in schema — native Shopify picker |
| Collection grid block: column count | Merchants think in columns — "show 4 products wide" is a natural mental model | LOW | Schema: `select` 2, 3, or 4 columns. CSS grid columns |
| Collection page: column count setting | Dawn has this — merchants expect it everywhere | LOW | Add to existing `collection.liquid` schema: `columns_desktop` (2-4) and `columns_mobile` (1-2) |
| Collection page: image ratio setting | The most-requested collection page setting in Shopify community — square vs portrait vs adapt | LOW | Schema: `select` with values `adapt`, `portrait`, `square`. CSS `aspect-ratio` per value |
| All blocks: editor preview works (no blank state) | If a block renders as empty white space in the editor, merchants cannot configure it | LOW | Every block needs a visible default state or placeholder text in schema `default` values |

#### Differentiators (Competitive Advantage)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Image + text block: vertical alignment control | Text center-aligned next to a tall image looks bad. Alignment control (top / center / bottom) fixes this | LOW | CSS `align-items` via a select setting with 3 options |
| Image + text block: full-width option | For dramatic hero-style image+text panels — spans 100% viewport width | LOW | Additional CSS class toggle from a `checkbox` schema setting |
| Multi-column block: column heading tag (H2/H3) | SEO-aware merchants care about heading hierarchy. H2 vs H3 matters for page structure | LOW | `select` schema setting with H2/H3 options. Rendered in Liquid with `assign` |
| Testimonial block: reviewer photo | Photo + name + quote is dramatically more credible than name + quote alone. Tactical brand customers are skeptical | LOW | Optional `image_picker` setting per testimonial block. Render as small circular avatar |
| Testimonial block: layout preset (grid vs carousel) | A single testimonial looks sparse. Grid of 3 is the standard social proof pattern | MEDIUM | Section-level `select` setting: grid layout (2 or 3 cols) vs stacked. Carousel is high complexity — defer |
| Collection grid block: product count limit | Merchants need control over how many products show — 4, 6, 8 are the common choices | LOW | `range` setting min 2, max 12 (even numbers). `limit` filter in Liquid `for` loop |
| Collection grid block: show/hide "View all" link | Gives merchants an upsell path to the full collection | LOW | `checkbox` setting. Renders a link to the collection URL |
| Collection page: card style options | Card vs borderless vs minimal — different aesthetics for different merchants. This is the #1 request beyond columns and ratio | MEDIUM | Schema `select`: `card` (border + padding shadow), `standard` (no border), `minimal` (image only). CSS classes on `.product-card` |
| Collection page: show/hide secondary image on hover | Common feature in premium themes — product alt image swaps in on hover | MEDIUM | `checkbox` setting. Requires JS event listener on card hover — moderate JS complexity |

#### Anti-Features (Do NOT Build)

| Anti-Feature | Why Avoid | Alternative |
|--------------|-----------|-------------|
| Carousel/slider for testimonials | JS dependency, touch handling, accessibility issues, layout instability during load, break at unpredictable screen sizes — high complexity, low value for this brand | Static grid of 3 testimonials; if there are more, merchant adds a second block group |
| Carousel/slider for image+text alternating rows | Same problems as testimonial carousel. Alternating left/right image+text rows (image flips per block) achieve the same visual rhythm without JS | Use `forloop.index` parity to auto-alternate image side, or give merchant an explicit left/right toggle per block |
| Too-granular blocks (separate block for heading, separate for paragraph, separate for button) | This is the explicit anti-pattern in Shopify's official block best practices — it creates editor clutter, confuses merchants, and produces fragile layouts dependent on block order | Group heading + body + button into a single block. Keep block count low |
| Unlimited block nesting depth | Deep nesting (blocks inside blocks inside blocks) requires Theme blocks (`@theme` type) and creates unpredictable merchant behavior | Keep image+text as a standalone section, not a nested block system. Only use `content_for 'blocks'` at one level deep |
| Background video autoplay | Performance hit, mobile data cost, WCAG accessibility violations (motion sensitivity) | Use poster image with a play button that opens video in a modal or external link |
| Quick-add to cart on collection grid block | Requires cart drawer JS integration — significant scope increase; the existing cart page handles this flow | Link product cards to product pages as standard. Quick-add can be a future enhancement |
| Pagination on collection grid block embedded in a non-collection page | A "featured collection" block on About Us or a custom page cannot paginate — Shopify only paginates on collection templates | Show a fixed number of products with a "View all" link to the collection |

---

## Feature Dependencies

```
[About Us Page]
    depends on --> [image_picker settings] (exists in Shopify schema)
    depends on --> [scroll-animations.liquid snippet] (already exists)
    depends on --> [css-variables.liquid snippet] (already exists)
    depends on --> [.btn styles in critical.css] (already exists)

[Team Section]
    depends on --> [Repeating blocks pattern] (Shopify blocks with @schema)
    uses --> [image_picker per block]

[Multi-Column Block]
    depends on --> [{% content_for 'blocks' %}] (Shopify blocks architecture)
    enhances --> [About Us Page values section]
    enhances --> [Any custom page template]

[Testimonial Block]
    uses --> [Star rating range setting]
    uses --> [Optional image_picker for reviewer photo]
    independent --> [No external review app required for custom quotes]

[Collection Grid Block]
    depends on --> [collection setting type] (Shopify schema)
    depends on --> [.product-card styles in critical.css] (already exists)
    competes-with --> [existing best-sellers.liquid section] (overlapping purpose — use different design/context)

[Collection Page: column/ratio/card controls]
    modifies --> [existing collection.liquid section schema]
    depends on --> [.product-card CSS structure] (already in critical.css)
    must-not-break --> [existing pagination, sort bar, banner in collection.liquid]

[Image + Text Block]
    independent --> [standalone section OR embeddable block]
    enhances --> [About Us page brand story section]
    enhances --> [Any custom page]
```

### Dependency Notes

- **Collection Grid Block conflicts with Best Sellers Section:** Both show products in a grid. Differentiate by use: collection grid block is for custom pages/About Us, best-sellers is for homepage with specific "hero product" styling. They share `.product-card` CSS.
- **Testimonial Block is independent of review apps:** These are merchant-written pull quotes, not synced from a reviews platform. That integration is out of scope.
- **Multi-column Block requires `content_for 'blocks'`:** This means the block must be a theme block (`"type": "@theme"`) used inside a section that has `content_for 'blocks'`. Design the section wrapper to accept `@theme` blocks.
- **Collection page schema changes must not break existing behavior:** The existing `collection.liquid` has a working sort bar, banner, and pagination. Adding column/ratio/card settings is additive — new settings must have safe defaults that reproduce current behavior.

---

## MVP Definition

This milestone is well-scoped. MVP = all active requirements from `PROJECT.md`. No deferral needed.

### Launch With (v1)

- [x] About Us: brand origin story section — text + optional image, full editor control
- [x] About Us: mission/ethos section — headline + supporting text, brand-forward design
- [x] About Us: team/people section — repeating person blocks with photo, name, title, bio
- [x] Image + text block — image left/right, image width 1/3 / 1/2 / 2/3, heading + body + optional button
- [x] Multi-column block — 2 or 3 columns, per-column icon/image + heading + text
- [x] Testimonial block — quote + name + 1-5 star rating, optional photo, grid layout
- [x] Collection grid block — collection picker, column count (2/3/4), product limit, view-all link
- [x] Collection page: `columns_desktop`, `columns_mobile`, `image_ratio`, `card_style` settings

### Add After Validation (v1.x)

- [ ] Testimonial block: carousel layout — only if merchants explicitly ask for it; current grid is sufficient
- [ ] Image + text block: full-width mode — easy to add but not blocking launch
- [ ] Collection page: secondary image on hover — nice-to-have, adds JS complexity
- [ ] Video embed option in brand story section — high impact but not required for a text-first storytelling launch

### Future Consideration (v2+)

- [ ] Press/media mention logos bar — only relevant once Ruck On has press coverage to display
- [ ] Integration with Shopify Product Reviews app for testimonial block — requires app block architecture
- [ ] Credibility bar with icons (veteran-owned, field-tested) — can be built with multi-column block at launch

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| About Us: origin story section | HIGH | LOW | P1 |
| About Us: mission/ethos section | HIGH | LOW | P1 |
| About Us: team section (repeating blocks) | HIGH | MEDIUM | P1 |
| Image + text block (left/right, width, CTA) | HIGH | LOW | P1 |
| Multi-column block (2-3 cols, icon + heading + text) | HIGH | LOW | P1 |
| Testimonial block (quote + name + stars) | HIGH | LOW | P1 |
| Collection grid block (collection picker + columns) | MEDIUM | LOW | P1 |
| Collection page: columns + image ratio + card style | HIGH | LOW | P1 |
| Testimonial block: reviewer photo | MEDIUM | LOW | P2 |
| Image + text block: vertical alignment | MEDIUM | LOW | P2 |
| Collection page: secondary hover image | MEDIUM | MEDIUM | P2 |
| Testimonial block: carousel layout | LOW | HIGH | P3 |
| Video embed in brand story | MEDIUM | MEDIUM | P3 |
| Press/media logos bar | LOW | LOW | P3 |

**Priority key:**
- P1: Must have for this milestone — builds what PROJECT.md defines as "Active"
- P2: Add in same development pass if time allows — low cost, good merchant value
- P3: Defer — either low value, high cost, or waiting on external factors

---

## Competitor Feature Analysis

These are established Shopify themes whose block and page patterns set merchant expectations.

| Feature | Dawn (Shopify default) | Debutify | Our Approach |
|---------|------------------------|----------|--------------|
| Image + text image position | Left / Right toggle only | Left / Right only | Left / Right + width control (1/3 / 1/2 / 2/3) |
| Multi-column block types | Text-only blocks | Basic text | Icon/image + heading + text per column |
| Collection page: columns | `columns_desktop` (2-4) + `columns_mobile` (1-2) | Present | Match Dawn baseline; same setting names |
| Collection page: image ratio | `adapt`, `portrait`, `square` | Present | Match Dawn baseline; same three options |
| Collection page: card style | `standard` | Present | Add `minimal` option to fit Ruck On's clean aesthetic |
| Testimonial block | Not native in Dawn | Basic | Quote + name + 1-5 star range + optional photo |
| Team section | Not native | App-required | Native blocks: photo + name + title + bio |
| About Us template | Generic page template only | Generic | Dedicated sections with brand-appropriate design and tactical aesthetics |

---

## Sources

- [Shopify - How to Write an About Us Page (2026)](https://www.shopify.com/blog/how-to-write-an-about-us-page) — MEDIUM confidence (Shopify official blog)
- [Shopify Developer Docs - Blocks Best Practices](https://shopify.dev/docs/storefronts/themes/architecture/blocks/best-practices) — HIGH confidence (official docs)
- [Shopify Developer Docs - Building with sections and blocks](https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks) — HIGH confidence (official docs)
- [Shopify - Meet the Team Page Examples (2026)](https://www.shopify.com/blog/meet-the-team-page-examples) — MEDIUM confidence (Shopify official blog)
- [Posstack - Dawn Image With Text Enhancements](https://posstack.com/blog/shopify-dawn-theme-image-with-text-enhancements) — LOW confidence (third-party analysis, verified against Dawn source)
- [Posstack - Dawn Multicolumn Enhancements](https://posstack.com/blog/shopify-dawn-theme-multicolumn-enhancement) — LOW confidence (third-party, useful for gap analysis)
- [Dawn source - main-collection-product-grid.liquid](https://github.com/Shopify/dawn/blob/main/sections/main-collection-product-grid.liquid) — HIGH confidence (official source)
- [Debutify Help - Image With Text Section](https://help.debutify.com/en/articles/5720012-how-to-customize-and-use-the-image-with-text-section) — LOW confidence (competitor docs, single source)
- Tactical brand About Us analysis: Propper, Rothco, Vertx, FirstTactical websites reviewed — LOW confidence (observation only)

---

*Feature research for: Shopify custom theme — About Us page + drag-and-drop block toolkit*
*Researched: 2026-03-06*
