# Connector Query Guide

How to pull the right data from each connector for each mode.

> **EU note:** This skill operates in two modes. EU mode (vat-return-prep) delegates to `vat-return-prep/SKILL.md`. The queries below apply to US mode only (quarterly estimates and 1099-NEC prep).

---

## Exact Online — Quarterly mode (P&L)

Pull a **Profit & Loss** report for the period January 1 through the last day of the most recently completed quarter.

Key fields to capture:
- `Revenue` (gross revenue, EUR)
- `TotalExpenses` (all operating expenses)
- `NetIncome` (= revenue − expenses; this is the basis for tax calculation)

If Exact Online returns multiple income/expense categories, sum them. You want the single bottom-line net profit figure.

**If the user's Exact Online is on cash basis**, use that. If accrual, note it in output — the accountant should confirm which basis to use for estimated taxes.

---

## Exact Online — Year-end mode (contractor payments)

Pull all **purchase invoices and payments** to vendors for the full tax year (Jan 1 – Dec 31).

Filter for:
- Vendor category = freelance, consulting, contract labor, subcontractor, or similar
- Any vendor paid more than the local 1099-equivalent threshold

For each vendor record, capture:
- Vendor name (legal name if available)
- Tax ID / registration number (from vendor profile — indicates W-9 or EU equivalent on file)
- Total payments for the year (EUR)
- Payment dates and amounts (for cross-reference)

**Common issue:** Many Exact Online users do not tag vendors by payment-reporting eligibility. If filtering returns few results, pull all significant vendor payments and let the owner/accountant classify them. Note this in output.

---

## PayPal — Year-end mode

Pull all **"Goods & Services" payments sent** (not received) for the tax year.

Key fields:
- Recipient name / email
- Total amount per recipient (aggregate for the year, EUR)
- Transaction type = "Payment" or "Business Payment"
- Date

**Exclude:** personal payments ("Friends & Family"), refunds, disputes, transfers to own accounts.

**Reporting note:** PayPal may issue its own tax reporting document to recipients above local thresholds. Flag this in output — the accountant determines whether the business must also file separately or can rely on PayPal's document.

---

## Mollie — Year-end mode

Pull all **transfers to external accounts** for the tax year — payouts to contractors (not payouts to the business's own bank).

Key fields:
- Recipient name / description
- Total transferred per recipient for the year (EUR)
- Payment description / metadata (to confirm these are for services rendered)

**Exclude:** Mollie payouts to the business's own bank account (these are revenue settlements, not contractor payments).

---

## Desktop / CSV fallback

If any connector is unavailable, ask the user to:
1. Export a P&L from Exact Online as CSV (Reporting → Profit & Loss → Export)
2. Export a transaction history from PayPal (Activity → Download → CSV)
3. Export a payout report from Mollie (Dashboard → Payouts → Export)

When reading uploaded CSVs, look for these columns (names vary by export):
- P&L: `Description`, `Amount`, `Type` (Income / Expense)
- PayPal: `Name`, `Type`, `Amount`, `Date`, `Transaction ID`
- Mollie: `Description`, `Amount`, `Created at`, `Status`

If columns don't match, ask the user to identify the payee name and amount columns.
