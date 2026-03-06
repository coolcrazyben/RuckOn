# External Integrations

**Analysis Date:** 2025-03-06

## APIs & External Services

**Typography:**
- Google Fonts API - Serves custom fonts (Black Ops One, Barlow, Barlow Condensed)
  - Endpoint: `https://fonts.googleapis.com/css2?family=...`
  - Used in: `layout/theme.liquid`, `layout/password.liquid`
  - Preconnect: `https://fonts.gstatic.com` for font file delivery

**Social Media:**
- Twitter (via Open Graph meta tags)
  - Implementation: `snippets/meta-tags.liquid` generates `twitter:card`, `twitter:title`, `twitter:description` meta tags
  - Type: Static metadata only, no API integration

**Search & SEO:**
- Open Graph Protocol
  - Implementation: `snippets/meta-tags.liquid` generates `og:title`, `og:url`, `og:description`, `og:image`, `og:price:amount`, `og:price:currency`
  - Supports: Facebook, LinkedIn, Pinterest sharing previews

## Data Storage

**Databases:**
- Shopify's managed database
  - Connection: Native Shopify API (accessed via Liquid)
  - Data types: Products, collections, customers, orders, cart, blog articles
  - Client: Shopify Liquid template language

**File Storage:**
- Shopify CDN (for theme assets and merchant uploads)
  - Images: `shopify://shop_images/` protocol in theme editor
  - Static assets: `assets/` directory (critical.css, SVGs, etc.)
  - Delivery: Via Shopify's image_url filter and CDN

**Caching:**
- Shopify's built-in CDN caching
  - Strategy: Page caching at CDN edge nodes

## Authentication & Identity

**Auth Provider:**
- Shopify native authentication
  - Implementation: Shopify customer login/logout/register forms
  - Uses: Shopify `{% form 'customer_login' %}`, `{% form 'create_customer' %}`, `{% form 'recover_customer_password' %}`
  - Stored in: Shopify customer database

**Newsletter Signup:**
- Shopify customer contact form
  - Implementation: `sections/newsletter.liquid` uses `{% form 'customer' %}`
  - Tags: Contact records tagged with `newsletter` for segmentation
  - Submission: Posts to Shopify contact endpoint

## Monitoring & Observability

**Error Tracking:**
- Not configured (relies on Shopify's default error handling)

**Logs:**
- Shopify Theme Editor - Browser console for client-side errors
- Shopify Admin - Order and customer activity logs

**Analytics:**
- Injected via `{{ content_for_header }}`
  - Shopify native analytics
  - Any installed Shopify apps (e.g., Klaviyo, Segment, custom tracking)
  - Note: Script injection happens in layout, no explicit configuration in theme code

## CI/CD & Deployment

**Hosting:**
- Shopify Hosted Platform
  - Theme ID: 151510188199 (primary production theme)
  - Alternative theme: 151505830055 (secondary/staging)

**CI Pipeline:**
- Shopify CLI theme push (manual deployment via cli commands)
  - Workflow: `git push && shopify theme push --theme 151510188199 --allow-live`
  - Deployed to: Live Shopify store
  - Theme Check validation: Recommended pre-push (not enforced in CI)

**Version Control:**
- GitHub repository with Shopify theme sync

## Environment Configuration

**Required env vars:**
- None (Shopify handles all credentials)

**Secrets location:**
- Shopify Admin Dashboard (no secrets in code)

## Payment Processing

**Checkout:**
- Shopify Checkout (native, handled outside theme)
  - Implementation: Form `action="/cart"` method POST in `sections/cart.liquid`
  - Checkout button: Links to Shopify's managed checkout page
  - Payment icons: Can be shown via theme editor (configured in `config/settings_schema.json`)

**Cart Management:**
- Shopify Cart API
  - Endpoint: `/cart/change.js` (POST via fetch in `sections/cart.liquid`)
  - Handles: Quantity updates, item removal via JavaScript

## Webhooks & Callbacks

**Incoming:**
- Shopify Order Webhooks (for external integrations if apps installed)
  - Managed by: Shopify apps (not directly in theme)

**Outgoing:**
- Shopify Form Submissions
  - Newsletter signup posts to Shopify contacts
  - Customer forms (login, password reset) post to Shopify auth endpoints

## Product & Inventory

**Product Data:**
- Shopify Product API
  - Accessed via: Liquid `product` object in `sections/product.liquid`
  - Data passed to client: Variant data via `application/json` script tag
  - Client-side processing: JavaScript parses variant JSON and updates prices/images

**Community Features:**
- Custom property: `properties[Community]`
  - Implementation: `sections/product.liquid` allows community selection on basic membership products
  - Tag-based: Products tagged with `basic-membership` trigger the community selector
  - Data persistence: Stored in Shopify cart line item properties

**Patch Subscription:**
- Custom detection: `sections/cart.liquid` checks for product tag `patch-subscription-tag`
  - Allows special community property handling for subscription products

## Form Integrations

**Newsletter Signup:**
- Endpoint: Shopify customer contact form
- Fields: email (required), tags (newsletter)
- Response: Success message or error display

**Blog Comments:**
- Shopify native comment system
  - Implementation: `sections/article.liquid` uses `{% form 'new_comment', article %}`
  - Moderation: Managed in Shopify Admin

**Customer Login/Registration:**
- Shopify native forms
  - Endpoints: Managed by Shopify
  - Supported forms: customer_login, create_customer, recover_customer_password

---

*Integration audit: 2025-03-06*
