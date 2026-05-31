---
name: monday-brief
description: Generates a one-page Monday morning briefing — cash, sales, pipeline, week ahead, VAT watch, top three to-dos. Works with Exact Online, Mollie/PayPal/SumUp, HubSpot/Pipedrive/Brevo, Calendly. Accepts optional post destination and save-to arguments.
allowed-tools: Read, WebFetch, Bash
---

Run the Monday Morning Briefing. Pull from every connector that's live, gracefully degrade when one isn't, and deliver a one-page brief the owner can read in under two minutes.

Parse arguments:
- `--post` (default `none`) — post the brief summary to `slack`, `teams`, or `none`
- `--save-to` (default `files`) — `files` (Google Drive / OneDrive), `desktop` (local), or `both`

## Step 1 — Run business-pulse

Trigger the `business-pulse` skill workflow. It pulls in this order, scoping to whatever is connected:

1. **Cash** — Exact Online balance + last 7 days of net flow (EUR)
2. **Sales trend** — Mollie/PayPal/SumUp last 7 days vs. prior 7 days, % change, top product/service
3. **Pipeline** — HubSpot/Pipedrive/Brevo deals moved, deals stalled (>14 days no activity), new inbound leads
4. **This week's commitments** — Calendar events with external attendees, deliverable deadlines; Calendly meetings booked for the week
5. **Watch-list** — unread Mail flagged "needs reply," Slack DMs awaiting response
6. **VAT watch** — if a VAT filing is due within 21 days, flag it with estimated liability
7. **The 3 things** — the three highest-leverage actions for today, ranked

If a connector is missing, note it in the brief rather than failing.

## Step 2 — Format the one-page brief

Layout (markdown, fits on one screen):

```
# Monday Brief — {Mon DD, YYYY}

## Cash (EUR)
{€X balance · {+/-}€Y net last 7 days · runway note}

## Sales (last 7d vs prior 7d)
{€X total · {+/-}Z% · top product: {name} ({€})}

## Pipeline
{N deals moved · M stalled · K new leads}

## Week ahead
- {Tue 10am} — {Customer X discovery call}
- {Thu EOD}  — {Proposal due to Y}
- {Fri 2pm}  — {Calendly: prospect intro call}
- ...

## VAT watch
{Next filing due: {date} · Estimated liability: €X · [run /vat-return-prep to prepare]}
(omit if no filing due within 21 days)

## Three things that need you today
1. {Highest-leverage action with one-line why}
2. {...}
3. {...}
```

## Step 3 — Save and (optionally) post

1. Save the brief to the chosen `--save-to` location:
   - `files` — Google Drive or OneDrive root, filename `monday-brief-YYYY-MM-DD.md`
   - `desktop` — `~/Desktop/monday-brief-YYYY-MM-DD.md`
   - `both` — both locations
2. If `--post slack` or `--post teams`, post the **Three things** section only (not the full brief) and link to the saved file.
3. Show the full brief in chat regardless of save target.

## Approval gates

- **Saving the file is auto.** No approval needed — it's the owner's own drive.
- **Posting to Slack/Teams requires confirmation.** Show the post draft and wait for "post it" before publishing.
- **Never post if the brief surfaces unflattering numbers** (significant cash drop, deal slipping) without explicitly asking the owner — the channel may have non-leadership members.

## Cadence note

This command is designed to run weekly. When run on Monday at 7am, the output goes straight to the owner's drive and (if configured) Slack/Teams DM.
