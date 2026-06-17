# Spendlens Expense Dashboard

A lightweight, single-page expense dashboard that converts multi-currency transactions to USD and surfaces spending insights for the Spendlens finance team.

**Live URL:** _(add your Netlify URL here after deploying)_

---

## What It Does

Loads 20 sample expense records across 10 currencies, converts them all to USD using a static rate snapshot (2026-05-01), and displays a summary dashboard with filtering, sorting, and an add-expense form. No backend, no database — everything runs in the browser.

---

## How to Run Locally

```bash
# No build step needed. Just open the file:
open index.html

# Or serve with any static server:
npx serve .
# then visit http://localhost:3000
```

---

## File & Folder Structure

```
spendlens/
├── index.html          # Entire app — data, logic, styles, markup in one file
├── README.md           # This file (technical handoff)
└── docs/
    ├── ceo-brief.md    # Plain-English briefing for non-technical stakeholders
    └── edge-cases.md   # Known failure modes and how they are handled
```

---

## Key Assumptions

- Exchange rates are static (snapshot: 2026-05-01). They do not update automatically.
- All data is in-memory. Refreshing the page resets any expenses added via the form.
- The EUR/USD rate slider is for scenario modelling only — it does not persist.
- No authentication or multi-user support is implemented.
- The 20 seed expenses are hardcoded in `index.html` inside the `INITIAL_EXPENSES` array.

---

## Known Limitations & What I'd Fix With 4 More Hours

| Limitation | Fix |
|---|---|
| Data resets on page refresh | Add `localStorage` persistence for added expenses |
| Static exchange rates | Fetch from an open rates API on load, fall back to static on failure |
| No edit or delete on expenses | Add row-level actions |
| No CSV export | Add a "Download as CSV" button for the finance team |
| EUR slider resets on refresh | Persist slider value in `localStorage` or URL params |
| No unit tests | Write tests for `toUSD()` and category aggregation logic |

---

## How the Currency Conversion Works

```js
// rates.js (inlined in index.html)
// Each rate = how many units of currency equal 1 USD
const BASE_RATES = { USD: 1.0, EUR: 0.9201, INR: 83.47, ... };

function toUSD(amount, currency) {
  const rate = BASE_RATES[currency];
  if (!rate || rate <= 0) return null; // guard: never divide by zero or null
  return amount / rate;
}
```

Adding a 25th currency: add one key-value pair to `BASE_RATES`. No other changes needed.

---

## Deployment (Netlify — recommended)

1. Push this folder to a GitHub repo (public)
2. Go to [netlify.com](https://netlify.com) → "Add new site" → "Import from GitHub"
3. Select the repo, leave build settings blank
4. Click **Deploy** — live in ~30 seconds
