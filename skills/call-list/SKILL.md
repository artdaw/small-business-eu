---
name: call-list
description: Ranks the top-5 leads most worth calling today, supplies talking points from email history, blocks time on the calendar or generates a Calendly link, and drafts follow-up messages. Works with HubSpot, Pipedrive, or Brevo. Accepts optional count and date arguments.
allowed-tools: Read, WebFetch, Bash
---

Run the lead prioritization. Scan the pipeline, rank by urgency and opportunity, pull relevant email context, and get the owner ready to make calls.

Parse arguments:
- `--n` (default: `5`) — number of leads to surface (1–10)
- `--date` (default: today) — date to build the call list for (`YYYY-MM-DD`)

## Step 1 — Pipeline scan

Using the `lead-triage` skill workflow:

1. Pull open deals and contacts with activity in the last 30 days from the connected CRM (HubSpot → Pipedrive → Brevo, in that order).
2. Pull email threads from Mail for each lead (last 3 emails per contact).
3. Score each lead on:
   - **Recency**: days since last owner touchpoint (lower = better)
   - **Stage**: how close to close (later stage = higher priority)
   - **Signal**: any recent inbound activity (email open, reply, calendar hold, web visit)
   - **Value**: deal size from CRM (in EUR)

## Step 2 — Rank and select top N

Rank all scored leads and select the top `--n`. For ties, prefer leads with unanswered inbound signals.

For each selected lead, produce a call card:

```
{Rank}. {Contact Name} — {Company}
Deal: €{amount} | Stage: {stage} | Last contact: {X days ago}
Signal: {most recent activity}

TALKING POINTS
• {point from email/deal context}
• {point from email/deal context}
• {open question to ask}

GOAL FOR THIS CALL: {one sentence — advance to next stage / re-engage / close}
```

## Step 3 — Schedule the calls

For each lead on the list, offer two options:

**Option A — Calendar block:** Block 20 minutes on the owner's calendar for the target date.
```
{time slot} — Call: {Contact Name} ({Company})
```
Wait for owner to confirm which calls to block before creating calendar events.

**Option B — Calendly link (if connected):** Generate a Calendly scheduling link for the lead to pick their own slot.
```
Calendly link for {Contact Name}: {URL}
```
This works well when the owner prefers the lead to self-schedule rather than receiving a booked invite.

Ask the owner which method they prefer before creating anything.

## Step 4 — Draft follow-ups

For any lead that has an unanswered email older than 3 days, draft a brief follow-up:
```
Subject: Re: {thread subject}

Hi {first name},

{One sentence referencing prior conversation}. {One sentence with a clear next step or question}.

{Sign-off}
```

## Connector failures

If no CRM is reachable (HubSpot, Pipedrive, and Brevo all unavailable), stop and tell the owner — lead scoring requires CRM data. If Mail is unreachable, skip email context and follow-up drafts and note it in output. If Calendar is unreachable, skip calendar blocking and note it.

## Approval gates

- **Never send emails automatically.** Present drafts for owner approval only.
- **Never create calendar blocks without owner confirmation** — show the proposed list first.
- **Never update CRM deal stages automatically.**
- **Never send a Calendly link without owner approval** — show the link and ask before sharing.

## Output

Present the ranked call list with talk tracks. Then show scheduling options (calendar block or Calendly link) and ask for confirmation. Then show follow-up drafts and ask which to send.
