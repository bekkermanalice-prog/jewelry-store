# JEWERLY O&A

A premium jewelry storefront with a built-in admin dashboard, delivered as a single static HTML file. No build step, no server, no framework — open `index.html` in a browser and it works.

## What it does

- **Storefront** — a catalog grid of jewelry pieces with filtering by collection, search, and sorting by price/name. Clicking any piece opens a detail view with an image gallery and a "Purchase via WhatsApp" button that pre-fills a message with the item name and price.
- **Admin dashboard** — reachable via the "Admin" link in the nav or footer, gated by a password (`1234` by default — change the `ADMIN_PASSWORD` constant in `index.html` before going live). Once signed in, the admin can add, edit, and delete catalog items: name, price, collection, a status badge, a description, a main image (file upload or URL), and a multi-image gallery (file upload or URL).
- **Persistence** — the catalog is stored in Firebase Realtime Database (`FIREBASE_DB_URL` in `index.html`), so items added or edited by the admin show up for every visitor on the live site. The admin session itself is stored in `sessionStorage` and clears when the tab closes.

## Key technologies

- Plain HTML + vanilla JavaScript (no framework, no bundler)
- Tailwind CSS via the CDN script (`cdn.tailwindcss.com`)
- Google Fonts (Cormorant Garamond + Manrope) via CDN
- Firebase Realtime Database (REST API) for the catalog; `localStorage` as an offline cache, `sessionStorage` for the admin session

## Running it locally

Just open `index.html` in any modern browser — double-click the file, or run a tiny static server if you prefer:

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Configuration

Open `index.html` and edit the constants near the top of the `<script>` block:

- `WHATSAPP_NUMBER` — your WhatsApp number in international format, digits only (no `+`, spaces, or dashes).
- `ADMIN_PASSWORD` — the password used to unlock the admin dashboard.

## Notes

- Because storage is per-browser `localStorage`, catalog changes made by the admin on one device/browser will not automatically appear on another device. For a multi-device, always-in-sync catalog, this would need to move to a real backend/database — the current build intentionally stays a static file per the project requirements.
