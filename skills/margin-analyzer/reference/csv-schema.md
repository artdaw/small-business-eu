# CSV Upload Schema

When the owner doesn't have Exact Online or a payment processor connected, they can upload exported CSV files. This document specifies what columns to expect and how to handle variations.

## Revenue CSV (transactions export)

Expected columns (order doesn't matter; headers are case-insensitive):

| Column | Required | Description |
|---|---|---|
| `date` | Yes | Transaction date (any standard format: YYYY-MM-DD, DD-MM-YYYY, etc.) |
| `item` or `product` or `service` or `description` | Yes | What was sold |
| `amount` or `revenue` or `total` | Yes | Transaction amount (EUR) |
| `quantity` or `qty` | No | Units sold — if missing, assume 1 per transaction |

**Exports that typically match this format:**
- Mollie: Dashboard → Transactions → Export (CSV)
- PayPal: Activity → Download (CSV)
- Exact Online: Reporting → Transaction list → Export
- Shopify: Orders → Export

If the export has more columns, ignore the extras. If a required column is missing, ask the owner which column maps to it.

## Cost CSV (expense or COGS export)

Expected columns:

| Column | Required | Description |
|---|---|---|
| `date` | No | Expense date (useful for trend analysis) |
| `item` or `product` or `service` or `category` | Yes | What the cost relates to |
| `amount` or `cost` or `expense` | Yes | Cost amount (EUR) |
| `type` | No | COGS vs. operating expense — if absent, ask the owner |

**Exports that typically match this format:**
- Exact Online: Reporting → Profit & Loss Detail → Export
- Any expense tracker CSV export

## Handling messy CSVs

Real-world exports are messy. Common issues:

- **Extra header rows:** Skip rows until you find one that looks like column names
- **Currency symbols:** Strip `€`, `,` from numeric fields before parsing
- **Negative refunds:** Include them — they reduce net revenue
- **Mixed currencies:** Flag it and ask which currency to use; default to EUR if unclear
- **"Gross" vs "Net" amounts:** Prefer net (after fees) for revenue; ask if unclear

## After loading

Confirm the data shape with the owner before proceeding:
- "I loaded X transactions from [date] to [date] across Y products. Does that look right?"
- If the date range or product count looks off, ask them to double-check the export filters.
