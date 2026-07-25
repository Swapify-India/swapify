# Frontend ↔ API Integration

**Project:** Swapify
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

This document lists every backend endpoint the frontend (`static/script.js`) calls,
what it's for, and how the response is used in the UI. For backend-side gotchas,
response-shape quirks, and deployment status, see `FRONTEND_INTEGRATION.md` in the
repository root (written by Dhruv, backend) — this document complements that one
from the frontend's perspective.

**Base URL:** configurable via `BACKEND_OVERRIDE_URL` in `script.js` (see
`03_SETUP_AND_HANDOVER.md`). Production: `https://swapify-3.onrender.com`.

**Auth:** authenticated calls send `Authorization: Bearer <token>`, attached via
`getAuthHeaders()`. The token is issued by `/login` and stored client-side.

---

## Authentication

### `POST /register`
**Request**
```json
{ "email": "user@example.com", "password": "Test1234!", "username": "rashi" }
```
**Response**
```json
{ "message": "User registered successfully" }
```
**Frontend usage:** Called from `performRealRegister`, triggered by the register
form in the auth modal. `username` is required — omitting it returns `422`.

### `POST /login`
**Request**
```json
{ "email": "user@example.com", "password": "Test1234!" }
```
**Response**
```json
{ "access_token": "eyJ...", "token_type": "bearer" }
```
**Frontend usage:** Called from `performRealLogin`. The token is saved via
`saveAuth`, attached to all subsequent authenticated requests, and triggers
`importLocalScanHistory` to sync any anonymous local activity into the account.

### `GET /profile`
**Response:** user id, username, email, join date, theme preference.
**Frontend usage:** `fetchBackendProfile` populates the Profile page and header
auth state after login/reload.

### `POST /theme`
**Frontend usage:** `syncThemeToBackend` — persists the light/dark mode choice to
the account so it follows the user across devices.

---

## Product Scanning & Scoring

### `GET /product/{barcode}?device_id=<optional>`
**Response (abridged):**
```json
{
  "product_name": "Kitkat",
  "brand": "Nestle",
  "grade": "F",
  "score": 1.0,
  "source": "swapify" | "openfoodfacts",
  "barcode_matched_on": { "...": "present only on a fallback catalogue match" }
}
```
**Frontend usage:** Core Scanner → Product Detail flow. Drives the Final Score,
Score Breakdown, and grade pill. `source` drives the Data Source Badge. A `200`
does not guarantee the product is in Swapify's own catalogue — check `source`
before treating it as curated data. A true miss returns `404`.

### `GET /validate-barcode/{barcode}`
**Frontend usage:** Used for manual barcode entry, to check the format/check digit
before submitting a lookup, and to offer a corrected suggestion if available.

### `GET /score/{barcode}` · `GET /v2/score/{barcode}`
**Frontend usage:** Standalone score lookups, used where only the score/grade is
needed without the full product payload (e.g. re-scoring after a preference change).

### `GET /similar/{barcode}`
**Response:** array of alternative products with `barcode`, `product_name`,
`brand`, `health_score`, `grade`, `image_url`.
**Frontend usage:** Populates the **Better Alternatives** section on Product
Detail, filtered to the same product category.

### `GET /product/{barcode}/badge`
**Frontend usage:** Confidence Badge data for a product.

### `POST /product/image`
**Frontend usage:** Uploads/associates an image with a product record (used when a
user contributes a product photo).

### `POST /report-missing`
**Frontend usage:** "Report a product we don't have" action, surfaced when a scan
returns `404`.

---

## Search & Categories

### `GET /search?q=&limit=&offset=&brand=&category=&grade=&min_score=&max_score=&sort=`
**Response:** a **bare JSON array** (not wrapped in an object); each item uses the
key `name`, not `product_name`.
**Frontend usage:** Powers in-app product search, distinct from the barcode/OCR
scan flow.

### `GET /search/autocomplete?q=`
**Frontend usage:** Type-ahead suggestions in the search input.

### `GET /categories`
**Frontend usage:** Populates the Categories page listing.

---

## History, Favorites, Preferences

### `POST /scan-history` · `POST /scan-history/import`
**Frontend usage:** `logScanToBackendHistory` writes a scan to the backend once
authenticated; `/scan-history/import` is used by `importLocalScanHistory` to bulk
-upload history captured while the user was logged out.

### `GET /history`
**Response:** the authenticated user's scanned products, newest first.
**Frontend usage:** Renders the History page's scan list. Requires a Bearer token —
returns `401` without one.

### `GET /preferences` · `POST /preferences` · `POST /update-preferences`
**Frontend usage:** Loads and persists the user's dietary preference chips
(`loadPrefs`/`savePrefs`), synced between `localStorage` and the backend.

### `POST /favorites` · `DELETE /favorites/{barcode}` · `GET /favorites`
**Frontend usage:** Favorites page and the favorite toggle on Product Detail.
Requires a Bearer token; `toggleFavorite` manages the optimistic local state while
`syncFavoriteAddToBackend`/`syncFavoriteRemoveToBackend` keep the backend in sync.

---

## My Swaps

### `POST /my-swaps` · `GET /my-swaps` · `DELETE /my-swaps/{original}/{alt}` · `POST /my-swaps/note`
**Frontend usage:** The My Swaps page — records swaps a user has actually made
(original product → chosen alternative), with an optional personal note per swap.

---

## Compare

### `GET /compare-list` · `POST /compare-list` · `DELETE /compare-list/{barcode}` · `DELETE /compare-list`
**Frontend usage:** Manages which products are queued in the Compare tray.

### `GET /compare/{barcode1}/{barcode2}`
**Frontend usage:** Two-product side-by-side comparison view.

### `POST /compare-multiple`
**Frontend usage:** Multi-product comparison (Compare page, more than two items).

---

## Reports & Activity

### `GET /weekly-summary`
**Frontend usage:** Weekly digest panel on the History page
(`fetchWeeklySummaryFromBackend` → `_renderWeeklyPanelCore`).

### `GET /monthly-report`
**Frontend usage:** Monthly Report panel embedded in the History page — grade
distribution and trend data.

### `GET /recent`
**Frontend usage:** Recently-viewed/scanned quick list, used on Home.

### `GET /home-feed`
**Frontend usage:** Populates the personalized Home page feed (recent + recommended
products).

### `GET /recommendations`
**Frontend usage:** Backs recommendation surfaces beyond the home feed (e.g. within
Categories/related-product contexts).

### `POST /activity` · `GET /activity/user/{user_id}` · `GET /activity/trends`
**Frontend usage:** Activity logging used to back the Profile page's recent-activity
list and bar chart (`renderBarChart`, `renderRecentScans`).

### `GET /digest/{user_id}` · `POST /digest/preference` · `GET /digest/preference`
**Frontend usage:** Weekly digest notification preference, managed from Settings.

---

## Ratings, Reviews & Sharing

### `POST /rate-product` · `GET /product/{barcode}/ratings` · `GET /user/ratings`
**Frontend usage:** Community Ratings section on Product Detail.

### `POST /reviews` · `GET /reviews/{review_id}` · `GET /product/{barcode}/reviews` · `DELETE /reviews/{review_id}` · `POST /reviews/{review_id}/vote` · `POST /reviews/{review_id}/replies`
**Frontend usage:** Reviews section on Product Detail — writing, reading, deleting,
upvoting, and replying to reviews.

### `GET /share/{barcode}`
**Frontend usage:** Generates the shareable summary used by the Share action on
Product Detail.

---

## Gamification

### `GET /badges`
**Frontend usage:** Badges panel on the Profile page.

### `GET /challenges` · `POST /challenges/{id}/join` · `GET /challenges/{id}/progress`
**Frontend usage:** Challenges page — listing, joining, and tracking progress.

### `GET /leaderboard`
**Frontend usage:** Leaderboard page ranking.

---

## Shopping List

### `POST /shopping-list` · `GET /shopping-list/mine` · `GET /shopping-list/{id}` · `GET /shopping-list/{id}/optimize` · `POST /shopping-list/{id}/replace` · `DELETE /shopping-list/{id}`
**Frontend usage:** Shopping List / Cart page — creating a list, viewing it,
running the "optimize toward healthier picks" action, swapping an item, and
deletion.

---

## AI Chat

### `POST /chat`
**Request:** user message, plus (optionally) the barcode of the product currently
being viewed, for context.
**Frontend usage:** `sendChatMessage` — powers the AI Nutritionist chat window,
including the context pill that shows which product the conversation relates to.

---

## OCR

### `GET /ocr/health`
**Frontend usage:** Checked before enabling the OCR scan option, so the UI can
gracefully hide/disable OCR if the backend's OCR engine isn't available.

### `POST /ocr/scan-label`
**Frontend usage:** Uploads a photographed label from the Scanner page's OCR mode
and receives parsed product/nutrition data back.

---

## System / Diagnostics

### `GET /health` · `GET /ping` · `GET /product-count` · `GET /cache-stats`
**Frontend usage:** `retryBackendConnection` and the sync-status banner
(`renderSyncBanner`) use `/health` to detect whether the backend is reachable and
show a reconnect banner if not.

### `GET /offline-products`
**Frontend usage:** Reserved for offline-catalogue support (see Future
Improvements in `05_FEATURE_STATUS.md` — not yet wired into a full offline mode).

---

## Not Called by the Frontend (backend-internal / admin)

- `POST /experiment/log-scan`, `GET /experiment/logs`, `GET /experiment/analytics`,
  `POST /admin/cache-clear`, `POST /debug/sentry-test` — used for field-testing
  instrumentation and admin diagnostics, not part of the consumer-facing UI.

---

## Notes on Request Conventions

- Authenticated endpoints expect `Authorization: Bearer <token>`. A `device_id`
  query parameter is **not** authentication — it identifies an anonymous device for
  history purposes only.
- Barcodes are sent to the backend exactly as captured by the scanner (raw digits,
  no client-side cleanup); the backend handles matching, including a fallback for a
  small number of catalogue entries with a check-digit/payload mismatch.
- `/search` responses are a bare array with a `name` key, while `/product/{barcode}`
  responses use `product_name` — these are **not interchangeable** and are handled
  with separate parsing logic in `script.js`.