# Gotchas — cash-flow-snapshot

Known edge cases and connector failure modes. 2–5 entries, Good/Bad format.

---

## 1. Exact Online AR aging includes invoices already collected

**Bad:** Including fully-paid invoices from the AR aging report inflates inflow
projections. Exact Online sometimes shows zero-balance invoices in aging exports.

**Good:** Filter AR rows to `AmountOutstanding > 0` before computing inflows. If the
connector doesn't expose outstanding balance directly, subtract known Mollie/PayPal/SumUp
settlements from the invoice total before including it.

---

## 2. Mollie and PayPal settlement lag varies by transaction type

**Bad:** Assuming all Mollie receipts settle in 1–2 business days. Both Mollie and
PayPal hold funds differently for disputes, new accounts, and high-value transactions —
using a flat settlement assumption produces overconfident inflow timing.

**Good:** Compute settlement lag from actual `transaction_date` → `completed_date`
pairs in the settlement history. Use the computed mean and stddev per
customer or transaction type.

---

## 3. CSV column names are inconsistent across accounting exports

**Bad:** Requiring exact column names like "Date", "Amount", "Type". Exact Online
CSV exports use "Transaction Date", "AmountDC", "Type". Other exports vary.
Rigid parsing fails silently.

**Good:** Fuzzy-match column headers (date → transaction date → txn date;
amount → AmountDC → debit/credit; type → category → account type). Show the header row
to the user and confirm mapping before computing — one question beats a silent
wrong forecast.

---

## 4. Fixed costs hidden in one-off AP entries

**Bad:** Only pulling recurring line items labeled as "recurring" in Exact Online.
Many SMBs don't tag fixed costs consistently — rent may appear as a one-off
vendor bill each month.

**Good:** Look for AP entries that appear in 3+ consecutive months with the same
vendor and similar amount (±10%). Treat these as recurring fixed costs in the
forecast. Surface the list to the user: "I'm treating these as fixed monthly
costs — does that look right?"

---

## 5. Confidence band formula breaks when mean payment lag is zero

**Bad:** Dividing stddev by a mean lag of 0 (e.g. immediate Mollie iDEAL payments)
produces a divide-by-zero error or an infinite band.

**Good:** If mean lag ≤ 1 day, set band_pct to 5% (low variance, near-immediate
settlement). Don't attempt the division.
