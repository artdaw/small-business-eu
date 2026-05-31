---
name: month-heads-up
description: Runs a 30-day forward cash outlook and flags anything that needs attention before month-end. Accepts optional 30 or 60 day horizon.
allowed-tools: Read, WebFetch, Bash
---

Run the month-end heads-up. Pull forward-looking cash data and give the owner a clear "here's what the next 30 days look like" picture with specific things to watch.

Parse arguments:
- `--horizon` (default: `30`) — forecast window in days (`30` or `60`)

## Step 1 — Current cash position

Using the `cash-flow-snapshot` skill workflow:

1. Pull Exact Online current cash and receivables balance (EUR).
2. Pull Mollie settled balance and pending payouts.
3. Pull PayPal settled balance (if connected).
4. Combine for total available + incoming cash.

## Step 2 — Upcoming obligations

1. Pull recurring expenses from Exact Online (payroll, subscriptions, rent/lease) due in the next 30 days.
2. Pull any outstanding invoices past due or due within 14 days.
3. Flag any VAT payment due in the horizon window (from business context or last VAT run).
4. Flag any payment that would push the balance below a comfortable buffer (default: <€2,000 or owner's Exact Online average monthly expense × 0.5).

## Step 3 — Cash-flow forecast

1. Project 30-day net cash: current balance + expected inflows − known obligations (including VAT if due).
2. Identify the single tightest week (lowest projected balance).
3. Flag if any week projects negative.

## Step 4 — Two things to watch

Surface no more than two specific, actionable watches:
- Which invoice(s) to chase now
- Which expense(s) to defer or negotiate

Format as:

```
Month-End Heads Up — {current date}
Horizon: next {X} days

Cash today: €{amount}
Projected end-of-period: €{amount}
Tightest week: {date range} — projected €{amount}

VAT due: {date} — estimated €{amount}
(omit if no VAT due in horizon)

TWO THINGS TO WATCH
1. {item} — {why it matters} — suggested action: {action}
2. {item} — {why it matters} — suggested action: {action}
```

## Connector failures

If Exact Online is unreachable, stop — the cash forecast requires Exact Online as the source of truth. If Mollie or PayPal is missing, run the forecast from Exact Online-only data and note what's excluded.

## Approval gates

- **Never initiate payments or send emails automatically.** Surface the data and actions for the owner to take.
- **Never project revenue that hasn't been confirmed in Exact Online or a payment processor.** Use conservative estimates only.

## Output

Present the formatted brief (all amounts in EUR) and offer to draft chase emails for any flagged overdue invoices.
