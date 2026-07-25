# Swapify — Frontend Documentation

**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

---

## Overview

Swapify is a health-conscious grocery-shopping companion. A user scans a product's
barcode (or photographs its label for OCR), and gets back a 0–10 health score, an
A–F grade, and — where relevant — healthier alternatives in the same category. The
frontend is a single-page application (one HTML shell, one stylesheet, one script
file, no build step) that talks to a FastAPI + SQLite backend over REST.

For full detail on any of the topics below, see the corresponding document in this
folder.

---

## Features

- Barcode scanning (camera) and OCR label scanning
- Voice input for barcode/search entry
- Product health scoring: final score, score breakdown, confidence badge, data-
  source badge, and category-aware "Better Alternatives"
- Favorites, Compare, and Share
- Community ratings and reviews per product
- Scan history, weekly summary, and a monthly report with trend charts
- Dietary preferences (toggleable, synced to account)
- Categories browsing
- Gamification: Challenges, Leaderboard, Badges
- My Swaps — a personal record of chosen product swaps
- Shopping List with a "healthier picks" optimizer
- In-app AI Nutritionist chat (context-aware, available from any page)
- Authentication (JWT-based), with local-only fallback while logged out and
  automatic sync of anonymous activity into the account after login
- Dark mode, synced across devices when logged in
- Fully responsive: mobile-first, with tablet and desktop layouts

See `05_FEATURE_STATUS.md` for the full feature-by-feature status table.

---

## Folder Structure

```
Swapify
│
├── docs/
│   ├── 01_UI_IMPLEMENTATION.md
│   ├── 02_PROJECT_STRUCTURE.md
│   ├── 03_SETUP_AND_HANDOVER.md
│   ├── 04_DESIGN_DECISIONS.md
│   ├── 05_FEATURE_STATUS.md
│   ├── 06_FRONTEND_API_INTEGRATION.md
│   └── README_FRONTEND.md
├── static/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── swapify_products.csv
├── src/
│   ├── app.py
│   ├── ocr_label_scanner.py
│   ├── category_taxonomy.py
│   └── observability.py
├── uploads/product_images/        Runtime-uploaded product photos
├── swapify.db
├── requirements.txt
├── API_DOCS.md                    Backend API reference (Dhruv)
├── FRONTEND_INTEGRATION.md        Backend→frontend integration notes (Dhruv)
├── DEPLOYMENT_FRONTEND.md         Frontend deployment notes (Rashi)
├── PERFORMANCE_REPORT.md          Backend performance/load-test report (Dhruv)
├── sync_db.py                     Ops script: syncs product CSV into the live DB
├── test_api.sh / test_api.ps1     Backend API test scripts (Dhruv)
```

See `02_PROJECT_STRUCTURE.md` for a full breakdown of what lives where and why —
including which items are frontend (this documentation's scope) versus backend/ops
artifacts owned by Dhruv, kept in the diagram only for completeness.

---

## Installation

```bash
git clone <repository-url>
cd Swapify
python -m venv .venv
```

Activate the virtual environment:

```bash
# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Running

From the `src/` directory:

```bash
python app.py
```

or, for auto-reload:

```bash
uvicorn app:app --reload
```

Then open:

```
http://localhost:8000
```

FastAPI serves the `static/` folder directly — there is no separate frontend build
or dev server. Full setup and troubleshooting steps are in
`03_SETUP_AND_HANDOVER.md`.

---

## Live Deployment

| Layer | Platform | URL |
| --- | --- | --- |
| Backend (FastAPI) | Render | https://swapify-3.onrender.com |
| Frontend (static files) | Vercel | https://swapify-three.vercel.app/ |
| Source control | GitHub | github.com/Rashi-123456/swapify |

Frontend and backend are hosted separately, so `BACKEND_OVERRIDE_URL` in
`static/script.js` points explicitly at the Render URL. See
`03_SETUP_AND_HANDOVER.md` for CORS and cold-start notes relevant to this setup.

---

## Screenshots

Not included in this documentation pass — the live app can be walked through
directly at `http://localhost:8000` once the backend is running, or via the
production deployment.

---

## Dependencies

The frontend has no JavaScript dependencies or build tooling — it's plain HTML,
CSS, and vanilla JavaScript. Backend Python dependencies (which the frontend relies
on being installed and running) are listed in `requirements.txt`:
FastAPI, Uvicorn, Requests, bcrypt, PyJWT, Pydantic, python-dotenv, cachetools,
python-multipart, Gunicorn (production only), pytesseract, and Pillow.

---

## Browser Support

Built and tested against current versions of Chrome, Edge, and Firefox. Camera-
based barcode scanning and speech-to-text voice input rely on standard browser Web
APIs, so a modern, up-to-date browser is recommended — older browsers may not
support these features even if the rest of the UI renders correctly.

---

## Known Issues

- OCR label scanning requires the native Tesseract engine to be installed on the
  host machine; if it isn't, the `/ocr/*` endpoints report OCR as unavailable and
  that entry point is hidden/disabled, but the rest of the app is unaffected.
- No offline product catalogue yet — `localStorage` covers offline history/
  favorites, but a fresh product lookup still requires connectivity.
- No automated frontend test suite; correctness has been verified manually against
  the live backend.
- See `FRONTEND_INTEGRATION.md` in the repository root (Dhruv, backend) for known
  API response-shape quirks relevant to any new frontend work.

---

## Future Improvements

See `05_FEATURE_STATUS.md` for the full list, including push notifications,
full offline support, expanded charting, social features beyond the leaderboard,
multi-language support, and a dedicated accessibility pass.

---

## Credits

- **Frontend:** Rashi
- **Backend:** Dhruv
- Product data supplemented via [Open Food Facts](https://world.openfoodfacts.org/)
  where a scanned product isn't in Swapify's own catalogue.