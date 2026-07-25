# UI Implementation Documentation

**Project:** Swapify — Frontend
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

---

## 1. Project Overview

Swapify is a health-conscious grocery-shopping companion. A user scans a product's
barcode (or a photo of its label via OCR), and the app returns a health score out of
10, a letter grade (A–F), and — where a product scores poorly — healthier alternatives
in the same category. Around that core "scan → score → swap" loop, the app builds out
history tracking, dietary preferences, comparisons, gamification (challenges and a
leaderboard), a monthly nutrition report, and an in-app AI nutrition assistant.

The entire frontend is a single-page application: one `index.html` shell containing
every screen as a hidden/visible `<div class="page">`, one global stylesheet
(`style.css`), and one application-logic file (`script.js`) that talks to a FastAPI
backend over REST. There is no build step and no frontend framework — navigation,
rendering, and state are handled with vanilla JavaScript (see
`02_PROJECT_STRUCTURE.md` for the technical breakdown).

This document describes **what has been built** — screen by screen — and how each
screen maps to the pages Rashi and the mentor designed together.

---

## 2. Pages Implemented

### 2.1 Home

**Purpose**
The landing screen and hub for the whole app. Surfaces a quick way into scanning,
a snapshot of the user's recent activity, and shortcuts into the rest of the app.

**Features**
- Primary "Scan" call-to-action button that opens the Scanner page
- Personalized home feed (recently scanned products, recommended products)
- Quick-access strip for preferences (chips for e.g. vegetarian, low-sugar, no-added-sugar)
- Header shortcuts to Weekly/Monthly reports, My Swaps, Categories, Challenges,
  Leaderboard, Shopping List and Settings
- Authentication entry point (login/register modal) and header auth state
- Dark mode toggle
- Floating "Compare" button and floating AI Chat button, both accessible from Home

**Files involved:** `index.html` (`#page-home`), `script.js` (home-feed rendering,
auth rendering, theme handling), `style.css` (hero/home layout rules)

---

### 2.2 Scanner

**Purpose**
Lets the user capture a product's barcode using the device camera, or enter/upload
a label for OCR-based recognition when a clean barcode scan isn't possible.

**Features**
- Live camera-based barcode scanning
- Manual barcode entry fallback
- OCR label scanning (photograph a nutrition label and have it parsed automatically)
- Voice input for barcode/product search (speech-to-text with a digit-word parser)
- Loading and error states while a scan is resolving against the backend
- On a successful scan, the user is routed straight to the Product page with the result

**Files involved:** `index.html` (`#page-scanner`), `script.js` (scanning logic,
`ocr_label_scanner.py`-backed OCR calls, voice input functions), `src/ocr_label_scanner.py`
(backend OCR support)

---

### 2.3 Product Detail

**Purpose**
The most important screen in the app — this is where a scan resolves into an
actionable health verdict for a single product.

**Features**

- **Final Score** — a prominent 0–10 health score for the scanned product
- **Score Breakdown** — the factors that make up the score (nutrients, additives,
  category-relative comparison) so the number isn't a black box
- **Confidence Badge** — indicates how reliable the underlying data is, so the user
  knows when a score is well-supported versus based on partial data
- **Data Source Badge** — shows whether the product came from Swapify's own curated
  catalogue or was resolved via the Open Food Facts fallback
- **Better Alternatives** — healthier products in the same category, shown
  immediately below the primary product so a poor score is followed by a solution,
  not left as a dead end
- **Favorite** — save the product to the user's Favorites list
- **Share** — generate a shareable summary card for the product
- **Compare** — add the product to the Compare tray to weigh it against others
- **AI Chat** — the floating AI Nutritionist can be asked about the currently viewed
  product directly (the chat window shows a context pill naming the product)
- **Community Ratings & Reviews** — user-submitted ratings, written reviews, and
  reply/voting on reviews for the product
- **Product Images** — the product's image, with a placeholder graphic when no image
  is available
- **How This Works** — a dedicated explainer page (`#page-how-scoring-works`) the user
  can open from the Product page to understand how the health score is calculated

**Files involved:** `index.html` (`#page-product`, `#page-how-scoring-works`),
`script.js` (score rendering, alternatives fetch, favorites/compare/share logic,
ratings & reviews rendering, chat context binding), `style.css` (score hero,
grade pill, badge and card styling)

---

### 2.4 History

**Purpose**
Gives the user a timeline of everything they've scanned, plus rolled-up views of
their eating pattern over time.

**Features**
- Chronological scan history list (grade, name, score, timestamp per entry)
- **Weekly summary panel** — a compact digest of the current week's scans
- **Monthly Report panel** — trends, grade distribution, and progress over the month
  (see 2.13 below — the Monthly Report lives inside History rather than as a
  separate bottom-nav destination)
- Clear-history option (also available from Settings)

**Files involved:** `index.html` (`#page-history`, `#weeklyPanelPage`,
`#monthlyPanelPage`), `script.js` (`renderWeeklyPanel`, weekly/monthly summary fetch
and render functions)

---

### 2.5 Preferences

**Purpose**
Lets the user tell the app about dietary restrictions/goals so that scoring context,
recommendations, and better-alternative suggestions can be personalized.

**Features**
- Toggle-able dietary preference chips (e.g. vegetarian, low-sugar, high-protein,
  no added sugar — rendered as rounded pills)
- Preferences persist locally and sync to the backend when logged in
- Reset-to-default option

**Files involved:** `index.html` (`#page-preferences`), `script.js` (`loadPrefs`,
`savePrefs`, `resetPrefs`, `renderPrefStrip`, `syncPrefToggles`)

---

### 2.6 Categories

**Purpose**
Lets the user browse the catalogue by product category rather than by scanning,
useful for pre-shopping research.

**Features**
- Grid/list of product categories
- Drill-in to see products and their grades within a category

**Files involved:** `index.html` (`#page-categories`), `script.js` (category
rendering, backed by `category_taxonomy.py` on the backend)

---

### 2.7 Leaderboard

**Purpose**
A gamified, social layer that ranks users by healthy-scanning activity to encourage
engagement.

**Features**
- Ranked list of users by score/points
- Highlights the current user's own rank

**Files involved:** `index.html` (`#page-leaderboard`), `script.js` (leaderboard
fetch/render), backend `GET /leaderboard`

---

### 2.8 Challenges

**Purpose**
Time-boxed goals (e.g. "scan 5 low-sugar products this week") that reward the user
for healthier shopping habits.

**Features**
- List of active challenges with progress indicators
- Join-challenge action
- Per-challenge progress tracking

**Files involved:** `index.html` (`#page-challenges`), `script.js` (challenge
list/join/progress rendering), backend `GET /challenges`,
`POST /challenges/{id}/join`, `GET /challenges/{id}/progress`

---

### 2.9 Settings

**Purpose**
Central place for account, notification, data, and appearance controls.

**Features**
- Dark mode switch (mirrors the header theme toggle, kept in sync with the backend)
- Notification preferences (daily reminder, challenge alerts, weekly digest)
- Clear scan history / clear favorites / reset preferences
- Logout
- Import of any locally-stored scan history into the user's backend account after login

**Files involved:** `index.html` (`#page-settings`), `script.js`
(`renderSettingsPage`, `toggleNotifPref`, `syncDigestPrefToBackend`,
`clearScanHistorySettings`, `clearFavoritesSettings`, `resetPreferencesSettings`,
`logoutFromSettings`, `importLocalScanHistory`)

---

### 2.10 Monthly Report

**Purpose**
A rolled-up, chart-driven view of the user's scanning and health trends across the
month.

**Features**
- Visual breakdown of grades scanned over the month
- Trend indicators (improving/declining pattern)
- Rendered inline within the History page

**Files involved:** `index.html` (`#monthlyPanelPage`, inside `#page-history`),
`script.js` (monthly report fetch/render), backend `GET /monthly-report`

---

### 2.11 Profile

**Purpose**
The user's personal dashboard — a snapshot of their account and activity plus quick
links into related screens.

**Features**
- Profile summary (name, lifetime scan count, badges)
- Quick-link buttons into Challenges, Leaderboard, Preferences, and Settings
- Recent scans and a simple bar chart of activity
- Badges panel

**Files involved:** `index.html` (`#page-profile`), `script.js`
(`renderProfilePanel`, `renderBarChart`, `renderRecentScans`,
`syncProfileTotalScansFromBackend`), backend `GET /profile`, `GET /badges`

---

### 2.12 Favorites

**Purpose**
A saved list of products the user wants to come back to.

**Features**
- List of favorited products with grade/score
- Remove individual favorite / clear all
- Favorites sync between local storage and the backend once the user is logged in

**Files involved:** `index.html` (`#page-favorites`), `script.js`
(`toggleFavorite`, `renderFavoritesPanel`, `removeFavorite`, `clearAllFavorites`,
`fetchFavoritesFromBackend`)

---

### 2.13 My Swaps

**Purpose**
Tracks the healthier-alternative swaps a user has actually chosen to make, as a
personal record of progress (distinct from the auto-generated "Better Alternatives"
suggestions on the Product page).

**Features**
- List of saved swaps (original product → chosen alternative)
- Optional note per swap
- Remove a saved swap

**Files involved:** `index.html` (`#page-swaps`), `script.js` (My Swaps rendering),
backend `POST /my-swaps`, `GET /my-swaps`, `DELETE /my-swaps/{original}/{alt}`,
`POST /my-swaps/note`

---

### 2.14 AI Chat

**Purpose**
An always-available AI Nutritionist the user can ask about the product they're
currently viewing, or general nutrition questions.

**Features**
- Floating chat button (FAB) present across the app, not tied to a single page
- Context pill showing which product (if any) the conversation is currently about
- Quick-reply substitution chips (e.g. "substitute for sugar")
- Empty-state guidance for first-time use

**Files involved:** `index.html` (`.chat-fab`, `#chatWindow`, `#chatMessages`,
`#chatContextPill`), `script.js` (`toggleChatWindow`, `sendChatMessage`), backend
`POST /chat`

---

### 2.15 Additional supporting screens

- **Compare** (`#page-compare`) — side-by-side comparison of up to several products,
  reached via a floating Compare button or from the Product page. Backed by
  `GET /compare/{barcode1}/{barcode2}` and `POST /compare-multiple`.
- **Shopping List / Cart** (`#page-cart`) — a shopping list the user can build and
  optimize toward healthier picks. Backed by `POST /shopping-list`,
  `GET /shopping-list/mine`, `GET /shopping-list/{id}/optimize`.
- **How Scoring Works** (`#page-how-scoring-works`) — a static explainer reached from
  the Product page, describing the scoring methodology in plain language.

---

## 3. Responsive Design

The layout is built mobile-first, since the primary use case is a shopper scanning
products in-store on a phone, and scales up for larger viewports.

**Desktop**
Full header navigation is shown; supplementary panels (weekly/monthly summaries,
comparisons) can render as wider multi-column layouts.

**Tablet**
Layout retains the single-column "app" feel but with larger touch targets and more
breathing room than mobile; header navigation collapses less aggressively than on
phone.

**Mobile**
The primary target. Bottom navigation bar (Home, History, Scan, Favorites, Profile)
sits within thumb reach; secondary navigation (Categories, Challenges, Leaderboard,
Shopping List, Settings, Weekly/Monthly) is tucked into a collapsible mobile nav
menu (`toggleMobileNav`/`closeMobileNav`) so the primary screen stays uncluttered.

---

## 4. Animations

Animation is used sparingly and purposefully, in line with the "subtle only" design
decision documented in `04_DESIGN_DECISIONS.md`:

- **Hover** — cards, buttons, and nav items lift or shift color slightly on hover to
  signal interactivity without being distracting
- **Transitions** — page switches, panel open/close, and modal open/close use short
  eased transitions (`--transition: 0.22s cubic-bezier(.4,0,.2,1)`) rather than hard
  cuts
- **Button animations** — primary buttons (Scan, Save, Send) give a brief pressed/
  active state for tactile feedback
- **Score animation** — the health score on the Product page animates in (count-up /
  reveal) when a scan resolves, drawing the eye to the single most important number
  on the screen
- **Toasts** — non-blocking toast notifications (`showToast`) animate in/out for
  confirmations (e.g. "Added to favorites") without interrupting the flow