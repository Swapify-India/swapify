# Feature Status

**Project:** Swapify
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

Status of every feature built into the frontend, cross-checked against the current
codebase (`static/index.html`, `static/script.js`, `src/app.py`).

| Feature | Status | Notes |
| --- | --- | --- |
| Barcode Scanner | Complete | Live camera scan + manual barcode entry |
| OCR Label Scanning | Complete | Backed by `src/ocr_label_scanner.py`; degrades gracefully if Tesseract isn't installed on the host |
| Voice Input | Complete | Speech-to-text with spoken-digit parsing for hands-free barcode entry |
| Product Scoring (Final Score) | Complete | 0–10 score + A–F grade per product |
| Score Breakdown | Complete | Shows the factors behind the score |
| Confidence Badge | Complete | Signals data reliability per product |
| Data Source Badge | Complete | Distinguishes Swapify catalogue vs. Open Food Facts fallback |
| Better Alternatives | Complete | Category-aware swap suggestions via `/similar/{barcode}` |
| Favorites | Complete | Local + backend sync |
| Share | Complete | Shareable product summary card |
| Compare | Complete | Side-by-side comparison, up to multiple products |
| Community Ratings | Complete | Star/numeric ratings per product |
| Reviews | Complete | Written reviews, replies, and voting |
| Product Images | Complete | With placeholder graphic fallback |
| "How Scoring Works" Explainer | Complete | Static page reached from Product Detail |
| Preferences (Dietary) | Complete | Toggle chips, local + backend sync |
| Scan History | Complete | Full history list + weekly summary panel |
| Monthly Report | Complete | Trend/grade-distribution charts, rendered inside History |
| Categories | Complete | Browse catalogue by category |
| Leaderboard | Complete | Ranked user activity |
| Challenges | Complete | Join + progress tracking |
| Profile | Complete | Summary, badges, recent activity, quick links |
| My Swaps | Complete | Personal record of chosen swaps, with notes |
| Shopping List / Cart | Complete | Build a list; optimize toward healthier picks |
| Settings | Complete | Notifications, data clearing, logout |
| Dark Mode | Complete | Full token-based theme, synced to backend |
| Theme Sync | Complete | Theme preference persists across sessions/devices when logged in |
| Authentication (Login/Register) | Complete | JWT-based; local-only fallback when logged out |
| Local → Backend Sync on Login | Complete | Anonymous scan history imports into the account after login |
| Quick Scan (Home shortcut) | Complete | One-tap entry into the Scanner from Home |
| AI Chat (AI Nutritionist) | Complete | Context-aware chat FAB, available on every page |
| Responsive Layout (Mobile/Tablet/Desktop) | Complete | Mobile-first, bottom nav + collapsible secondary nav |
| Admin/Experiment Logging Hooks | Complete (backend) | `/experiment/*` endpoints used for real-world scan-testing data collection, not a user-facing screen |

---

## Future Improvements

Ideas noted during development but intentionally out of scope for this phase:

- **Push notifications** for challenge deadlines and weekly digests (currently
  digest/notification *preferences* are captured in Settings, but delivery is
  in-app only — no browser/mobile push yet)
- **Offline-first scanning** — currently relies on `localStorage` for history/
  favorites while offline, but the product catalogue itself isn't cached for full
  offline lookup (there is a `GET /offline-products` endpoint on the backend that
  could be built on for this)
- **Richer charting** on the Monthly Report (the current implementation uses
  lightweight custom bar rendering rather than a full charting library — sufficient
  for now, but could be extended with more chart types as data volume grows)
- **Social features beyond the Leaderboard** — e.g. following friends' swaps,
  sharing challenges directly with another user
- **Multi-language support** — the UI is currently English-only
- **Accessibility pass** — a dedicated audit for screen-reader labeling and keyboard
  navigation across all pages, beyond what's been done ad hoc during build
- **Automated tests** for the frontend (currently no test suite covers `script.js`;
  correctness has been verified manually against the live backend)