# Technology Stack

**Analysis Date:** 2025-03-06

## Languages

**Primary:**
- Liquid (Shopify template language) - All dynamic page rendering
- HTML5 - Semantic markup structure
- CSS3 - Styling, animations, responsive design
- JavaScript (ES6+) - Client-side interactivity (vanilla, no framework)

## Runtime

**Environment:**
- Shopify Liquid Template Engine (server-side)
- Browser runtime (client-side: Chrome, Firefox, Safari, Edge)

**Package Manager:**
- No npm/yarn/pnpm (this is a pure Shopify theme, not a Node.js project)
- No external package dependencies required

## Frameworks

**Core:**
- Shopify Skeleton Theme v1.0.0 - Base theme architecture and patterns

**CSS Framework:**
- Custom CSS (no Bootstrap, Tailwind, or utility framework)
- CSS Grid and Flexbox for layout
- CSS custom properties (variables) for theming

**Build/Dev:**
- Shopify CLI - Theme development, preview, and deployment
- Theme Check (`.theme-check.yml`) - Linting and best practices validation

## Key Dependencies

**Critical:**
- Google Fonts API - Typography (Black Ops One, Barlow, Barlow Condensed)
  - Loaded via `https://fonts.googleapis.com/css2` with font-display: swap
  - Preconnected to `https://fonts.gstatic.com` for performance

**Infrastructure:**
- Shopify Platform - E-commerce, checkout, inventory, customer management
  - Uses Shopify Liquid object access (product, collection, cart, customer, etc.)
  - `{{ content_for_header }}` injects Shopify scripts (analytics, apps, etc.)

## Configuration

**Environment:**
- Managed through Shopify Admin theme editor
- No .env files (secrets handled by Shopify)
- Theme editor settings defined in `config/settings_schema.json`
- Live settings stored in `config/settings_data.json`

**Build:**
- `.theme-check.yml` - Theme linting config
- `.shopifyignore` - Files excluded from theme push
- `.gitignore` - Git ignore rules
- `.gitattributes` - Line ending handling (CRLF)

## Platform Requirements

**Development:**
- Shopify CLI (latest version)
- Git for version control
- Modern browser (Chrome, Firefox, Safari, Edge)
- Text editor or IDE (VS Code recommended)

**Production:**
- Shopify Plus or Shopify Standard plan
- Modern browser support (no IE11 compatibility)
- JavaScript enabled (for interactive features)

---

*Stack analysis: 2025-03-06*
