# Setup & Handover Documentation

**Project:** Swapify
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

This is the most important document for anyone picking up the project — it covers
everything needed to get Swapify running locally end to end, and what to know before
touching it.

---

## 1. Requirements

Before you start, make sure you have:

- **Python** 3.10+ (project has been run and tested against Python 3.13)
- A modern **browser** (Chrome, Edge, or Firefox recommended — camera-based barcode
  scanning and speech input rely on standard browser Web APIs)
- **Internet access**, for:
  - installing Python dependencies
  - the Open Food Facts fallback lookup (used when a scanned barcode isn't in
    Swapify's own catalogue)
  - the AI Chat feature (calls an external LLM provider — see §5)
- The **backend running** — the frontend is static HTML/CSS/JS and does nothing
  useful on its own; almost every screen depends on the FastAPI backend responding.

Optional, only if you want OCR label scanning to work locally:
- **Tesseract OCR** installed as a native binary on your machine (the Python side,
  `pytesseract`, is already in `requirements.txt`, but it needs the actual Tesseract
  engine present on the system `PATH`). If Tesseract isn't installed, the app still
  runs fine — the `/ocr/*` endpoints simply report OCR as unavailable and the
  barcode-scanning flow is unaffected.

---

## 2. Getting the Code

```bash
git clone <repository-url>
cd Swapify
```

---

## 3. Backend Setup

### 3.1 Create a virtual environment

```bash
python -m venv .venv
```

### 3.2 Activate it

**Windows**
```bash
.venv\Scripts\activate
```

**macOS / Linux**
```bash
source .venv/bin/activate
```

### 3.3 Install dependencies

```bash
pip install -r requirements.txt
```

This installs FastAPI, Uvicorn, the auth stack (`bcrypt`, `PyJWT`), `pydantic`,
`python-dotenv`, `cachetools`, `python-multipart`, `gunicorn` (used in production
only), and the OCR Python bindings (`pytesseract`, `Pillow`).

### 3.4 Run the backend

From the `src/` directory:

```bash
python app.py
```

or, for auto-reload during development:

```bash
uvicorn app:app --reload
```

(If running from the project root instead of `src/`, use `uvicorn src.app:app
--reload` — the codebase supports both import styles.)

### 3.5 Open the app

```
http://localhost:8000
```

FastAPI serves the `static/` folder directly, so opening the backend's root URL
loads `index.html` and the full app — there is no separate frontend server needed
for local development.

---

## 4. Configuring the Frontend's Backend URL

The frontend needs to know where the backend lives. This is controlled by **one
line** near the top of `static/script.js`:

```js
const BACKEND_OVERRIDE_URL = 'https://swapify-3.onrender.com';
```

- **Running everything locally / same-origin deployment** (frontend served by the
  same FastAPI instance): set this to `null`. The app will fall back to
  `window.location.origin` and call `http://localhost:8000` automatically.
- **Frontend hosted separately from the backend** (e.g. frontend on Netlify/Vercel/
  GitHub Pages, backend on Render/Railway): set this to the backend's full URL, no
  trailing slash.

This is the **only** place a deployment target needs to be changed — every API call
in `script.js` is built from `BACKEND_BASE_URL`, which is derived from this one
constant.

The current production backend is live at:

```
https://swapify-3.onrender.com
```

---

## 5. Backend Configuration Reference

The backend reads its configuration from environment variables (with sane local
defaults), typically via a `.env` file or the hosting platform's environment
settings:

| Variable | Purpose | Local default |
| --- | --- | --- |
| `SECRET_KEY` | Signs JWTs for auth | falls back to a dev constant — **must** be overridden in production |
| `SWAPIFY_DB_PATH` / `DATABASE_PATH` | Path to the SQLite database file | `swapify.db` in the project root |
| `SWAPIFY_CSV_PATH` | Path to the product catalogue seed CSV | `static/swapify_products.csv` |
| `CORS_ORIGINS` | Allowed frontend origins | `*` (open) locally |
| `CORS_ORIGIN_REGEX` | Regex fallback for allowed origins | preset to always allow local dev origins |
| `ADMIN_TOKEN` | Protects the `/experiment/logs` and `/experiment/analytics` admin endpoints | dev placeholder — **must** be overridden in production |
| `OPENROUTER_API_KEY` / `GEMINI_API_KEY` | Credentials for the AI Chat / AI Nutritionist feature | unset locally disables live AI responses |

You do not need to set any of these to get the app running locally with a fresh
SQLite database — the defaults are sufficient for development. They matter for a
real deployment (see §7).

---

## 6. What Depends on the Backend

If the backend is **not running**, the following will not work, even though the
static frontend will still load:

- **Scanner** — barcode/OCR lookups fail (no product data to resolve against)
- **Product Detail** — no score, no alternatives, no ratings/reviews
- **History** — nothing to fetch; weekly/monthly summaries stay empty
- **Favorites** — can partially work offline via `localStorage`, but will not sync
  or persist across devices
- **My Swaps, Compare, Categories, Leaderboard, Challenges, Shopping List** — all
  require live backend data
- **Login / Registration** — no auth server to talk to
- **AI Chat** — no backend endpoint to answer questions
- **Monthly Report** — no data to summarize

In short: the frontend is a shell without the backend. Always confirm the backend
is reachable (`GET /health` should return `200`) before debugging a "blank" screen.

---

## 7. Data Sources

- **SQLite (`swapify.db`)** — the primary datastore: users, scan history,
  favorites, preferences, reviews/ratings, challenges, leaderboard, shopping lists,
  and the product catalogue once seeded.
- **CSV seed (`static/swapify_products.csv`)** — the initial product catalogue used
  to populate the database on first run/import.
- **OCR (Tesseract via `pytesseract`)** — parses a photographed nutrition label into
  structured data when a barcode scan isn't possible or the barcode isn't found.
- **Open Food Facts** — an external product database the backend falls back to when
  a scanned barcode isn't present in Swapify's own catalogue. A response in this
  case still returns `200`, but is tagged `"source": "openfoodfacts"` rather than
  Swapify's own data — see `06_FRONTEND_API_INTEGRATION.md`.

---

## 8. Live Deployment

| Layer | Platform | URL |
| --- | --- | --- |
| Backend (FastAPI) | Render | `https://swapify-3.onrender.com` |
| Frontend (static files) | Vercel | `https://swapify-three.vercel.app/` |
| Source control | GitHub | `github.com/Swapify-India/swapify` |

The frontend and backend are hosted **separately** (Vercel + Render), which is why
`BACKEND_OVERRIDE_URL` in `static/script.js` is set to the Render URL rather than
left as `null` — a same-origin fallback wouldn't work across two different hosts.

Two things to keep in mind with this split-hosting setup:

- **CORS** — the Render backend's `CORS_ORIGINS` (and/or `CORS_ORIGIN_REGEX`) must
  explicitly allow the Vercel domain, or authenticated requests from the live
  frontend will fail even though everything works locally.
- **Render cold starts** — the free tier spins down when idle; the first request
  after a period of inactivity can take 30–50 seconds. A keep-alive ping to
  `GET /health` helps, but don't assume a slow first load on the live site is a bug.

See `DEPLOYMENT_FRONTEND.md` in the repository root for the original frontend
deployment notes (file list, hosting options considered, CORS reminder).

---

## 9. Handover Notes

- The **frontend is a single HTML/CSS/JS file set with no build step** — there is
  nothing to compile, bundle, or transpile. Edit `static/index.html`, `style.css`,
  or `script.js` directly and refresh the browser.
- The backend and frontend are decoupled by design — the one-line
  `BACKEND_OVERRIDE_URL` switch in `script.js` is the entire "deployment
  configuration" from the frontend side.
- For anything relating to backend deployment status, database schema, or API
  behavior quirks, see the backend team's own handover notes
  (`FRONTEND_INTEGRATION.md` in the repository root, written by Dhruv). It documents
  known API response shapes, authentication requirements, and a few gotchas worth
  reading before wiring up new screens against the live API.
- If you hit a screen that looks broken, check in this order: (1) is the backend
  running / is `GET /health` reachable, (2) is `BACKEND_OVERRIDE_URL` pointing at the
  right place, (3) is the user actually authenticated for endpoints that require it
  (`/history`, `/favorites`, `/preferences`, `/my-swaps`, etc. all require a Bearer
  token).