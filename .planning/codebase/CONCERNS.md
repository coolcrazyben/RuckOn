# Codebase Concerns

**Analysis Date:** 2025-03-06

## Tech Debt

**Hardcoded rgba() colors scattered throughout CSS:**
- Issue: Brand color values are defined in `snippets/css-variables.liquid` but specific rgba() opacity variants are hardcoded in individual sections and `assets/critical.css`. This creates maintenance burden when brand guidelines change.
- Files: `assets/critical.css` (lines 117, 323, 347, 370-371, 385), `sections/product.liquid` (lines 472, 498, 513, 524, 555, 574, 625, etc.)
- Impact: Changing brand color strategy or opacity values requires editing multiple files instead of single source. Risk of inconsistent colors if updates missed.
- Fix approach: Extract all rgba() variants to CSS custom properties in `snippets/css-variables.liquid` (e.g., `--color-tan-10pct`, `--color-olive-15pct`) and reference consistently.

**Monolithic section files with 1000+ lines:**
- Issue: `sections/product.liquid` (1298 lines), `sections/community-creation-page.liquid` (1010 lines), and `sections/cart.liquid` (809 lines) contain both markup, styling, and JavaScript in single files.
- Files: `sections/product.liquid`, `sections/community-creation-page.liquid`, `sections/cart.liquid`
- Impact: Harder to maintain, test, and debug. Makes code review slower. Difficult to reuse product-specific logic across other sections.
- Fix approach: Extract reusable components into snippets (e.g., `snippets/product-gallery.liquid`, `snippets/product-options.liquid`, `snippets/variant-selector.liquid`). Keep sections as thin presentation wrappers.

**Inline JavaScript with minimal error handling:**
- Issue: JavaScript in product.liquid (lines 863-1047) uses try/catch only for JSON parsing but lacks error handling for DOM queries, fetch failures recovery, and edge cases.
- Files: `sections/product.liquid` (lines 893-1046), `sections/cart.liquid` (lines 621-677), `sections/header.liquid` (lines 650-690)
- Impact: Silent failures when DOM elements don't exist (e.g., if variant data missing). No user feedback on fetch errors in cart community selector. Mobile nav may not respond if JavaScript fails.
- Fix approach: Add null checks before DOM manipulation. Implement retry logic for fetch operations. Log errors to console in development. Add data-required attributes to critical elements.

**No centralized error tracking or logging:**
- Issue: Errors silently fail. No Sentry, Bugsnag, or similar service configured to catch and report JavaScript errors in production.
- Files: All `{% javascript %}` blocks across sections
- Impact: Production bugs go unnoticed. No visibility into user-facing failures until customers report them.
- Fix approach: Integrate error tracking service (e.g., Sentry Lite via CDN). Log critical errors to analytics service.

**Missing input validation on community name field:**
- Issue: Community name field (product.liquid, lines 209-231) uses client-side validation only. No server-side validation ensures data quality.
- Files: `sections/product.liquid` (lines 209-231, 1006-1026)
- Impact: Malformed or missing community names can be submitted. Patch printing could fail silently.
- Fix approach: Add server-side validation on cart update endpoint. Return error response if community name empty or exceeds maxlength.

## Known Bugs

**Cart community selector AJAX doesn't validate response:**
- Symptoms: If the `/cart/change.js` endpoint returns an error, the UI shows "Error — try again" but the cart may still have stale data. No refresh happens.
- Files: `sections/cart.liquid` (lines 641-676)
- Trigger: Network timeout, server error, or invalid item key during community selector change
- Workaround: User must manually refresh page to see actual cart state

**Product variant image sync misses edge cases:**
- Symptoms: When a variant has no featured image, the main image doesn't update. The thumbnail active state updates but main image remains stale.
- Files: `sections/product.liquid` (lines 957-966)
- Trigger: Products with variants where some have images and others don't
- Workaround: Ensure all variants have images in Shopify admin

**Mobile nav doesn't trap focus properly:**
- Symptoms: Focus can leak out of mobile nav drawer to elements behind it when nav is open.
- Files: `sections/header.liquid` (lines 679-691)
- Trigger: Open mobile nav, tab through links, reach bottom link, tab again
- Workaround: Press Escape or click backdrop to close nav

**Password page email capture form doesn't persist across navigation:**
- Symptoms: If user enters email on password page and then navigates away via browser back button, form may be reset.
- Files: `sections/password.liquid` (lines 48-90)
- Trigger: Browser caching behavior with form POST
- Workaround: None — expected behavior for password-protected pages

## Security Considerations

**innerHTML used for price HTML injection:**
- Risk: If `formatMoney()` or price data becomes compromised, malicious HTML could be injected.
- Files: `sections/product.liquid` (line 942)
- Current mitigation: Data comes from Shopify system, not user input. formatMoney function is local and safe.
- Recommendations: Replace innerHTML with textContent + createElement for safer DOM updates. Consider using a template element approach.

**Rich text fields rendered without escaping:**
- Risk: Editor might paste HTML/JavaScript in description tabs if using richtext settings.
- Files: `sections/product.liquid` (lines 305, 318, 329), `sections/article.liquid`, `sections/password.liquid` (lines 59-60)
- Current mitigation: Shopify sanitizes richtext field outputs by default.
- Recommendations: Verify richtext fields are marked as `richtext` type in schema (automatic sanitization). Document that custom HTML is not allowed.

**No Content Security Policy (CSP) headers:**
- Risk: Third-party script injection, inline script execution not restricted.
- Files: `layout/theme.liquid` (loads Google Fonts, Shopify scripts)
- Current mitigation: Shopify handles most CSP concerns. Google Fonts loaded via standard tag.
- Recommendations: Add CSP meta tag in `layout/theme.liquid` to restrict script sources. Document allowed third-party integrations.

**form.posted_successfully? not validated:**
- Risk: Form submission success state displayed without verifying actual email capture in Shopify contact form.
- Files: `sections/password.liquid` (line 53)
- Current mitigation: Shopify form object is reliable.
- Recommendations: Add server-side email logging webhook if compliance audit required.

## Performance Bottlenecks

**Product page with many variants loads all variant JSON inline:**
- Problem: `sections/product.liquid` (line 339-341) embeds ALL product variants as JSON in page. For products with 100+ variants, this adds significant payload.
- Files: `sections/product.liquid` (lines 339-341)
- Cause: JavaScript needs variant data to match selected options. All variants passed upfront instead of lazy-loaded.
- Improvement path: Load variant data on-demand via AJAX. Store variant map in sessionStorage after first load. Limit initial variant array to 50 most popular.

**Sticky product gallery repaints on every scroll:**
- Problem: `.product-gallery` uses `position: sticky` with `top: calc(var(--header-total-height) + 24px)`. No throttling on scroll listener measurement.
- Files: `sections/product.liquid` (lines 425-427)
- Cause: Browser must recalculate sticky position on every scroll frame.
- Improvement path: Use CSS will-change hint. Ensure header height vars are stable. Consider intersection observer for sticky breakpoint instead.

**No lazy loading on off-screen images in grid sections:**
- Problem: Collection, best-sellers, and shop-by-category grids load all product images. Image tags use `loading: lazy` but no sizes attribute optimization.
- Files: `sections/best-sellers.liquid`, `sections/collection.liquid`, `sections/search.liquid`
- Cause: Images below fold load at same priority as above-fold images.
- Improvement path: Add responsive image sizes. Implement native lazy intersection observer. Prioritize above-fold image loading.

**No caching headers set for critical CSS:**
- Problem: `assets/critical.css` loaded on every page but no long-term caching.
- Files: `layout/theme.liquid` (loads critical.css)
- Cause: Shopify asset URLs don't include content hash by default in some older setups.
- Improvement path: Verify Shopify's asset versioning. Add cache-control headers in theme editor settings if available.

## Fragile Areas

**Mobile navigation state management:**
- Files: `sections/header.liquid` (lines 650-691)
- Why fragile: JS relies on data attributes (`aria-expanded`, `aria-hidden`) to track state. If HTML structure changes (e.g., ID renamed), nav breaks silently. Event listener cleanup incomplete on section update.
- Safe modification: Update HTML IDs → must update all 6 event listener targets. Test mobile nav after any header changes.
- Test coverage: No automated tests for nav open/close/escape key behavior. Manual testing required.

**Product variant selection logic:**
- Files: `sections/product.liquid` (lines 885-922)
- Why fragile: Matching algorithm assumes variant.options array order matches option display order. If Shopify variant options get reordered, matching fails. No validation that selectedOptions length matches available options.
- Safe modification: Add console logging to verify option order before matching. Test with products that have >2 option types. Verify Shopify doesn't reorder options.
- Test coverage: Only tested with 1-2 variants during development likely. Need test products with 3+ options, out-of-stock variants.

**Community selector dropdown binding:**
- Files: `sections/cart.liquid` (lines 637-677)
- Why fragile: Assumes `cart-community-select` exists. Uses `dataset.itemKey` which must match Shopify's line item key format. Fetch error handling shows message but doesn't revert UI state.
- Safe modification: Add null checks. Test with invalid item keys. Verify Shopify's line item key format hasn't changed.
- Test coverage: No error case testing (network timeout, bad response, permission denied).

**Scroll animation IntersectionObserver:**
- Files: `snippets/scroll-animations.liquid`
- Why fragile: Global observer watches all `.fade-up` elements. If many elements added via AJAX (e.g., infinite scroll), observer may not detect them if added after initial setup.
- Safe modification: Ensure any dynamic content also adds `.fade-up` class. Test with pagination and AJAX-loaded content.
- Test coverage: Static page content only tested. No dynamic content scenarios verified.

## Scaling Limits

**Product gallery thumbnail grid:**
- Current capacity: ~15 thumbnails visible, 72px each (from critical.css). Beyond ~20 images, layout breaks to new row.
- Limit: Products with 50+ variants/images would need horizontal scroll, which isn't implemented.
- Scaling path: Implement carousel for thumbnails (e.g., Swiper.js). Or implement lazy-load tabs (e.g., "View more images"). Or add pagination.

**Related products hardcoded to 4 columns:**
- Current capacity: 4-column grid at desktop, 3 at tablet, 2 at mobile. If related_limit >12, grid becomes unusable.
- Limit: Collections with very large product counts may serve 50+ related products, causing performance issues.
- Scaling path: Respect related_limit setting (already does). Add carousel view for related products. Or paginate related section.

**Cart quantity input type="number":**
- Current capacity: Browser-native number input supports 0-999 safely on most devices.
- Limit: JavaScript stepper only checks `cur > 1` for decrease. No upper limit. Type="number" may behave oddly with very large numbers.
- Scaling path: Add max quantity validation. Set `max` attribute on input. Validate on server (Shopify handles this).

## Dependencies at Risk

**Google Fonts CDN dependency:**
- Risk: If fonts.googleapis.com is down or blocked (corporate firewall, sanctions), text falls back to system fonts, design breaks.
- Impact: Site becomes unusable (headings unreadable if fonts missing).
- Migration plan: Host fonts locally (downloadable from Google Fonts). Add font-display: swap for better fallback. Or migrate to system font stack as fallback.

**Shopify API form tags reliability:**
- Risk: If Shopify form endpoints change behavior, validation could break. Community selector AJAX to `/cart/change.js` may become deprecated.
- Impact: Cart updates, email capture, password entry could fail silently.
- Migration plan: Monitor Shopify API changelog. Have fallback form submission if fetch fails. Test quarterly against latest Shopify API version.

## Missing Critical Features

**No accessible focus management in modals/drawers:**
- Problem: Mobile nav drawer opens but focus doesn't move into nav. Screen reader users can't easily interact with expanded nav.
- Blocks: Accessibility compliance (WCAG 2.1 AA). Mobile usability for assistive technology users.

**No analytics event tracking for conversions:**
- Problem: Email capture form (password.liquid) has no pixel/event tracking to measure capture rate. Community selector updates (cart.liquid) not tracked.
- Blocks: Inability to measure feature adoption and success. Marketing attribution gaps.

**No A/B testing framework:**
- Problem: CTA button text, email form copy, hero images all hardcoded in schema. Can't run experiments.
- Blocks: Data-driven optimization of conversion funnels.

## Test Coverage Gaps

**Product variant selection with edge cases:**
- What's not tested: Products with 10+ variants, out-of-stock combinations, variant images missing, option ordering edge cases
- Files: `sections/product.liquid` (lines 885-922)
- Risk: Silent failures when user selects unavailable combination. Wrong image displayed.
- Priority: High (revenue impact)

**Cart line item update error recovery:**
- What's not tested: Network timeouts, server 500 errors, invalid line item IDs, concurrent updates
- Files: `sections/cart.liquid` (lines 641-676)
- Risk: Cart shows "Error" message but data is stale. User completes checkout with wrong selections.
- Priority: High (order accuracy)

**Mobile navigation accessibility:**
- What's not tested: Keyboard navigation (Tab, Shift+Tab, Escape), screen reader announcement of open state, focus trapping
- Files: `sections/header.liquid` (lines 650-691)
- Risk: Mobile nav inaccessible to keyboard and screen reader users (WCAG violation).
- Priority: Medium (compliance + accessibility)

**Password form email validation:**
- What's not tested: Invalid email formats accepted, long emails (255+ chars), SQL injection attempts, rate limiting
- Files: `sections/password.liquid` (lines 48-90)
- Risk: Invalid data stored in contacts. Potential spam/bot abuse if no rate limit.
- Priority: Medium (data quality + security)

**Product tab switching interactions:**
- What's not tested: Rapid tab switching, missing tab panels, keyboard navigation of tabs, focus management
- Files: `sections/product.liquid` (lines 1029-1046)
- Risk: Tab content doesn't display, focus gets lost, keyboard users stuck.
- Priority: Low (basic functionality works but edge cases untested)

---

*Concerns audit: 2025-03-06*
