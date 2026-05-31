---
name: tax-season-organizer
description: >
  Prepares tax-season materials for small business owners — framed as
  deliverables for their accountant, not tax advice. EU mode (default): VAT
  return prep — pulls transaction data from Exact Online, categorises by VAT
  rate, separates intra-EU reverse-charge from domestic, and outputs a
  country-specific VAT summary for NL, DE, or BE. Non-EU / US mode: quarterly
  estimated tax calculation and year-end 1099-NEC prep.

  Trigger whenever the user mentions: quarterly VAT, BTW, MwSt, VAT return,
  estimated taxes, 1099s, 1099-NEC, year-end tax prep, contractor payments,
  W-9s, or any phrase suggesting they are preparing for a tax deadline or
  handing materials to an accountant.
---

# Tax Season Organizer

> **Framing:** This skill produces prep material for a tax adviser or accountant,
> not tax advice. State every assumption explicitly so the accountant can adjust.

## Quick start

Determine whether the owner needs EU VAT prep or non-EU (US) tax prep. Default
to EU VAT mode if business context shows an EU country. Then pull the relevant
data, calculate or compile, and deliver a structured document the accountant can
work from directly.

```
EU owner: "what do I owe for VAT this quarter?"
→ Pull transaction data from Exact Online
→ Categorise by VAT rate and transaction type
→ Separate intra-EU B2B (reverse charge) from domestic
→ Output VAT return summary for NL/DE/BE

US / non-EU owner: "what do I owe for estimated taxes?"
→ Pull YTD P&L from Exact Online or CSV
→ Calculate estimated income tax + SE tax
→ Show quarterly amount due with due date

User: "I need to send out 1099s"
→ Pull contractor payments from Exact Online + PayPal + Mollie
→ Flag contractors paid ≥ $600 USD equivalent
→ Output 1099-NEC candidate list
```

## Determine mode

Read business context (`## Business context`) for country field. If country is
in the EU, default to **EU VAT mode**. If country is US, default to **US tax
mode**. If unknown, ask:

> "Are you filing a VAT return (EU) or preparing estimated income taxes / 1099s
> (US)?"

For the full EU VAT workflow, use the dedicated `vat-return-prep` skill — it
handles country selection, VAT rate categorisation, OSS/IOSS detection, and
formatted output per member state. This skill delegates to it:

> Run `vat-return-prep` for EU owners. Return here only if the owner explicitly
> needs US 1099 or estimated tax prep.

---

## EU VAT mode → delegate to `vat-return-prep`

If the owner is in the EU and asks about VAT, taxes, or quarterly obligations,
invoke `vat-return-prep` immediately. Do not run the US tax paths below.

---

## US / Non-EU mode: Quarterly estimated tax

### 1. Pull YTD financials

Use Exact Online to pull a Profit & Loss report from January 1 of the current
year through the last day of the most recently completed quarter. If Exact Online
is not connected, ask the user to upload a P&L as CSV or paste the key numbers.

### 2. Ask about prior estimated payments

Before calculating, ask: "How much have you already paid in estimated taxes so
far this year?"

### 3. Calculate estimated liability

1. **SE tax** = net profit × 0.9235 × 0.153 (halve the deductible portion)
2. **Adjusted net** = net profit − (SE tax / 2)
3. **Federal income tax** = apply the bracket rate (default 22%; state explicitly)
4. **Total annual liability** = federal income tax + SE tax
5. **Quarterly payment** = (total annual liability − payments made) ÷ quarters remaining
6. **Safe harbor check** — note 100% of prior-year rule (110% if AGI > $150k)

State every assumption: bracket, business type, state taxes excluded.

---

## US / Non-EU mode: Year-end 1099 prep

### 1. Pull contractor payments

Query Exact Online, PayPal, and Mollie for all payments made to individuals or
businesses for services in the tax year. Do not include payments for goods,
refunds, or internal transfers.

### 2. Aggregate by payee

Combine across sources and sum by individual or business entity. Flag likely
duplicates for human review — never auto-merge.

### 3. Apply the $600 threshold

- **Flag for 1099-NEC:** payee paid ≥ $600 for services
- **Near-threshold alert:** flag payees paid $400–$599

### 4. Check W-9 status

For each flagged payee, note whether a W-9 / EIN is on file. Mark as:
- ✅ W-9 on file
- ⚠️ Missing — must collect before filing
- ❓ Unknown

### 5. Deliver the 1099 prep package

Structure: header, summary counts, 1099-NEC candidates table, missing W-9
action list, near-threshold table, payment processor note (PayPal/Mollie may
issue their own 1099-K), next-steps checklist.

---

## Guardrails

- **Not tax advice.** Open every deliverable with this: "Prepared for review by
  your accountant — not tax advice."
- **State every assumption.** Bracket, business structure, exclusions.
- **Don't merge payees automatically.** Flag duplicates for human review.
- **Don't file anything.** Output is prep material.

## Reference files

- [reference/calculation-assumptions.md](reference/calculation-assumptions.md)
- [reference/connector-queries.md](reference/connector-queries.md)
- [reference/gotchas.md](reference/gotchas.md)
