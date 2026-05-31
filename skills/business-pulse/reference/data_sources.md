# Data Sources

Exact mapping from each pulse section to the MCP tool that produces it. **Dispatch all calls in a single parallel batch** — do not pull serially.

## Cash & Finance (Exact Online)

| Metric | Tool | Notes |
|---|---|---|
| Cash / bank balance | Exact Online cash/bank balance | Current balance in EUR; show delta vs. prior week if available |
| MTD revenue | Exact Online P&L report | Current month vs. prior month |
| Outstanding receivables | Exact Online AR aging | Filter to open/unpaid |
| AR aging | Exact Online AR aging | Group by days since due: 0–30, 31–60, 61+ |
| Overdue invoices | Exact Online AR aging | Filter to due_date > 30 days past; name customer + amount + days overdue |
| VAT due date | Business context + Exact Online VAT summary | Monthly filers: last day of month following quarter; quarterly filers: varies by country |

**Exact Online fallback**: if Exact Online returns an error or "not connected" state, mark the entire Cash section as "n/a — Exact Online unavailable" and continue. Do not retry.

## Revenue & Sales (Mollie / PayPal / SumUp)

| Metric | Tool | Notes |
|---|---|---|
| 7-day settlement total | Mollie payments (status: paid) | Sum completed settlements in window |
| Sales trend | Mollie payments | This 7 days vs. prior 7 days; compute delta |
| Failed / pending transactions | Mollie payments (status: failed/open) | Flag any > €200 |
| PayPal settlements | PayPal transactions (status: S) | Same as Mollie — sum + trend |
| SumUp payouts | SumUp payouts (status: PAID) | POS sales and payout history |

Use whichever payment connectors are available. If multiple are connected, report combined and per-source.

## Pipeline (HubSpot / Pipedrive / Brevo)

Try HubSpot first; fall back to Pipedrive; fall back to Brevo CRM deals.

| Metric | Tool | Notes |
|---|---|---|
| Pipeline by stage | HubSpot `get_crm_objects` (deals) / Pipedrive deals list | Group by stage; sum amount in EUR |
| Deals closed this week | HubSpot / Pipedrive | Filter closedate in window, stage = closed-won |
| Deals gone cold | HubSpot / Pipedrive | Open deal, no activity 7+ days |
| New leads this week | HubSpot / Pipedrive / Brevo | Filter createdate in window |
| Stalled/slipped deals | HubSpot / Pipedrive | Open deals where closedate < today |

## Commitments (Google Calendar / Outlook Calendar)

| Metric | Tool | Notes |
|---|---|---|
| This week's key items | `list_events` | Filter to current week; surface meetings with customers, deadlines |
| Next 7 days | `list_events` | Forward-looking view; highlight anything with external parties |
| Calendly meetings | Calendly events (if connected) | Scheduled calls with leads or customers |

## Watch List (Gmail / Outlook Mail)

| Metric | Tool | Notes |
|---|---|---|
| Urgent threads | `search_threads` | `is:important OR is:starred` in last 7 days |
| Customer escalations | `search_threads` | "escalation," "complaint," "cancel," "refund," "urgent" in last 7 days |
| Time-sensitive requests | `search_threads` | `is:unread` + "deadline," "ASAP," "today" |

**Mail fallback**: if the call errors (auth flaky), skip Watch List silently and add "Mail unavailable" to the appendix.

## Internal Signals (Slack / Teams)

| Metric | Tool | Notes |
|---|---|---|
| Urgent threads | Slack search (if connected) | Threads with @mentions or urgency signals in owner-relevant channels |
| Action items | Slack search | Messages directed at the owner or tagged for follow-up |

## Risks scan

Run these alongside the metric pulls — don't wait for metrics to finish first.

| Risk | Source | Trigger condition |
|---|---|---|
| Overdue AR | Exact Online invoices | due_date > 30 days past, unpaid |
| Stalled deals | HubSpot / Pipedrive | Open deal, no activity 7+ days |
| Slipped deals | HubSpot / Pipedrive | Open deal, closedate in past |
| Urgent mail threads | Gmail / Outlook | `is:important` or escalation keywords |
| Failed payments | Mollie / PayPal / SumUp | Failed or pending > €200 |
| VAT deadline | Business context | Filing due within 14 days |

## Parallelization

All of the above should fire in a single tool-call batch. A complete pulse is typically 10–18 parallel calls. If one errors, the rest proceed normally and the failed source appears in "Sources unavailable" at the bottom of the pulse.
