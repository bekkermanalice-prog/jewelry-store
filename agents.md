# AGENTS.md

## Architecture

This project is a single static file: `index.html`. It contains all markup, Tailwind config (via the CDN's `tailwind.config` inline script), CSS (in a `<style>` block), and JavaScript (in a single IIFE at the bottom). There is no build step, package.json, or bundler — the file is meant to be opened directly in a browser or served as-is by any static host.

The page renders two mutually-exclusive views within the same DOM:

- **Storefront** (`<main id="top">` plus the header/footer) — always rendered, read-only for customers.
- **Admin dashboard** (`#adminView`) — a full-screen overlay, hidden by default, shown after a correct password is entered. It sits over the storefront rather than being a separate page/route.

## State management

- The shared catalog lives in Firebase Realtime Database (`FIREBASE_DB_URL`, path `/products`), accessed via the REST API (`fetch` GET/PUT on `/products.json`) — no Firebase SDK. `loadProducts()` fetches it; `saveProducts()` PUTs the whole `products` array.
- `localStorage` (`joa_catalog_v1`) is only a cache used to render instantly before the Firebase fetch completes, and as a fallback if Firebase is unreachable.
- If the Firebase `/products` path is empty, it is seeded from the cached catalog (or `SEED_PRODUCTS` if there is no cache). Firebase drops empty arrays, so `normalizeProducts()` restores missing `gallery`/`badge` fields.
- The database rules must allow public read and write on `/products` for this to work.
- Admin login state lives in `sessionStorage` (`joa_admin_session`) — intentionally session-scoped, not persisted to `localStorage`, so the dashboard isn't left unlocked indefinitely.
- Uploaded images (main + gallery) are converted to base64 data URLs via `FileReader` and stored directly inside the product object in `localStorage`. There is no separate object storage — this keeps the app a single static file with zero backend dependencies, at the cost of `localStorage`'s ~5-10MB size ceiling if an admin uploads many/large images.

## Key sections in `index.html`

- `SEED_PRODUCTS` — initial demo catalog data.
- `renderTabs` / `renderCatalog` / `openProductModal` / `renderModalGallery` — storefront rendering.
- `showLoginModal` / `enterAdmin` / `exitAdmin` — admin auth flow (password check against the `ADMIN_PASSWORD` constant).
- `renderAdminList` / `openItemForm` / item form submit handler / `promptDelete` — admin CRUD.
- `whatsappLink(product)` — builds the `wa.me` deep link with a pre-filled message; used by both the storefront product modal and the general contact CTAs.

## Conventions

- No comments explaining *what* code does — identifiers and structure should be self-explanatory. Comments (sparse) are only used for section dividers and the one non-obvious note about `localStorage` being intentional, not a placeholder for a "real" backend.
- Styling is Tailwind utility classes directly in markup; the small custom `<style>` block only covers things Tailwind utilities can't express cleanly (grain overlay, marquee animation, shimmer text, custom scrollbar).
- Colors are defined as a `plum` palette in the Tailwind CDN config (`tailwind.config.theme.extend.colors.plum`) — use these shades rather than introducing new arbitrary colors, to keep the purple/white palette consistent.

## Known limitations / non-obvious decisions

- Catalog data is shared across all visitors through Firebase. Because the admin password check is client-side, the Firebase write rule is effectively open to anyone who knows the database URL.
- The admin password (`1234` by default) is checked client-side in plain JavaScript, which means it is visible to anyone who reads the page source. This is acceptable for the stated use case (a single trusted admin deterring casual visitors) but is not real authentication — do not extend this pattern to protect genuinely sensitive data without adding a real backend.
- `WHATSAPP_NUMBER` and `ADMIN_PASSWORD` are hardcoded constants near the top of the `<script>` block in `index.html` — update them there before deploying for a real business.
