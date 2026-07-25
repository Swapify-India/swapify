# Project Structure Overview

**Project:** Swapify — Frontend
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

---

## 1. High-Level Architecture

Swapify is a single-page application with no build tooling and no frontend
framework. One HTML shell holds every screen; one stylesheet holds all styling;
one script file holds all application logic. The frontend talks to a FastAPI +
SQLite backend over plain REST/JSON.

```
Frontend
│
index.html  ──►  contains all pages (one <div class="page"> per screen)
   │
   ▼
style.css   ──►  global styling (design tokens, layout, components, dark mode)
   │
   ▼
script.js   ──►  application logic (navigation, state, rendering, API calls)
   │
   ▼
Backend APIs (FastAPI, see 06_FRONTEND_API_INTEGRATION.md)
   │
   ▼
SQLite (swapify.db) + swapify_products.csv (catalogue seed) + Open Food Facts (fallback)
```

Navigation between "pages" is not real page navigation — there is exactly one HTML
document. Every screen is a `<div id="page-xxx" class="page">` inside `index.html`,
and `showPage(name)` in `script.js` toggles the `active` class to switch which one is
visible. This keeps the app fast (no reloads, no route/build step) at the cost of a
single large HTML/JS file — a trade-off that made sense for a project of this scope
and timeline.

---

## 2. Folder Structure

```
Swapify/
│
├── src/                          Backend application code (FastAPI)
│   ├── app.py                    All API routes, DB access, auth, scoring logic
│   ├── ocr_label_scanner.py      OCR (Tesseract) label-scanning support
│   ├── category_taxonomy.py      Product → category classification rules
│   └── observability.py          Logging / error-reporting hooks
│
├── static/                       Everything the browser loads directly
│   ├── index.html                All frontend pages (single HTML shell)
│   ├── style.css                 Global styling, design tokens, dark mode
│   ├── script.js                 All frontend application logic
│   └── swapify_products.csv      Seed data for the product catalogue
│
├── docs/                         Project documentation (this folder)
│
├── uploads/
│   └── product_images/           Runtime storage for uploaded product photos
│
├── swapify.db                    SQLite database (created/updated at runtime)
├── requirements.txt              Python backend dependencies
│
├── API_DOCS.md                   Backend API reference (Dhruv)
├── FRONTEND_INTEGRATION.md       Backend→frontend integration notes (Dhruv)
├── DEPLOYMENT_FRONTEND.md        Frontend deployment notes (Rashi)
├── PERFORMANCE_REPORT.md         Backend performance/load-test report (Dhruv)
├── sync_db.py                    Ops script: syncs the product CSV into the live DB
├── test_api.sh                   Backend API test script (bash)
└── test_api.ps1                  Backend API test script (PowerShell)
```

> Note: some backend deployments also keep `css/`, `js/`, `images/`, and `icons/`
> as separate asset folders under `static/` as the app grows. In the current build,
> `style.css` and `script.js` are consolidated as single top-level files under
> `static/` for simplicity — see `04_DESIGN_DECISIONS.md` for the reasoning.
>
> The root-level `.md` files (aside from `DEPLOYMENT_FRONTEND.md`), `sync_db.py`,
> and the `test_api.*` scripts are backend/ops artifacts owned by Dhruv, not part
> of the frontend build — included here only so the diagram reflects the full
> repository.

---

## 3. `static/index.html` — Page Inventory

Each screen is a top-level `<div id="page-*" class="page">` inside the single HTML
document. `showPage('name')` shows the matching `#page-name` div and hides the rest.

| Page ID | Screen |
| --- | --- |
| `page-home` | Home |
| `page-scanner` | Scanner (barcode + OCR) |
| `page-product` | Product Detail (score, badges, alternatives, ratings) |
| `page-history` | History (scan log, weekly & monthly panels) |
| `page-favorites` | Favorites |
| `page-profile` | Profile |
| `page-swaps` | My Swaps |
| `page-categories` | Categories |
| `page-cart` | Shopping List |
| `page-compare` | Compare |
| `page-preferences` | Dietary Preferences |
| `page-leaderboard` | Leaderboard |
| `page-challenges` | Challenges |
| `page-settings` | Settings |
| `page-how-scoring-works` | "How Scoring Works" explainer |

The AI Chat window and the Auth (login/register) modal are **not** pages — they are
overlays (`#chatWindow`, `#authOverlay`) that float above whichever page is active,
so the user can open them from anywhere in the app.

---

## 4. `static/script.js` — Functional Breakdown

`script.js` is organized into functional blocks (roughly in file order). The
functions below are the primary entry points into each area — most have supporting
helper functions around them.

### Navigation & Shell
- `showPage(name)` — switches the active page
- `toggleMobileNav` / `closeMobileNav` — mobile hamburger menu
- `checkHeaderNavFit` — collapses header nav into the mobile menu when it overflows
- `resolveBackendUrl(url)` — normalizes backend-relative URLs (see §6 below)

### Theme
- `toggleTheme` — switches light/dark mode, persists to `localStorage`
- `syncThemeToBackend` — persists the choice to the user's account when logged in
- `updateThemeIcon` — updates the header icon to match the active theme

### Authentication
- `doLogin` / `performRealLogin` — email/password login, stores JWT
- `doRegister` / `performRealRegister` — account creation
- `fetchBackendProfile` — pulls the profile after auth
- `getAuthHeaders` — attaches `Authorization: Bearer <token>` to authenticated calls
- `isReallyLoggedIn` — distinguishes a real backend session from local-only usage
- `doLogout` / `clearAuth` — session teardown

### Scanner
- Barcode capture and manual entry handling
- OCR label scan requests to the `/ocr/*` endpoints (backed by
  `src/ocr_label_scanner.py`)
- Voice input: `toggleVoice`, `startVoice`, `stopVoice`, `wordsToDigits` — speech
  recognition with a spoken-digit-to-numeral parser for reading out a barcode

### Product / Scoring
- Score and grade rendering, `buildHeroScoreHTML`
- Confidence and data-source badge rendering
- "Better alternatives" fetch and render (calls `/similar/{barcode}`)
- Share-card generation

### History
- `loadHistory` / `addToHistory` — local scan history (device-level, works offline)
- `logScanToBackendHistory` — pushes a scan to the backend when authenticated
- `renderWeeklyPanel` / `fetchWeeklySummaryFromBackend` — weekly digest
- Monthly report fetch/render (backend `GET /monthly-report`)

### Preferences
- `loadPrefs` / `savePrefs` / `resetPrefs` — dietary preference chip state
- `renderPrefStrip` / `syncPrefToggles` — UI sync for the preference chips

### Compare
- Compare-tray state, add/remove products, side-by-side render
- Calls `GET /compare/{barcode1}/{barcode2}` and `POST /compare-multiple`

### Monthly Report
- Chart rendering for grade distribution / trend over the month
- `renderBarChart` — simple bar-chart renderer reused for both the profile activity
  chart and report views

### Favorites
- `loadFavorites` / `saveFavoritesList` / `toggleFavorite` / `isInFavorites`
- `syncFavoriteAddToBackend` / `syncFavoriteRemoveToBackend` /
  `fetchFavoritesFromBackend` — local/backend sync
- `renderFavoritesPanel` / `removeFavorite` / `clearAllFavorites`

### Authentication-adjacent: Local ↔ Backend Sync
- `importLocalScanHistory` — after login, pushes any history/favorites gathered
  while the user was browsing anonymously up to their new account

### Navigation shortcuts / Gamification
- Challenges list/join/progress rendering
- Leaderboard fetch/render
- Badges panel rendering

### Settings
- `renderSettingsPage`, notification-preference toggles, data-clearing actions,
  `logoutFromSettings`

### Charts
- `renderBarChart` and related small chart helpers used across Profile, History,
  and Monthly Report — implemented as lightweight custom rendering rather than a
  charting library, to avoid adding a dependency for a handful of simple bar/line
  visuals

### AI Chat
- `toggleChatWindow`, `sendChatMessage` — chat UI and calls to `POST /chat`,
  including passing the currently-viewed product as context

---

## 5. `static/style.css` — Structure

- **Design tokens** (`:root` custom properties) — color palette, spacing radius
  tokens, shadow tokens, transition timing. See `04_DESIGN_DECISIONS.md` for the
  reasoning behind the palette.
- **Dark mode** — a second token set under `[data-theme="dark"]` that overrides the
  same variable names, so components never need dark-mode-specific rules — they
  simply reference the token and the value swaps.
- **Layout primitives** — container widths, page/panel structure, responsive
  breakpoints.
- **Components** — buttons, cards, badges/pills (grade, confidence, data-source),
  chips (preferences), nav bar, chat window, modals/overlays.
- **Score & grade styling** — dedicated rules for the hero score display and the
  grade-color system (flat, no gradients — see Design Decisions).

---

## 6. Backend Connection

The frontend is backend-agnostic about where it's hosted, controlled by a single
constant near the top of `script.js`:

```js
const BACKEND_OVERRIDE_URL = 'https://swapify-3.onrender.com';
const BACKEND_BASE_URL = BACKEND_OVERRIDE_URL || window.location.origin;
```

- If the frontend is served **by the same FastAPI app** (or behind the same reverse
  proxy) as the backend, `BACKEND_OVERRIDE_URL` can be left `null` and the app falls
  back to same-origin requests.
- If the frontend is hosted **separately** from the backend (current production
  setup: backend deployed on Render), `BACKEND_OVERRIDE_URL` points at the live
  backend URL, and every fetch in `script.js` is built from `BACKEND_BASE_URL`.
- `resolveBackendUrl()` additionally rewrites root-relative URLs the backend returns
  (e.g. product image paths like `/product-images/_placeholder.svg`) so they resolve
  against the backend's origin rather than the frontend's, which matters whenever the
  two are on different hosts/ports.

This is the **only line that needs to change** to point the whole app at a different
backend — see `03_SETUP_AND_HANDOVER.md`.

---

## 7. Backend Code (for context)

While this document is frontend-focused, the backend files it talks to are:

- `src/app.py` — all FastAPI routes, JWT auth, SQLite access, scoring logic, and
  Open Food Facts fallback (see `06_FRONTEND_API_INTEGRATION.md` for the endpoint
  reference)
- `src/ocr_label_scanner.py` — label OCR support (Tesseract + Pillow), used by the
  Scanner page's OCR mode
- `src/category_taxonomy.py` — the shared rules that classify a product into a
  category, which is what keeps "Better Alternatives" comparisons sensible (a
  noodle product is never suggested as a swap for a sauce)
- `src/observability.py` — logging/error-reporting hooks