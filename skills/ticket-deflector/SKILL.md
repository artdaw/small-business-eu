---
name: ticket-deflector
description: >
  Reads a forwarded customer email or ticket, pulls order/refund status from
  Mollie and PayPal and account history from HubSpot or Brevo, drafts a
  tone-matched reply in the owner's writing voice, and can issue a Mollie or
  PayPal refund with explicit owner approval. Use when the user says "draft a
  response," "answer this customer," "where's my order," or "I want a refund."
compatibility: "Requires Mollie or PayPal, HubSpot or Brevo, Mail."
---

# Ticket Deflector

## Quick start

Forward or paste a customer email — Claude pulls order status from Mollie (primary) or PayPal, looks up the customer in HubSpot or Brevo, and drafts a reply in the owner's voice. If a refund is needed, it stages the details and waits for explicit approval before issuing anything.

```
User: "answer this customer" [forwards email]
→ Extract customer email + issue from thread
→ Pull Mollie transaction status (fall back to PayPal)
→ Pull HubSpot or Brevo contact history
→ Draft reply in owner's voice
→ Owner approves draft → send or stage
→ If refund needed: approval prompt → owner confirms → issue via Mollie or PayPal
```

## Workflow

1. **Read the customer message.** Accept a forwarded Gmail or Outlook thread or pasted text. Extract: customer email address, name, order or transaction ID (if present), and the core issue — refund request, order status question, or general complaint. If multiple issues are present, address them in the order they appear.

2. **Pull order status from payment processors.** Check in order:
   - **Mollie**: search by customer email or payment ID. Capture: amount, date, status (paid/pending/failed/refunded), payment method (iDEAL, card, SEPA).
   - **PayPal**: search by customer email or transaction ID. Capture: amount, date, status. Use a 7-day window if searching by email — PayPal throttles wide date-range queries.
   - If neither is connected, note it in the draft and continue.
   - If no transaction matches either processor, flag it inline in the draft — do not guess at a match.
   - If multiple transactions match, surface all of them and ask the owner which one applies before drafting.

3. **Pull customer history from CRM.** Check in order:
   - **HubSpot**: search contacts by email. Pull: lifecycle stage, notes, open deals, recent activity.
   - **Brevo**: search contacts by email. Pull: contact attributes, recent email interactions, CRM deals.
   - If no contact exists in either, note it and offer to create one after the reply is sent — do not create during the response workflow.

4. **Draft the reply.** Write in the owner's writing voice. Adjust tone to fit the issue type:
   - Refund request → empathetic, clear, action-oriented
   - Order status question → factual, reassuring
   - General complaint → acknowledge, explain, offer resolution
   Flag any data gaps inline in the draft with a bracketed note so the owner sees the gap before sending.

5. **Approval gate — owner reviews the draft.** Present the full draft. Do not send or stage it until the owner approves. The owner may edit freely before approving.

6. **Approval gate — refund issuance.** If a refund is warranted, surface a dedicated confirmation prompt after the owner approves the draft:

   **Mollie refund:**
   > *"Issue refund of €[amount] to [customer name] ([email]) for Mollie payment [ID]? Reply Y to proceed."*

   **PayPal refund:**
   > *"Issue refund of €[amount] to [customer name] ([email]) for PayPal transaction [ID]? Reply Y to proceed."*

   Wait for explicit confirmation. If the owner's reply is anything other than a clear yes, stop and ask what they'd like to do instead.

7. **Send or stage the reply.** After draft approval, ask the owner: send via Gmail/Outlook now, or save as a draft? Execute their choice. Then log the interaction as a note on the HubSpot or Brevo contact.

8. **Report.** One short paragraph: reply sent or staged, refund issued or not, CRM note logged.

## Approval gates

- **Never issue a Mollie or PayPal refund without explicit owner confirmation** — always show amount, customer name, email, and transaction/payment ID before executing.
- **Never send the reply without owner review.** Always present the full draft first.
- **Never create a CRM contact during the response flow.** Offer it afterward.
- **Never auto-select a transaction.** If multiple match, surface them all and let the owner choose.
- **Never fabricate order details.** If neither processor has a record, say so inline in the draft — do not invent a status.

## Reference

- [reference/gotchas.md](reference/gotchas.md) — Good / Bad patterns for tone, payment lookup, and ambiguous refund scenarios
- [reference/examples/respond-refund-request.md](reference/examples/respond-refund-request.md) — worked example: refund request with Mollie payment found
