# Copilot / AI agent instructions for jkke-portfolio

Purpose: Provide focused, actionable context so an AI code assistant can be immediately productive working on this static JKKE portfolio/site.

Summary (big picture)
- This is a small static site (HTML + inline JS + JSON dataset). Main user-facing feature is a JKKE catalog and simple BQ (Bill of Quantities) builder that uses localStorage to persist a "project" cart across pages and produce PDF/Excel exports.
- Key data source: `bqData.json` (flat JSON object keyed by JKKE code, e.g. `"1A1": { description, price }`). This is the single source of truth for item descriptions and unit prices.
- No build system, server side, or tests. Pages are standalone and designed to run in the browser or via simple static hosting (GitHub Pages or local static server).

Key files and responsibilities
- `bqData.json` — the dataset of JKKE codes/descriptions/prices. Editing here changes system data everywhere.
- `semuajkke.html` — catalog UI, search, add-to-cart; uses `localStorage` to save `projectCart` (the project BQ). It loads `bqData.json` and renders items in pages.
- `jkke.html` — main lookup dashboard and estimator (search by code, price lookup, area-based calculator). Integrates the Dify/udify chatbot via an embed token.
- `generateJKKE.html` — export utilities for generating Lampiran C (PDF/Excel) from `projectCart` using `pdfMake` and `XLSX` libs (CDN).
- `bq.html` — Bill of Quantities page (table + PDF export). Reads `projectCart` from `localStorage` and supports quantity editing/deleting/clearing.
- `checkout.html` — final export/checkout page (expects project data in a different key sometimes; see gotchas).
- `index.html` — portfolio landing page linking to system features.

Developer workflows (what to run and verify)
- Preview locally: serve files with any static server (examples):
  - Python: `python -m http.server 8000` (then open `http://localhost:8000/jkke.html`)
  - Node: `npx http-server . -p 8000` or use VS Code Live Server extension
- Manual verification checklist after code changes:
  1. Load `semuajkke.html` and confirm data loads from `bqData.json` and paging/search works.
  2. Add an item to the project (catalog), open `bq.html`, confirm it appears, change quantity, generate PDF/Excel from `generateJKKE.html` and `bq.html`.
  3. Test `jkke.html` lookup, estimator, and chatbot button (click to ensure the embed loads).  
- No unit/integration tests exist; use browser manual tests and check PDF/Excel outputs for correct totals and formatting.

Important project conventions & patterns (be explicit)
- LocalStorage key: `projectCart` — this is the canonical key used across most pages. Always update this key when changing the cart logic.
- Item object shape used across pages (common fields): `{ code, description, price, quantity, remarks }`. Note: some pages use `quantity` while others read/update `qty` — search for both (`quantity` and `qty`) before renaming a property.
- Code parsing rule: code -> `section = code.charAt(0)`, `billNo = code.substring(1)` used in multiple exports and displays; keep consistent when you introduce features that classify codes.
- Currency formatting: multiple implementations exist (e.g., `formatNumber()` in `semuajkke.html` and `toLocaleString` in other pages). Prefer the existing helpers in the page you edit.
- Export libraries are used via CDN: `pdfMake` (for PDFs) and `XLSX` (for Excel). Use them instead of introducing new heavy dependencies.
- UI uses inline styles + vanilla JS. Keep changes simple and avoid adding frameworks unless the project scope expands.

Security & secrets
- The Dify/udify chatbot token is embedded in `jkke.html` (`window.difyChatbotConfig.token`). Treat tokens as sensitive; do not commit new secrets inadvertently. If you need to rotate or remove the token, test the widget after changes.

Common pitfalls & gotchas (things to watch for)
- Inconsistent property names: `quantity` vs `qty` vs `quantity` usage across pages. This causes silent bugs when editing cart logic — search the repo for both terms and align changes carefully.
- Multiple localStorage keys: `projectCart`, `bqDetails`, `savedData` appear in different files (e.g., `checkout.html` references `bqDetails`/`savedData`). Confirm the intended storage key before changing persistence logic.
- Cross-page coupling: pages assume the same item object shape (description, price). When changing shape, ensure all pages that read `projectCart` are updated (catalog, bq, generate, checkout).
- Large `bqData.json`: the dataset is authoritative and large. Prefer non-destructive edits and keep formatting consistent (strings for description, numbers for price).

How to make safe changes (recommendations)
- When adding a new field to the item shape: add it to `bqData.json` and update `semuajkke.html` (catalog render), `bq.html` (table render), `generateJKKE.html` (export logic) and `checkout.html` (download/render). Run manual flows to validate.
- When fixing UI/JS bugs: reproduce in the browser, add console logs to trace `localStorage` reads/writes, and ensure pages remain compatible after fixes.
- For formatting/locale changes: keep locale usage (`ms-MY`, `en-MY`) consistent with user-facing pages.

Search tips (where to look first)
- Search for: `projectCart`, `bqData.json`, `quantity`, `qty`, `formatNumber`, `pdfMake`, `XLSX`, `difyChatbotConfig`, `localStorage`.

If you plan a larger refactor
- Propose a migration plan first: outline how you'll migrate localStorage keys and object shapes, provide a short compatibility shim, and list pages to update and manual tests to run.

Contact / follow-up
- If anything in the UX, data format, or token usage is unclear, ask for clarification before making large changes.

---
If this looks right, I can refine wording, add examples for a concrete change (e.g., adding a new field to `bqData.json` and propagating it through the UI), or run a small fix (like unifying `quantity` vs `qty`) — tell me which you'd like next.