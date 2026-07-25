# Design Decisions Documentation

**Project:** Swapify
**Author:** Rashi (Frontend Developer)
**Last Updated:** July 2026

This document captures the design reasoning that existed only in conversation or
in my head while building Swapify — the "why" behind decisions that aren't obvious
from reading the code alone. It's written so that anyone extending the UI later
understands the intent, not just the implementation.

---

## 1. Color Palette

**Teal / deep navy-teal as the primary brand color, lime as the accent.**

```
--navy: #0B3B3A        (primary dark tone)
--navy-light: #0E7C6E  (mid teal)
--accent: #1CB399      (primary interactive accent)
--lime: #C6F135        (secondary accent / highlight)
```

**Reason:** Teal/green tones read immediately as "health" and "freshness" without
being as generic as pure green, which is heavily overused in health-app UIs. Lime is
used sparingly as a highlight/accent (e.g. active states, small pops of energy) —
enough to feel lively without competing with the grade-color system, which needs
green to mean something very specific (see below).

---

## 2. Grade Colors

The health grade (A–F) uses a flat, distinct color per band:

- **A** — green
- **B** — blue
- **C** — orange
- **D / F** — red

**Reason:** This needs to be readable at a glance, in a store, often on a small
phone screen, possibly in bad lighting — a shopper does not have time to read a
number carefully. A traffic-light-style progression (green → red) is the most
universally understood signal for "good → bad" and requires zero learning curve.
Grade colors are deliberately kept flat/solid rather than gradient-filled (see
Animations, below) so they stay legible and don't visually compete with the brand
palette.

---

## 3. Navigation

**Bottom navigation bar** (Home, History, Scan, Favorites, Profile) as the primary
navigation, with secondary destinations (Categories, Challenges, Leaderboard,
Shopping List, Settings, Weekly/Monthly) tucked into a header/mobile menu.

**Reason:** Thumb-friendly. Swapify is a shopping-aisle app — used one-handed, often
while holding a product in the other hand or pushing a cart. Anything the user needs
constantly (scanning, checking history, favorites, their own profile) sits within
comfortable thumb reach at the bottom of the screen. Less-frequent destinations don't
need that prime real estate, so they live in a secondary menu instead of crowding the
main nav bar.

---

## 4. Product Page — Score-First Layout

**Reason the health score comes first, before anything else on the Product page:**
The score is the single reason the user opened the app. Everything else — the
breakdown, the badges, the alternatives, the reviews — exists to support or explain
that one number. Leading with it means a user glancing at their phone for two
seconds in a store still gets the answer they came for, even if they never scroll
further.

---

## 5. Preferences — Rounded Chips

**Reason:** Rounded, pill-shaped toggle chips read as modern and low-friction
compared to checkboxes or a settings-style list. Dietary preferences are something a
user sets once and rarely revisits, so the interaction should feel light and quick
— tap to toggle, immediate visual feedback (fill/active state) — rather than feeling
like filling out a form.

---

## 6. Confidence Badge

**Reason:** Not every product in the catalogue has equally complete data — some
entries are hand-verified, others come from less certain sources. Rather than
presenting every score with false uniform authority, the Confidence Badge tells the
user how much to trust the number in front of them. This builds long-term trust in
the app: users learn that Swapify is upfront about the limits of its own data,
rather than discovering inconsistencies on their own and losing confidence in every
score.

---

## 7. Data Source Badge

**Reason:** Transparency. When a product isn't in Swapify's own curated catalogue,
the app falls back to Open Food Facts rather than returning nothing. The Data Source
Badge makes that fallback visible instead of silently blending two data sources
together — the user should always be able to tell whether they're looking at
Swapify's own curated data or a third-party lookup.

---

## 8. Better Alternatives — Placement

**Reason:** The primary product is always shown first, followed immediately by
healthier alternatives in the same category — never alternatives alone, and never
buried below reviews or other secondary content. The logic: if a product scores
poorly, the very next thing the user sees should be "here's a better option," not a
dead end. This turns a bad-news moment (low score) directly into a next action,
which is the whole value proposition of the app.

---

## 9. Charts (Monthly Report)

**Reason:** A month of scan history as a table of numbers is hard to act on. A
simple visual — grade distribution, a trend line/bar over time — lets a user see
"I'm buying healthier than last month" or "I've slipped" in one glance, which is far
more motivating than scrolling a list. Charts here are intentionally simple
(lightweight custom bar rendering rather than a full charting library) since the
data itself is simple; the goal is clarity, not visual complexity.

---

## 10. Animations — Subtle Only

**Reason:** Swapify's job is to communicate trustworthy nutrition information
quickly. Animation is used only to (a) provide feedback that an action registered
(button press, toast, favorite toggled) and (b) draw the eye to the one important
number on a screen (the score reveal). Anything beyond that — heavy transitions,
decorative motion — would work against a "professional, trustworthy health tool"
feel and risks feeling gimmicky on a screen people are using to make real
purchasing decisions.

---

## 11. Responsive Design Priority

**Reason mobile is the primary target, not an afterthought:** Realistically, almost
every scan happens standing in a store aisle, on a phone. Desktop/tablet support
exists for browsing, comparing, and reviewing history at home, but the interaction
model (camera scanning, one-handed use, quick glance-and-decide) is fundamentally a
mobile use case, so mobile layout and thumb-reach navigation were designed first,
with desktop/tablet treated as an expansion of the same layout rather than a
separate design.

---

## 12. Dark Mode

**Reason:** Shopping happens at all hours, and grocery stores are often lit with
harsh fluorescent lighting where a bright white screen is genuinely uncomfortable to
look at (and can wash out barcode-scanning visibility). Dark mode is offered as a
comfort option rather than purely an aesthetic one, with the entire palette
re-mapped through the same CSS custom properties used in light mode — so components
never need dark-mode-specific styling, they simply inherit the swapped tokens.

---

## 13. AI Chat — Floating Button

**Reason:** The AI Nutritionist needed to be reachable from literally anywhere in
the app — a user might want to ask "can I substitute this for something without
palm oil?" while looking at a product, or ask a general nutrition question from
Home. A floating action button that persists above whichever page is active (rather
than living inside one specific screen) makes that possible without adding a new
bottom-nav slot, which was already at capacity with the five most-used
destinations.