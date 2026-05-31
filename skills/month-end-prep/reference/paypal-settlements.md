# Payment Processor Settlements Reference

## Contents

- [Mollie](#mollie)
- [PayPal](#paypal)
- [SumUp](#sumup)
- [Reconciliation logic (all processors)](#reconciliation-logic-all-processors)

---

## Mollie

### Settlement report structure

The Mollie MCP returns settlement objects. Key fields:

| Field | Notes |
|---|---|
| `id` | Settlement ID (e.g. stl_jDk30akdN) |
| `settledAt` | ISO 8601 timestamp when funds were transferred to bank |
| `amount.value` | Net settlement amount in EUR (fees already deducted) |
| `amount.currency` | Always EUR for EU accounts |
| `status` | Use only `paidout` for reconciliation |

**Payout timing:** Mollie batches payouts — typically T+1 or T+2 business days. A payment on April 28 may settle on April 30. Match by `settledAt`, not payment creation date.

**Fees:** Mollie deducts transaction fees before payout. The `amount.value` is net. When reconciling against Exact Online, compare to the net bank deposit.

**Refunds:** Appear as negative amounts in the settlement or as separate refund transactions. Cross-check with Exact Online credit notes — a refund that doesn't have a matching credit note is a flag.

**SEPA Direct Debit:** SEPA mandates have a 5-business-day collection window. A mandate authorised on April 1 may not settle until April 8. Account for this lag in the 30-day window.

---

## PayPal

### Settlement report structure

The PayPal connector returns a Transaction History or Settlement report.
Key fields:

| Field | Notes |
|---|---|
| `transaction_info.transaction_id` | PayPal transaction ID |
| `transaction_info.transaction_initiation_date` | When the transaction occurred |
| `transaction_info.transaction_amount.value` | Gross amount in EUR (positive = money in) |
| `transaction_info.fee_amount.value` | PayPal fee (negative) |
| `transaction_info.transaction_status` | Use only `S` (Success) and `P` (Pending) |
| `payer_info.email_address` | Customer email — useful for matching |

**Deposit date vs. transaction date:** PayPal batches payouts. A sale on April 28 may not deposit until May 2. Match by **deposit date** when reconciling against Exact Online bank entries.

**Refunds:** Appear as negative amounts with `transaction_type` = `T1107` or `T1114`. They should offset the original sale — flag a refund that doesn't have a corresponding Exact Online credit note.

**Holds:** Transactions in status `F` (Funds on Hold) have not been deposited yet. Note them separately — they'll appear in next month's settlement.

**Rate limiting:** PayPal throttles wide date-range queries. Use a 7-day window when searching by email. For reconciliation covering a full month, break the query into weekly segments and combine results.

---

## SumUp

### Settlement report structure

SumUp MCP returns payout objects. Each payout covers 1–2 business days of POS sales.

Key fields:

| Field | Notes |
|---|---|
| `id` | Payout ID |
| `date` | ISO 8601 date when funds arrived in the bank account |
| `amount` | Net payout amount in EUR (fees already deducted) |
| `currency` | EUR |
| `status` | Use only `PAID` for reconciliation |

**Fees:** SumUp deducts fees before the payout — `amount` is net. Compare to the net bank deposit in Exact Online.

**Instant Transfer:** SumUp offers next-business-day settlement by default; Instant Transfer settles same-day. If the owner uses Instant Transfer, transaction date and payout date will usually match.

---

## Reconciliation logic (all processors)

Use this logic to match processor settlements against Exact Online bank deposits:

```
for each processor settlement in target month:
    find Exact Online deposit where:
        abs(Exact.amount - processor.net_amount) < €0.50
        AND abs(Exact.date - processor.settlement_date) <= 2 days

    if match found:
        mark as RECONCILED
    elif abs(Exact.amount - processor.net_amount) < €0.50 (date mismatch only):
        flag as DATE_MISMATCH (usually a timing difference — low priority)
    elif processor settlement not matched at all:
        flag as MISSING_IN_EXACT
    elif Exact deposit not matched to any processor record:
        flag as UNMATCHED_DEPOSIT (may be SEPA bank transfer, direct debit, or other income)
```

**Multi-processor businesses:** Run this logic independently per processor, then aggregate flags. An Exact deposit that doesn't match Mollie may legitimately be a PayPal or SumUp payout — don't flag it until you've checked all connected processors.

**SEPA transfers:** Many EU B2B payments arrive as SEPA credit transfers directly to the bank account — these appear in Exact Online as `UNMATCHED_DEPOSIT` because there's no processor record. They are not errors; they're direct bank payments. Flag them separately as "SEPA direct — no processor record, verify against AR."
