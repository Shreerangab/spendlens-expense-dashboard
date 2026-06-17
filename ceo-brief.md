# CEO Briefing Note — Spendlens Expense Dashboard

**To:** CEO  
**From:** Product Analyst  
**Date:** 2026-05-01  
**Re:** Sprint 1 — What we built, what we traded off, what's next

---

## What We Built

A live, browser-based expense dashboard that pulls 20 transactions spanning 10 currencies and converts everything to USD in one place. The finance head no longer needs to Google exchange rates or maintain a manual spreadsheet. The dashboard shows total spend, a category breakdown, the top spending merchants, and a full sortable transaction table — all updated instantly when new expenses are added.

There is also a "What-If" slider that lets anyone adjust the EUR/USD rate and immediately see how it changes the total. This is useful ahead of board meetings when the team wants to model currency sensitivity.

---

## The Three Most Important Trade-offs

**1. Static rates over live API rates**  
We used a fixed rate snapshot rather than calling a live currency API. This means rates are locked to 2026-05-01. The upside is that every run of the report produces identical numbers — critical for audit trails and board packs. The downside is that the rates go stale. A live API would require an API key, error handling for outages, and a decision about when to refresh rates during a reporting period.

**2. No data persistence**  
Expenses added through the form disappear when you refresh the page. We prioritised getting a working, deployed product over building a backend. A proper solution would store data in a database or at minimum in the browser's local storage.

**3. Single-file architecture over a multi-file codebase**  
The entire app — data, logic, styles, and markup — lives in one HTML file. This made deployment trivially simple (drag and drop) and removed any build-step complexity. The trade-off is that the file will become hard to maintain as it grows past a few hundred lines.

---

## Three Things to Prioritise in the Next Sprint

**1. Persistent storage (highest impact)**  
Right now, every added expense is lost on refresh. Connecting even a simple backend (Supabase free tier, for example) would make the tool genuinely usable by the finance team day-to-day. Expected impact: transforms this from a demo into a real operational tool.

**2. Live exchange rates with a fallback**  
Integrating an open exchange-rate API (fetched once on load, with the static snapshot as fallback) would keep numbers current without sacrificing auditability. Expected impact: eliminates the manual rate-update step the finance team currently does monthly.

**3. CSV export**  
The finance head's existing workflow ends in a spreadsheet. A one-click "Export to CSV" button would close that loop and make adoption frictionless. Expected impact: immediate time saving for the monthly board report cycle.

---

*What is not finished: the app has no user authentication, no multi-user support, and no edit or delete functionality on existing expenses. These were out of scope for a 48-hour sprint and are documented in the README.*
