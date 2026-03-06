# Testing Patterns

**Analysis Date:** 2025-02-14

## Test Framework

**Status:** No automated testing framework configured.

**Approach:** Manual testing and Shopify theme editor live preview.

**Run Commands:**
```bash
# No test runner available
# Theme testing performed via:
# 1. Shopify CLI local development: `shopify theme dev`
# 2. Shopify theme editor live preview
# 3. Manual browser testing
```

## Test File Organization

**Location:** Not applicable — no test files in codebase.

**Testing Method:**
- Theme Editor Preview (Shopify Customize panel)
- Browser DevTools inspection
- Manual interaction testing in development environment

## Theme Testing Best Practices

While no automated test suite exists, the theme follows patterns suitable for manual validation:

**Schema Validation:**
- All sections have complete `{% schema %}` blocks with valid JSON
- Schema defines all customizable properties (colors, text, images, toggles)
- Required settings documented in schema `label` fields
- Optional settings have sensible defaults

**Liquid Template Testing:**
- Fallback assignments prevent nil/blank errors:
  ```liquid
  {%- assign featured_collection = collections[section.settings.collection] -%}
  {%- if featured_collection == empty or featured_collection == nil -%}
    {%- assign featured_collection = collections.frontpage -%}
  {%- endif -%}
  ```
- Conditional rendering checks before output:
  ```liquid
  {%- if section.settings.show_badge and product.type != blank -%}
    <span class="product-card__badge">{{ product.type }}</span>
  {%- endif -%}
  ```

**JavaScript Testing:**
- Vanilla JavaScript in `{% javascript %}` tags (no framework dependencies)
- Features tested via browser interaction:
  - Mobile navigation: hamburger toggle, backdrop click, escape key
  - Product variants: pill selection, price update, availability state
  - Scroll behavior: header shadow on scroll, fade-up animations

**Example JavaScript Pattern (Header Scroll Behavior):**
```javascript
(function () {
  var wrapper = document.getElementById('site-header-wrapper');
  if (wrapper) {
    window.addEventListener('scroll', function () {
      wrapper.classList.toggle('is-scrolled', window.scrollY > abHeight);
    }, { passive: true });
  }
})();
```
- Defensive: checks element exists before operating
- Uses passive listener for scroll performance
- Safe IIFE to avoid global scope pollution

**Example JavaScript Pattern (Mobile Navigation):**
```javascript
(function () {
  var nav = document.getElementById('mobile-nav');
  var toggle = document.getElementById('mobile-nav-toggle');
  if (!nav || !toggle) return;  // Early exit if elements missing

  function openNav() {
    nav.classList.add('is-open');
    nav.setAttribute('aria-hidden', 'false');
    toggle.setAttribute('aria-expanded', 'true');
  }

  toggle.addEventListener('click', openNav);
})();
```
- Checks for element existence before binding events
- Updates ARIA attributes for screen reader feedback
- Handles Escape key and backdrop click for accessibility

## Manual Test Coverage Areas

**Header Section (`sections/header.liquid`):**
- Desktop: nav links split around centered logo
- Tablet/Mobile: hamburger menu appears, nav links hidden
- Mobile nav: opens/closes on hamburger click, backdrop click, escape key
- Cart icon: displays count badge, updates on cart change
- Search icon: links to search page (desktop and mobile)
- Logo: clickable, links to home

**Product Page (`sections/product.liquid`):**
- Image gallery: main image updates on thumbnail click
- Variants: pills update selected state, price updates, URL params update
- Quantity stepper: +/- buttons adjust quantity input
- Tabs: tab buttons toggle panel visibility
- Add to cart: button state changes on variant availability
- Community fields: show/hide based on product tags

**Product Cards (`assets/critical.css`):**
- Tall card in best-sellers grid: media scales with row height on desktop
- Mobile: tall card collapses to standard aspect ratio
- Hover state: CTA text fades in, image scales slightly
- Prices: sale price highlighted, compare-at struck through

**Collection & Search:**
- Product grid loads correctly
- Pagination: current page highlighted, links navigate
- Sorting works (Shopify native sort_by filter)

**Scroll Animations:**
- `.fade-up` elements with `data-delay` stagger in view
- IntersectionObserver threshold at 12% (visible before enter viewport)
- Only fires once per element (observer.unobserve called)

## Accessibility Testing

**Requirements Met:**
- Semantic HTML: `<header>`, `<footer>`, `<nav>`, `<main>`, `<article>`
- ARIA landmarks: `role="banner"`, `role="contentinfo"`, `role="tablist"`, `role="tab"`
- ARIA labels: all icon-only buttons have `aria-label`
- ARIA state: `aria-expanded`, `aria-selected`, `aria-hidden`, `aria-current`
- Image alt text: all images have fallback alt (product.title or custom)
- Color contrast: tan (#e8d5b0) on black (#111009) meets WCAG AA
- Focus management: mobile nav closes and returns focus to toggle button
- Form labels: all inputs have associated `<label>` or `aria-label`

**Tested Manually:**
- Screen reader navigation (NVDA/JAWS on Windows, VoiceOver on Mac)
- Keyboard-only navigation (Tab through interactive elements)
- Focus indicators (buttons and links have visible focus state)
- Text scaling (content remains readable at 200% zoom)

## Mocking

**Not Applicable:** Shopify Liquid theme doesn't require mocking.

**Data Sources:**
- All product/collection data comes from Shopify API automatically
- No external API calls in theme
- No database queries (Shopify handles backend)
- Test data available in development shop

## Fixtures and Test Data

**Development Shop Setup:**
- Product catalog with sample products (image, variants, pricing)
- Collections: at least one for best-sellers section
- Navigation menus: main-menu with links
- Settings: configured in `config/settings_data.json` with dark theme defaults

**Community Data (Custom):**
- Community names populated from Shopify admin (custom product metafields or lines)
- Rendered via `snippets/community-list.liquid`
- No fixtures file; uses live shop data

## Coverage

**Requirements:** None enforced

**Critical Sections for Manual Testing:**
- Header navigation (highest user interaction)
- Product page (purchase path)
- Cart functionality (core conversion)
- Checkout (Shopify native, not tested in theme)

**Lower Priority:**
- Blog/article pages (informational, less complex)
- Footer links (navigation)
- Account pages (Shopify native)

## Test Types

**Manual Testing (Current Approach):**

**Desktop Browser Testing:**
- Chrome, Firefox, Safari latest versions
- Test at: 1920x1080, 1280x720
- Check: nav layout, product cards, scrolling

**Mobile Browser Testing:**
- Chrome/Safari on iOS (latest)
- Chrome on Android (latest)
- Test at: 375px (iPhone SE), 768px (iPad)
- Check: mobile nav, touch interactions, responsive images

**Accessibility Testing:**
- Keyboard navigation (Tab, Shift+Tab, Enter, Escape)
- Screen reader (NVDA on Windows)
- Color contrast tools (WCAG AA standard)
- Focus visible on all interactive elements

**Performance Testing (Manual):**
- Lighthouse audit via Chrome DevTools
- Image lazy-loading verification
- Critical CSS loading (preload=true)
- JavaScript bundle size (minimal; only vanilla JS)

## Error Scenarios to Test Manually

**Edge Cases:**
- Product with no images: renders placeholder SVG (`'product-1' | placeholder_svg_tag`)
- Product with no variants: quantity stepper only, no pills
- Empty collection: "No products found" message (via `{% else %}` in `{% for %}`)
- Missing section setting: uses default value or renders nothing
- Very long product title: wraps correctly, doesn't break layout
- Sold-out variant: button disabled, "Sold Out" text shown

**Mobile Navigation:**
- Open menu, tap link: closes menu, navigates
- Open menu, press Escape: closes menu
- Open menu, tap backdrop: closes menu
- Tap hamburger twice: opens then closes (toggle)

**Scroll Animations:**
- Scroll to element: .fade-up class triggers, element animates in
- Scroll back up and down: animation doesn't trigger twice (observer.unobserve prevents)
- Multiple elements with staggered delays: arrive in sequence (data-delay attribute)

## No Test Runner or CI Pipeline

**Note:** Theme does not have:
- Jest, Vitest, or other unit test framework
- Cypress, Playwright, or E2E test framework
- GitHub Actions or CI/CD pipeline for tests
- Test coverage reporting

**Recommendation for Future:**
If automated testing becomes necessary:
1. Add Shopify CLI theme testing (theme-check linter)
2. Consider E2E testing via Playwright for critical paths
3. Add Lighthouse CI for performance monitoring
4. Implement schema validation via JSON Schema validators

---

*Testing analysis: 2025-02-14*
