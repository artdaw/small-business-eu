---
name: invoice-chase
version: 0.3.0
description: >
  Drafts overdue-invoice reminder emails from Exact Online and Mollie/PayPal
  data, matched to each customer's payment history and tone (gentle for good
  customers, firm for repeat late payers). Sends via Mollie or PayPal with
  owner approval; non-Mollie/PayPal invoices queue as mail drafts. SEPA
  reference numbers included in all reminders. Use when the user asks
  "who owes me money," mentions overdue invoices, or wants to follow up
  on unpaid invoices.
---

# Invoice Chase

## Quick start

Pull the AR aging report, score each customer by payment history, draft a tone-matched reminder for each overdue invoice, and present them to the owner. Nothing sends until the owner says so.

```
User: "who owes me money"
→ Pull AR aging from Exact Online
→ Cross-reference Mollie and PayPal settlements (last 7 days)
→ Score each customer: good-payer / occasionally-late / repeat-late
→ Draft tone-matched reminders with SEPA reference numbers
→ Show summary table + drafts. Wait for "send these."
```

## Setup (first run only)

Ask the owner two questions before running for the first time:

1. **Mail connector**: "Do you use Gmail or Outlook for drafts?" — store the answer; use it for all non-Mollie/PayPal draft queuing.
2. **Payment processors**: "Do you use Mollie, PayPal, or both for invoicing? I'll include those in the overdue sweep." — store the answer.

Do not ask again on subsequent runs.

## Workflow

1. **Pull overdue receivables.** Query Exact Online AR aging for all invoices more than 1 day past due. If Exact Online is not connected, ask the user to upload an AR aging export CSV.

2. **Cross-reference payment history.** For each overdue customer, query Mollie and/or PayPal for settled transactions:
   - **Mollie**: query payments by customer email, status `paid`, last 7 days
   - **PayPal**: `transaction_status: S` (settled only), last 7 days

   **If Mollie or PayPal returns a rate limit or error:**
   - Retry once with a 3-day window instead.
   - If still failing, skip the cross-reference for that processor. Flag affected customers as "payment processor unavailable — verify manually" in the summary table. Proceed with Exact Online history only.

   If a customer shows a settled payment within the query window, flag as "possibly paid — verify" and exclude from the draft queue.

3. **Score each customer.** Read [reference/tone-matching.md](reference/tone-matching.md) for scoring logic. Result: `good-payer`, `occasionally-late`, or `repeat-late`.

4. **Draft reminder emails.** One email per customer — consolidate multiple overdue invoices into one email. Match tone to score. Include:
   - Invoice number(s) and amount(s) in EUR
   - SEPA payment reference (use Exact Online invoice number as the payment reference)
   - IBAN/payment details from Exact Online if available
   See [reference/examples/gentle-reminder.md](reference/examples/gentle-reminder.md) and [reference/examples/firm-reminder.md](reference/examples/firm-reminder.md).

5. **Present drafts to owner.** Show a summary table first:

   | Customer | Amount Due | Days Late | Tone | Send via |
   |---|---|---|---|---|
   | Acme BV | €1,200 | 18 days | Gentle | Mollie |
   | Smith GmbH | €450 | 47 days | Firm | Gmail draft |

   Then show each draft email in full. Wait for owner to say "send these" or approve individually.

6. **Send or queue — only after approval.**
   - Mollie invoices: send the reminder via Mollie.
   - PayPal invoices: send via PayPal.
   - Other invoices: queue as a draft in the owner's configured mail app.
   - Never send without explicit approval.

7. **Report what happened.** List what was sent, what was queued as draft, and what was flagged (possibly paid, excluded).

## Approval gates

- **Never send or queue a draft without explicit owner approval.** Present all drafts first; wait for the go-ahead.
- **Never include a customer who paid in the last 7 days.** Flag as "possibly paid — verify" instead.
- **Never send to a customer not in the Exact Online AR report.** No reminders from memory alone.
- **One approval covers one batch.** Adding a customer or changing a draft after approval starts a new round.

## Reference

- [reference/tone-matching.md](reference/tone-matching.md) — scoring logic, tone guidelines, subject line formulas
- [reference/gotchas.md](reference/gotchas.md) — known failure modes
- [reference/examples/gentle-reminder.md](reference/examples/gentle-reminder.md) — good-payer email example
- [reference/examples/firm-reminder.md](reference/examples/firm-reminder.md) — repeat-late-payer email example
