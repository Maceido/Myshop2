<!-- .github/copilot-instructions.md - guidance for AI coding agents working on this repo -->
# Project snapshot

This repository is a Shopify storefront theme built using Liquid templates, static assets, and inline client-side JS. The theme uses the standard Shopify directory layout: `layout/`, `templates/`, `sections/`, `snippets/`, `assets/`, `config/`, and `locales/`.

# Big picture (what to know first)

- **Rendering model:** Pages are rendered server-side by Shopify Liquid. Templates in `templates/` map to routes (e.g., `templates/product.liquid` for product pages). `layout/theme.liquid` provides the global page chrome and includes `{{ content_for_layout }}`.
- **Composition:** `sections/` provide reusable blocks, `snippets/` contain small HTML + JS fragments (example: `snippets/sidebar-menu.liquid`). Templates and sections often `render` snippets: `{% render 'sidebar-menu' %}`.
- **Assets & styles:** Static CSS/JS are under `assets/` (this repo uses `assets/theme.css` and in-template <script> tags). Styles are loaded in `layout/theme.liquid` with `{{ 'theme.css' | asset_url | stylesheet_tag }}`.
- **Server → Client data flow:** Liquid variables (e.g., `product`, `cart`, `shop`) are used server-side and occasionally serialized into JS via `{{ product | json }}` (see `templates/product.liquid`). When you change how data is used in client JS, update the Liquid serialization accordingly.

# Key files to reference (examples)

- `layout/theme.liquid` — global head, header, sidebar, and `{{ content_for_layout }}` placement.
- `templates/product.liquid` — product page layout, example of `{{ product | json }}` and client-side variant handling.
- `snippets/sidebar-menu.liquid` — category menu and small inline JS for toggling sublists.
- `config/settings_schema.json` — theme settings; e.g., `accent_color` is declared here (use via `settings.accent_color` in Liquid).
- `assets/theme.css` — central stylesheet; project uses BEM-like class names (e.g., `.product-card`, `.sidebar-menu`).

# Project-specific patterns and conventions

- Inline scripts are common. Expect JS to be embedded in Liquid templates/snippets (not separated by a bundler). When adding JS that depends on Liquid data, prefer serializing the object with `| json` close to its consumer.
- Forms typically use Shopify form tags: `{% form 'product', product %}` — keep these untouched when adding custom client handlers unless also preserving Shopify's expected inputs (e.g., `input[name="id"]`).
- Use `routes.*` globals for links (`routes.cart_url`, `routes.root_url`, `routes.account_url`) instead of hardcoding paths.
- Collections access: `collections['frontpage'] | default: collections['all']` is used to select featured products in `templates/index.liquid`. Follow the same `| default:` fallback pattern.

# Dev / edit / deploy workflows (practical commands)

- Local preview & edit: use the Shopify CLI to preview and push theme changes. Common commands (run from project root):

  - `shopify theme serve` — serve a local preview (hot-reloads assets & templates).
  - `shopify theme push` — push current files to a store theme.

If you don't have Shopify CLI configured, edits can be tested by deploying to a development theme. There is no Node.js/webpack setup in this repo, so adding a bundler will be an explicit change requiring new files.

# Debugging tips (what to check first)

- Liquid errors: syntax or missing variables show on rendered pages in the Shopify preview; check `layout/theme.liquid` and the template that caused the error.
- JS errors: open the browser console. Many scripts access DOM elements that exist only on certain templates (e.g., product page script expects `#product-form`). Guard your code with existence checks.
- Variant handling: client JS relies on `product.variants` and sets a hidden `input[name="id"]` for form submissions — preserve that behavior when modifying product scripts.

# When to change what files

- Add a new visual block: create a file in `sections/` and include it in the corresponding `template` or via the Shopify theme editor schema (if adding schema, modify the new section file accordingly).
- Small reusable markup/JS: add to `snippets/` and `render` from templates — keep snippets minimal and pure HTML/JS.
- Global styles: update `assets/theme.css`. If adding many JS files, place them in `assets/` and include via `{{ 'file.js' | asset_url | script_tag }}`.

# Examples (copy-paste safe snippets)

- Serialize product for client JS (used in `templates/product.liquid`):

  `var productData = {{ product | json }};`

- Render snippet from layout (used in `layout/theme.liquid`):

  `{% render 'sidebar-menu' %}`

# What not to assume

- There is no package.json or build system in the repository root. Don't add changes that require a build step without also adding CI instructions and a lockfile.
- Sections `main-product.liquid` and `main-collection.liquid` are present but empty; they may be placeholders — check with the maintainer before deleting.

# When you need more information

- Ask the repo owner which Shopify store and CLI credentials to use for `shopify theme push` / `serve` and whether introducing a JS build system is acceptable.

---

If any of the areas above look incomplete or you want examples expanded (for instance, a recommended branching/commit message style or CI steps for theme pushes), tell me what to include and I'll update this file.
