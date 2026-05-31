# Exact Online — Reconciliation Reference

## Reports to pull

### Profit & Loss (P&L)

Use the **Profit and Loss** report via the Exact Online MCP (CData connector):
- Date range: first day to last day of target month
- Accounting method: **Accrual** unless the user has confirmed they run cash-basis
- Include all GL accounts
- Currency: EUR (Exact Online is EUR-native for NL/BE divisions; DE/FR divisions also EUR)

Key fields used downstream:
| Field | Notes |
|---|---|
| `Revenue` | Top-line revenue |
| `GrossProfit` | Revenue minus cost of goods/services |
| `GrossProfitMargin` | Compute as GrossProfit / Revenue |
| `NetIncome` | Bottom line |
| `TotalExpenses` | Operating expenses subtotal |

### Transaction Register

Pull the **General Ledger entries** or **Transaction List** for the target month.
This is the line-item detail used for uncategorized flagging and duplicate detection.

Key fields:
| Field | Notes |
|---|---|
| `Date` | Transaction date |
| `Type` | Invoice, Payment, Bank entry, Journal, etc. |
| `AmountDC` | Amount in division currency (EUR) |
| `GLAccount` | The ledger account code and description |
| `Description` | Free-text memo or counterparty name |
| `Document` | Attached document count (receipt, invoice) |

## Identifying uncategorized transactions

Flag a transaction as uncategorized if `GLAccount` is:
- A generic suspense account (often "9999" or similar)
- Blank / null
- A "to be categorized" placeholder account
- Bank feed catch-all with no ledger mapping

## Pagination

The CData MCP server may limit result rows per query. For high-volume divisions,
use date-range filters to pull in monthly batches rather than requesting the full
year at once.

## Common data issues

**Split transactions** — a single bank entry allocated across multiple GL accounts
appears as multiple rows with the same date and amount total but different account
codes. These are NOT duplicates. Group rows by `EntryID` — rows sharing an ID are
splits of one transaction.

**Bank import vs. manual entries** — Exact Online bank import entries have a
`BankAccount` reference; manual journal entries do not. Neither is a problem on
its own, but it helps explain missing receipts on manual entries.

**Mollie/PayPal/SumUp transactions already in Exact Online** — if the owner has
set up automatic bank import from their payment processor, those settlements already
appear in the ledger. Don't double-count them during reconciliation. Check whether
Exact Online deposit lines have "Mollie", "PayPal", or "SumUp" in the description
before matching against processor reports.

**VAT lines** — Exact Online posts separate VAT debit/credit lines for each
transaction. Exclude VAT lines when computing revenue figures; include them only
in the VAT reconciliation step (Step 5 of month-end-prep).

**Retainer / prepayment transactions** — a customer prepayment is a liability
(deferred revenue) until the work is delivered. Flag any large credit-side entries
on a deferred revenue account for the owner's awareness.

**SEPA batch payments** — a single SEPA batch file may cover multiple vendor
payments but appear as one bank entry in Exact Online. Drill into the batch detail
to reconcile individual payees against AP invoices.
