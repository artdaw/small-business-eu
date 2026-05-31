---
name: vat-return-prep
version: 1.0.0
description: >
  Prepares EU VAT return materials for small business owners in the Netherlands
  (BTW), Germany (MwSt / Umsatzsteuer), and Belgium (BTW/TVA) — v1 scope.
  Pulls transaction data from Exact Online, categorises by VAT rate (0%,
  reduced, standard), separates intra-EU B2B reverse-charge transactions from
  domestic sales, flags OSS/IOSS eligibility for cross-border e-commerce, and
  outputs a structured VAT summary the owner or accountant can file from.
  Not tax advice — prep material only.

  Trigger when the user says: "VAT return," "BTW aangifte," "Umsatzsteuer,"
  "MwSt-Voranmeldung," "quarterly VAT," "what do I owe VAT," "VAT filing,"
  "OSS," "IOSS," or any phrase suggesting they are preparing a VAT return.
---

# VAT Return Prep

> **Framing:** This skill produces prep material for your accountant or tax
> adviser, not tax advice. State every assumption explicitly.

## Quick start

```
User: "prepare my Q1 VAT return"
→ Ask country if not in business context (NL / DE / BE)
→ Pull transaction data from Exact Online for the quarter
→ Categorise: standard rate, reduced rate, zero rate, exempt, intra-EU B2B
→ Compute net VAT position (output tax − input tax)
→ Flag reverse-charge, OSS/IOSS, and any anomalies
→ Deliver structured VAT summary + accountant handoff packet
```

## Step 1 — Confirm country and period

1. Read `## Business context` for country. If not present, ask: "Which country is your business registered in — Netherlands, Germany, or Belgium?"
2. Confirm the filing period. Ask: "Which quarter are you preparing — Q1, Q2, Q3, or Q4 [year]?" Default to the most recently completed quarter.
3. Note the filing deadline:
   - **Netherlands (BTW):** 1 month and 1 day after quarter end (e.g. Q1 → April 30)
   - **Germany (MwSt):** 1 month after quarter end for quarterly filers (e.g. Q1 → April 30); 10th of following month for monthly filers
   - **Belgium (BTW/TVA):** 20th of month following quarter end (e.g. Q1 → April 20)

## Step 2 — Pull transaction data from Exact Online

Fetch all transactions for the filing period:
- Sales invoices: customer, amount excl. VAT, VAT amount, VAT code/rate, country of customer
- Purchase invoices: supplier, amount excl. VAT, VAT amount recoverable, country of supplier
- VAT codes to map to rate categories (see Step 3)

If Exact Online is not connected, ask the user to upload a transaction export CSV with columns: date, type (sale/purchase), amount excl. VAT, VAT amount, VAT rate %, customer/supplier country, description.

## Step 3 — Categorise transactions

**Sales (output tax):**

| Category | Description | VAT rate |
|---|---|---|
| Domestic standard | Sales to domestic customers, standard goods/services | NL: 21% / DE: 19% / BE: 21% |
| Domestic reduced | Sales to domestic customers, reduced-rate goods/services | NL: 9% / DE: 7% / BE: 6% or 12% |
| Domestic zero-rated | Exports outside EU, certain food/medicine | 0% |
| Intra-EU B2B (ICP) | Sales to VAT-registered businesses in other EU member states | 0% + reverse charge; list separately |
| Intra-EU B2C | Sales to consumers in other EU member states | Local rate of destination country — OSS eligible |
| Exempt | Insurance, healthcare, education, financial services | No VAT |

**Purchases (input tax / aftrekbare voorbelasting / Vorsteuer):**
- Map each purchase invoice to the above categories
- Flag any purchase with no VAT code — may indicate non-recoverable input tax
- Flag purchases from non-EU suppliers where import VAT may apply

**Reverse charge flag:** For each intra-EU B2B sale, verify the customer's VAT number is recorded in Exact Online. If missing, flag as "VAT number required — cannot apply reverse charge without it."

## Step 4 — Check OSS/IOSS eligibility

If the owner has intra-EU B2C sales (selling to consumers in other EU member states):
- If total intra-EU B2C sales exceed €10,000/year across all EU member states: the owner should be registered for OSS (One Stop Shop) and charging the destination country's VAT rate.
- Flag: "Your intra-EU B2C sales appear to exceed the €10,000 OSS threshold. Are you enrolled in OSS? If not, consult your accountant — you may need to register."
- If already enrolled in OSS: note that these sales should be reported in the OSS return, not the domestic VAT return.

## Step 5 — Compute net VAT position

| Line | Amount |
|---|---|
| Output tax (VAT collected on sales) | €X |
| Input tax (VAT recoverable on purchases) | −€X |
| **Net VAT payable / (reclaimable)** | **€X** |
| Intra-EU ICP sales (informational) | €X |

If the net position is negative (reclaimable), note: "You have a VAT credit this quarter. Depending on your filing method, this may be automatically refunded or carried forward."

## Step 6 — Flag anomalies

Scan for and flag:
- Invoices with missing or inconsistent VAT codes
- High-value sales (> €2,000) with 0% VAT where standard rate might apply — flag for accountant review
- Intra-EU B2B sales missing customer VAT numbers
- Any transaction categorised as reduced-rate that may not qualify (applies per-country rules)
- Large input tax amounts on purchases that may not be fully recoverable (e.g. mixed-use assets)

## Step 7 — Deliver the VAT prep package

Structure the output as:

**Header:**
> VAT Return Prep — [Country] [Quarter] [Year]
> Prepared: [date] — For review by your accountant — Not tax advice

**1. Period summary**
- Filing period, filing deadline, country
- Filing frequency (monthly / quarterly)

**2. Output tax (sales VAT)**
Table: category, gross sales, VAT rate, VAT amount

**3. Input tax (recoverable VAT on purchases)**
Table: category, net purchases, VAT rate, VAT recoverable

**4. Net VAT position**
Output tax − Input tax = Net payable / (reclaimable)

**5. Intra-EU ICP list (if applicable)**
Customer VAT numbers, country, total sales — needed for the ICP / Zusammenfassende Meldung / Opgave intracommunautaire handelingen

**6. OSS/IOSS note (if applicable)**
Whether OSS threshold is exceeded; whether sales are reported here or in OSS return

**7. Anomalies flagged**
Numbered list with each flag and action required

**8. Next steps**
- Confirm VAT numbers for intra-EU customers
- Resolve flagged anomalies with accountant
- File by [deadline]
- Pay by [deadline] (same as filing deadline in most cases)

## Guardrails

- **Not tax advice.** Open every deliverable with this statement.
- **State every assumption.** VAT rates assumed, reduced-rate goods/services included or excluded, OSS status assumed.
- **Don't file anything.** Output is prep material.
- **v1 scope: NL, DE, BE only.** If owner is in FR, ES, IT, or another EU country, note: "VAT return prep for [country] is coming in v2. For now, I can give you a general transaction categorisation — your accountant will need to map it to the local form."

## Reference files

- [reference/vat-rates.md](reference/vat-rates.md) — VAT rate tables for NL, DE, BE; reduced-rate goods lists
- [reference/icp-rules.md](reference/icp-rules.md) — intra-EU B2B rules, reverse charge, VAT number validation
- [reference/oss-ioss.md](reference/oss-ioss.md) — OSS/IOSS threshold, registration, reporting obligations
- [reference/gotchas.md](reference/gotchas.md) — common errors: wrong VAT codes, missing VAT numbers, mixed OSS/domestic
