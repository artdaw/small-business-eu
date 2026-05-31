---
name: month-end-prep
description: >
  Walks an SMB owner through month-end close: reconciles Exact Online against
  Mollie, PayPal, and SumUp settlements, flags uncategorized transactions,
  suspicious duplicates, and missing receipts, then runs a VAT reconciliation
  check, writes a plain-English P&L narrative, and exports a close packet
  (xlsx + one-page PDF). Use when the user says "close the month," "month-end,"
  "reconcile," "what's missing," "P&L," or asks why revenue or margin changed
  this month.
---

# Month End Prep

## Quick start

Connect Exact Online and at least one payment processor (Mollie, PayPal, or SumUp),
then say "let's close the month." Claude walks you through each step of the checklist,
pausing for your input at each gate before moving forward.

If a connector is missing, Claude falls back to asking for a CSV export — it won't
silently skip a step.

## Workflow

Work through these steps in order. Each step has a completion state; don't advance
until the current step is settled.

### Step 1 — Agree on the target month

Ask the user which month to close. Default to the prior calendar month if they don't
specify. Confirm before pulling any data.

### Step 2 — Pull Exact Online P&L and transaction register

Fetch:
- Profit & Loss report for the target month (revenue, COGS, gross margin, operating
  expenses, net income) — all figures in EUR
- Transaction register: every income and expense line item
- VAT summary: total VAT collected, total VAT paid, net VAT position for the month

Flag immediately:
- **Uncategorized transactions** — any line with missing or "general" ledger category
- **Transactions needing review** — Exact Online's own review flag

Present the count ("14 transactions need a category") and list them for the user to
classify before proceeding. Don't advance with open uncategorized items unless the
user explicitly says "skip for now."

See [reference/exact-reconcile.md](reference/exact-reconcile.md) for field
mappings and API notes.

### Step 3 — Pull payment processor settlements

Fetch settlement reports from Mollie, PayPal, and SumUp — whichever are connected —
for the same calendar month.

Match each settlement deposit against the Exact Online bank deposit line:
- **Match** — amount and date agree within 2 days → mark as reconciled
- **Difference < €0.50** — rounding/fee; note but don't flag
- **Difference ≥ €0.50** — flag with the delta amount
- **Settlement exists, no Exact deposit** — flag as "missing in Exact Online"
- **Exact deposit exists, no settlement** — flag as "deposit not in processor data"

See [reference/settlements.md](reference/settlements.md) for settlement
report field mappings (Mollie, PayPal, SumUp).

### Step 4 — Detect suspicious duplicates

Scan the transaction register for likely duplicate charges or deposits. Flag a
transaction as a suspicious duplicate when **all three** match:
- Same amount (within €0.01)
- Same vendor or customer name
- Posted within 5 calendar days of each other

Present flagged pairs to the user. They decide whether each is legitimate (e.g., a
recurring weekly subscription) or a real duplicate to void.

See [reference/gotchas.md](reference/gotchas.md) for common false-positive patterns
and how to distinguish them.

### Step 5 — VAT reconciliation check

Pull the VAT summary from Exact Online for the target month:
- VAT collected on sales (output tax)
- VAT paid on purchases (input tax)
- Net VAT position (payable or reclaimable)

Cross-check against the transaction register:
- Flag any high-value transaction (> €500) with 0% VAT where standard rate is expected
- Flag any intra-EU B2B sale missing the reverse-charge note
- Note whether a VAT return is due this month or next (based on the owner's filing
  frequency stored in business context — monthly or quarterly)

Present the VAT position summary and flag any anomalies. Do not calculate the VAT
return filing itself — use `vat-return-prep` for that.

### Step 6 — Receipts check (Desktop connector)

If the Desktop connector is available, scan the receipts folder (ask the user for the
path; default `~/Documents/Receipts`) for the target month.

For each expense transaction in Exact Online above €25 with no attached document:
- Check for a matching receipt file (match by amount ± €0.50 and date within 3 days)
- **Matched** → note as "receipt on file"
- **Not matched** → flag as "missing receipt"

List missing receipts. The user can supply the file or mark as "receipt not required"
(e.g., a recurring auto-pay with no receipt).

If Desktop connector is not available, ask the user to confirm which expenses they have
receipts for — don't silently skip this step.

### Step 7 — Owner sign-off gate

Present a summary before going further:

```
Uncategorized transactions:  X of X resolved
Settlement discrepancies:    X flagged, X resolved
Suspicious duplicates:       X flagged, X cleared
VAT anomalies:               X flagged
Missing receipts:            X outstanding
```

Ask: "Ready to write the P&L summary and export the close packet?"

**Do not proceed to Steps 8–9 without explicit confirmation.**

### Step 8 — Write the P&L narrative

Write a plain-English summary of the month — the kind an owner would share with their
accountant, not a CFO memo. Aim for 150–250 words.

Structure:
1. **Headline** — one sentence: "March came in at €X net, up/down Y% from February."
2. **Revenue** — what drove the number; name products, services, or customers if
   the data shows concentration.
3. **Gross margin** — whether it held, rose, or compressed, and the main reason why.
4. **Key expenses** — any line that moved more than 10% MoM or is outside the normal
   range; one sentence each.
5. **VAT position** — net VAT payable or reclaimable this month; flag if it's materially
   different from prior months.
6. **Bottom line** — net income vs. prior month; ask if they have a target to compare.
7. **Watch list** — 1–3 things to monitor next month.

Avoid jargon; define anything that isn't plain English.

### Step 9 — Export the close packet

Produce two files:

**`close-packet-[YYYY-MM].xlsx`** — four sheets:
- `P&L` — the Exact Online P&L data, formatted in EUR
- `Reconciliation` — matched and flagged transactions side by side (Mollie, PayPal, SumUp)
- `VAT` — VAT summary: output tax, input tax, net position, anomalies flagged
- `Action Items` — any outstanding flags (uncategorized, missing receipts, etc.)

**`close-packet-[YYYY-MM]-summary.pdf`** — one page:
- Month and business name at the top
- Key figures (revenue, gross margin %, net income, VAT position)
- The P&L narrative from Step 8
- Count of open action items, if any

Save both to the Desktop (or a path the user specifies). Confirm the file locations.

## Approval gates

- **Never run reconciliation on a month that has been filed.** Confirm the books are
  still open before pulling data.
- **Never modify an Exact Online transaction directly.** Surface flags; the owner
  makes changes in Exact Online.
- **Always pause at Step 7** before producing outputs. Unresolved flags must be
  acknowledged or explicitly skipped.

## Graceful degradation

| Missing connector | Fallback |
|---|---|
| Exact Online | Ask for an Exact Online export CSV (P&L + transaction detail) |
| Payment processor | Ask for a settlement CSV from the processor's portal |
| Desktop (receipts) | Ask the user to confirm receipt status for each flagged expense |

## Reference files

- [reference/exact-reconcile.md](reference/exact-reconcile.md) — Exact Online field
  mappings, API pagination, common data issues
- [reference/settlements.md](reference/settlements.md) — settlement
  report structure for Mollie, PayPal, and SumUp
- [reference/close-packet-format.md](reference/close-packet-format.md) — xlsx column
  specs, PDF layout, file naming convention
- [reference/gotchas.md](reference/gotchas.md) — duplicate false positives, split
  transactions, partial-month edge cases
- [reference/examples/pl-narrative.md](reference/examples/pl-narrative.md) — worked
  P&L narrative example
