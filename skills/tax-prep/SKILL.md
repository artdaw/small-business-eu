---
name: tax-prep
description: Dispatches to the right tax workflow based on owner location. EU owners go to vat-return-prep; US/non-EU owners get quarterly estimated tax or 1099-NEC prep via tax-season-organizer.
allowed-tools: Read, WebFetch, Bash
---

Run the tax prep workflow. Detect EU vs. non-EU context and route accordingly.
Act immediately — the user typed /tax-prep, so skip the discovery phase.

Parse arguments:
- `--mode` — `vat` (EU VAT return), `quarterly` (US estimated tax), `1099` (year-end 1099-NEC), `both` (quarterly + 1099); default: auto-detect
- `--country` — ISO country code (NL, DE, BE, FR, ES, US…); default: read from business context
- `--year` (default: current year)
- `--quarter` (default: most recently completed quarter)

**Framing:** Open every deliverable with "Prepared for review by your accountant — not tax advice."

## Step 1 — Detect mode

1. Read `## Business context` from session memory for the `country` field.
2. If country is in the EU (NL, DE, BE, FR, ES, IT, AT, PL, SE, DK, FI, IE, PT, or any other EU member state):
   - Default mode: `vat`
   - Confirm: "I'll prepare your VAT return summary for [country]. Is that right?"
3. If country is US or non-EU:
   - Default mode based on date: Oct–Jan → `both`, otherwise → `quarterly`
   - Confirm: "Based on the time of year, I'll prepare [mode]. Want me to do something different?"
4. If country is unknown, ask: "Are you filing a VAT return (EU) or preparing estimated income taxes / 1099s (US)?"

## Step 2 — EU: run vat-return-prep

If mode is `vat`, invoke the `vat-return-prep` skill immediately. Pass `--country` and `--quarter` arguments. Return the result to the owner.

## Step 3 — US: quarterly estimated tax (if mode includes quarterly)

Delegate to `tax-season-organizer` quarterly path:
1. Pull YTD Profit & Loss from Exact Online (Jan 1 through last completed quarter).
2. If Exact Online is not connected, ask the user to paste net income or upload a CSV.
3. Ask: "How much have you already paid in estimated taxes this year?"
4. Calculate: SE tax, adjusted net income, federal income tax estimate (default 22% bracket), quarterly payment due.
5. State every assumption explicitly — bracket, business type, exclusions.
6. Deliver the formatted estimate with the due date for the current quarter.

## Step 4 — US: year-end 1099 prep (if mode includes 1099)

Delegate to `tax-season-organizer` 1099 path:
1. Pull contractor/vendor payments from all connected sources: Exact Online, PayPal, Mollie.
2. Aggregate by payee across sources. Flag likely duplicates for human review — never auto-merge.
3. Apply the $600 threshold. Flag near-threshold payees ($400–$599).
4. Check W-9 status in Exact Online for each flagged payee.
5. Deliver the 1099-NEC candidate list with missing W-9 action items.

## Approval gates

- **Not tax advice.** State this in every output header.
- **State every assumption.** Bracket, business type, excluded deductions.
- **Don't merge payees automatically.** Flag duplicates for human review.
- **Don't file anything.** Output is prep material only.

## Output

End with a next-steps checklist for the accountant: missing items to collect, assumptions to verify, deadlines to hit.
