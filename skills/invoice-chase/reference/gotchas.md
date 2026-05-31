# Gotchas

Known failure modes for invoice-chase.

---

**Customer paid via SEPA bank transfer — not visible in Mollie or PayPal.**

The payment processor cross-reference only catches processor payments. A customer who paid by SEPA credit transfer may still appear as overdue in AR. Note this in the summary: "Payment processor history only — SEPA transfers not verified." Let the owner confirm before sending.

---

**Exact Online AR includes internal or test accounts.**

Some setups include internal billing accounts or test records in AR. Before drafting, filter out customers whose email domain matches the owner's domain, and flag any customer name containing "Test," "Internal," or "Demo."

---

**Multiple overdue invoices from the same customer — send one email only.**

Never draft two separate reminders to the same customer in one batch. Consolidate all overdue invoices into one email with a total amount and a list of invoice numbers. Two emails to the same person in one batch looks disorganized and may trigger a spam filter.

---

**Mollie or PayPal reminder send fails for customers without an account.**

Mollie payment links work for any payment method (iDEAL, Bancontact, credit card, SEPA); no Mollie account required for the payer. If Mollie returns a send error, fall back to queuing a mail draft and report the fallback: "Mollie send failed for [customer] — queued as draft instead." Do not silently drop the reminder.

---

**Mollie and Exact Online may both carry the same invoice.**

If a customer appears in both Exact Online AR and Mollie overdue, it may be the same invoice in two systems. Match on invoice number first; if no number match, match on amount + due date. When uncertain, flag to the owner and send only one reminder rather than two.

---

**Mollie API returns rate limit errors.**

Mollie's API rate-limits when the requested date window is wide or multiple calls are fired in rapid succession.

*Fix:* Always query settlements with a **7-day window** ending today. This is the default in the workflow.

*Retry pattern:* If a 7-day query returns 429, retry immediately with a **3-day window**. A narrower window reduces the response payload and usually succeeds.

*Fallback:* If the 3-day retry also returns 429, skip the Mollie cross-reference for this run entirely. Flag every customer in the batch as "Mollie unavailable — verify manually" in the summary table. Proceed with Exact Online-only scoring. Do not silently drop the caveat — the owner needs to know the cross-reference was skipped before approving any sends.
