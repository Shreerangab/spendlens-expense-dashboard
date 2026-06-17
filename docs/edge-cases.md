# Edge Case Analysis — Spendlens Expense Dashboard

_Prepared as part of the contractor handoff. Documents known failure modes, current handling, and correct behaviour._

---

## Edge Case 1 — Currency amount of zero

**What could go wrong:** A user submits an expense with amount = 0. The conversion produces $0.00 USD, which silently inflates the transaction count without affecting totals. This corrupts the "average spend per transaction" metric and the category count.

**Current handling:** The add-expense form validates `amount > 0` and rejects zero with the message _"Amount must be a positive number."_

**Correct behaviour:** Reject at input. Zero-value expenses are not meaningful financial records.

---

## Edge Case 2 — Negative amount

**What could go wrong:** A user types -500 into the amount field. This would subtract from category totals, making the total spend appear lower than it is. A refund might legitimately be negative, but without a clear refund model this is ambiguous.

**Current handling:** The form validates `amount > 0` using HTML `min="0.01"` and a JS check. Negative values are rejected.

**Correct behaviour:** Reject negative amounts at the form level. If refunds are a future requirement, introduce a separate "Refund" transaction type with explicit handling in the aggregation logic.

---

## Edge Case 3 — Exchange rate is null or undefined

**What could go wrong:** If `BASE_RATES['XYZ']` is undefined (e.g. a typo in the currency code, or a currency that was removed from the rate file), then `amount / undefined` produces `NaN`. NaN propagates silently through all arithmetic — category totals, overall total, and merchant rankings all become `NaN` without any visible error.

**Current handling:** `toUSD()` checks `if (!rate || rate <= 0) return null`. The UI then renders `—` rather than a number. The form prevents submission with the message _"Exchange rate for [currency] is unavailable."_

**Correct behaviour:** Fail visibly and loudly. Never let a bad rate silently corrupt aggregated totals.

---

## Edge Case 4 — Exchange rate of exactly zero

**What could go wrong:** `amount / 0` in JavaScript produces `Infinity`, not an error. `Infinity` then propagates through totals, and the entire dashboard shows `Infinity` for all USD figures.

**Current handling:** The `rate <= 0` guard in `toUSD()` catches this and returns `null` before the division happens.

**Correct behaviour:** Treat a zero rate as a data error. Log a warning and display the affected row as unavailable.

---

## Edge Case 5 — Add expense form submitted with empty fields

**What could go wrong:** If merchant name is blank and the user clicks "Add Expense", an empty string gets pushed into the expense list. It renders as an empty table row and is included in top-merchant rankings with whatever amount was entered.

**Current handling:** The form checks `if (!merchant)` and shows _"Merchant name is required."_ Date is also validated. Amount is checked for a positive number.

**Correct behaviour:** All five fields (merchant, amount, currency, category, date) must be non-empty before the record is created. Each field should show a specific, actionable error message.

---

## Edge Case 6 — Merchant name with special characters (XSS risk)

**What could go wrong:** A merchant name like `<script>alert('xss')</script>` entered into the form could be injected directly into the DOM via `innerHTML`, executing arbitrary JavaScript in the user's browser.

**Current handling:** All merchant name strings are passed through `escHtml()` before being inserted into `innerHTML`. This replaces `<`, `>`, `&`, and `"` with their HTML entity equivalents.

**Correct behaviour:** Always sanitise user-supplied strings before DOM insertion. Alternatively, use `textContent` instead of `innerHTML` for plain-text fields.

---

## Edge Case 7 — Very large amount causing display overflow

**What could go wrong:** An amount like 999999999999 would render as `$999999999999.00` in the table, breaking the column layout on narrow screens and making the dashboard unreadable.

**Current handling:** The form rejects amounts above 10,000,000 with a warning message. This is a pragmatic cap for a personal/SMB expense tool.

**Correct behaviour:** Cap input at a reasonable maximum. Additionally, the USD formatter could use compact notation (e.g. `$1.2M`) for large values to protect layout integrity.

---

## Edge Case 8 — Category filter returns no results

**What could go wrong:** If a user filters by a category that has no matching expenses (possible after the dataset changes), the table renders nothing and the user sees a blank space with no feedback about why.

**Current handling:** When `filtered.length === 0`, the table renders a single row with the message _"No expenses in this category."_

**Correct behaviour:** Always show an empty state with a clear, action-oriented message. Optionally offer a one-click way to clear the filter.

---

## Edge Case 9 — App on a narrow mobile screen

**What could go wrong:** Wide tables with 7 columns do not reflow on mobile. Columns get cut off, horizontal scrolling appears inconsistently across browsers, and the KPI cards stack awkwardly.

**Current handling:** The table wrappers have `overflow-x: auto`, allowing horizontal scrolling. KPI cards use CSS Grid with `auto-fit` and `minmax(200px, 1fr)` to reflow. The layout is tested down to 380px.

**Correct behaviour:** On mobile, consider collapsing the expense table to show only Date, Merchant, and USD columns by default, with a toggle to expand. The summary cards and filter buttons should be the primary mobile experience.

---

## Edge Case 10 — EUR slider resets on page refresh

**What could go wrong:** A user adjusts the EUR rate to model a scenario, shares the screen with a colleague, and then refreshes. The slider snaps back to 0.9201 and all recalculated figures revert — mid-conversation.

**Current handling:** Not handled. The slider value is in-memory only.

**Correct behaviour:** Persist the slider value in `localStorage` or encode it in the URL as a query parameter (e.g. `?eur=1.05`), so the state survives a refresh and can be shared via link.
