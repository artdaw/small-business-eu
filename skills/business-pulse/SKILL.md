---
name: business-pulse
description: >
  Produces a one-page cross-functional business snapshot for EU SMB owners —
  cash position (Exact Online), sales trend (Mollie/PayPal/SumUp), pipeline
  movement (HubSpot or Brevo/Pipedrive), this week's commitments (Calendar),
  urgent watch-list items (Gmail/Slack), upcoming VAT deadline, and the single
  most important thing needing attention today. All figures in EUR. Proactively
  tries every available connector and gracefully scopes to whatever is connected.
  Trigger when the user asks how the business is doing, wants a snapshot, a
  weekly summary, a Monday brief, or says anything like "what am I missing"
  or "catch me up on the business."
---

# Business Pulse

One prompt, one page. Pull live data from every connected tool, synthesize it into a single scannable brief, and surface the single most important thing to act on today. Do the work — don't ask the user to help find the data.

## Step 1 — Pull data in parallel

**Dispatch all connector calls in a single parallel batch** — see `reference/data_sources.md` for the exact tool-to-metric mapping. Do not pull serially; latency turns a 30-second skill into a painful wait.

Connectors to attempt simultaneously:

- **Exact Online** — cash balance (EUR), MTD revenue, outstanding receivables, overdue invoices, VAT due date
- **Mollie / PayPal / SumUp** — 7-day settlements, sales trend, failed/pending transactions
- **HubSpot** — pipeline by stage, deals moved/closed, deals gone cold, new leads
- **Brevo** — CRM deals, recent email campaign performance (if HubSpot not connected)
- **Pipedrive** — pipeline by stage (if HubSpot not connected)
- **Google Calendar / Outlook Calendar** — key meetings, deadlines, events this week and next 7 days
- **Gmail / Outlook Mail** — threads flagged urgent, customer complaints, time-sensitive requests
- **Slack / Teams** — urgent internal signals, threads needing owner attention

If a connector errors or returns no data, record it internally and move on. Never block the pulse on a single bad integration.

**Exact Online fallback**: if Exact Online returns an error or empty response, mark the Cash section "n/a — Exact Online unavailable" and proceed. Do not retry or ask the user to reconnect.

**Gmail/Outlook fallback**: auth is intermittently flaky. If the call errors, skip the Watch List section silently and note "Mail unavailable" in the appendix.

## Step 2 — Compute metrics

Read `reference/thresholds.md` for red/yellow/green cutoffs. Compute:

- **AR aging** — open Exact Online invoices grouped by days since due date (0–30, 31–60, 61+)
- **Pipeline coverage** — HubSpot / Pipedrive weighted pipeline ÷ monthly revenue target
- **Revenue trend** — this month's Exact Online revenue vs. prior month (or 7-day Mollie/PayPal/SumUp vs. prior 7 days)
- **VAT due** — next VAT filing deadline from business context (monthly or quarterly)

Assign a 🟢/🟡/🔴 status to each section. If a source returned nothing, mark the metric "n/a" and note it in the appendix.

## Step 3 — Flag risks proactively

Scan for actionable items. Every risk entry must name a specific record and a next step.

- Exact Online invoices past due > 30 days — name customer, amount, days overdue
- HubSpot / Pipedrive deals with no activity in 7+ days, or close date in past but still open
- Gmail / Outlook threads marked urgent or containing "escalation," "complaint," "cancel," "refund"
- Failed or pending Mollie/PayPal/SumUp transactions > €500
- VAT deadline within 14 days — flag with estimated liability from last close-month run

## Step 4 — Compose the output

Use the exact template in `reference/output_template.md`. Include only sections where real data exists — omit headers for connectors that weren't available. Adapt depth to context: a casual "how are we doing" gets a fuller report; "quick snapshot before a call" gets a tighter one.

Cross-connector synthesis is where this skill earns its keep. If a Slack message connects to a stalled Pipedrive deal, surface that link in the #1 Priority section.

Writing rules:
- Numbers lead, words follow. Never write "revenue is healthy" — write "€43k this month, ▲ 8% MoM."
- Every number carries a delta vs. the prior period where available.
- Names and euros, not adjectives. "€4,200 from Acme BV, 23 days overdue" beats "some concerning receivables."
- No filler. If a section has nothing worth reporting, write "No material changes" and move on.

## Step 5 — Export and share (once)

After presenting the pulse, offer once:
- "Want me to save this as a file?" (use Files connector if available)
- "Should I post this to your Slack?" (only if Slack is connected and the user confirms — Slack write requires explicit approval)

If they say yes, do it. If they say no or don't respond, move on — don't ask again.

## Scope variants

- **"Just cash" / "financial check"** → only Cash & Finance + AR-related risks
- **"Pipeline only" / "deals check"** → only Pipeline section + stalled-deal risks
- **"Watch list" / "anything urgent"** → only Watch List + all risks, no metric sections
- **"Quick snapshot before a call"** → TL;DR + #1 Priority only, no full sections
- **"VAT check"** → only VAT deadline and position, skip all other sections

## What not to do

- **Do not ask permission before pulling data.** If the skill was invoked, run it.
- **Do not invent or estimate numbers.** If a source returned nothing, say "n/a" explicitly.
- **Do not skip the delta.** A number without a comparison is a missed insight.
- **Do not surface connector errors mid-pulse.** Log them to the appendix.

## Reference files

- `reference/data_sources.md` — exact connector tool → metric mapping with fallbacks
- `reference/thresholds.md` — 🟢/🟡/🔴 cutoffs, tunable per owner
- `reference/output_template.md` — exact markdown structure; do not deviate
- `reference/gotchas.md` — known failure modes
