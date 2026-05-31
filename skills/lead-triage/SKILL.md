---
name: lead-triage
version: 0.2.0
description: >
  Scores inbound leads by engagement signals, company fit, and urgency
  markers to produce a "call these 5 today" list with talking points, drafts
  the follow-ups, and blocks Calendar time. Works with HubSpot (primary),
  Pipedrive (secondary), or Brevo CRM (tertiary). Use when the user asks to
  prioritize leads, who to call first, or about their pipeline.
---

# Lead Triage

## Quick start

Pull inbound leads from the connected CRM, score them, and surface a ranked call list with talking points. Drafts follow-ups and proposes calendar slots — never sends or books without owner approval.

```
User: "prioritize my leads"
→ Pull contacts from HubSpot (or Pipedrive / Brevo if HubSpot not connected)
→ Score each across engagement, company fit, urgency, recency
→ Return ranked list (size adapts to volume) with talking points
→ Offer to draft follow-ups and propose calendar slots
```

## Workflow

1. **Select CRM connector.** Try in this order:
   - **HubSpot**: fetch contacts with `lifecyclestage` = `Lead` or `MQL` and `hs_lead_status` ≠ `Unqualified`. Use the field list in [reference/hubspot-scoring.md](reference/hubspot-scoring.md).
   - **Pipedrive**: fetch open deals in early pipeline stages (lead, prospect, qualified). Pull associated persons. Use the scoring model below adapted to Pipedrive's stage names.
   - **Brevo**: fetch CRM deals in early stages. Pull associated contacts.
   - If none are connected: *"No CRM is connected — connect HubSpot, Pipedrive, or Brevo and try again."*

2. **Clarify if trigger is ambiguous.** If the user said only "pipeline" without a qualifier, ask: *"Quick pipeline overview (deal stages + total value) or prioritized call list?"* — then route accordingly.

3. **Score each lead.** Apply the four-dimension model:
   - **Engagement** — email replies, opens, site visits in the CRM (last 30 days only)
   - **Company fit** — industry and employee count vs. owner's ICP (default: any industry, 1–50 employees)
   - **Urgency** — lead age, stage duration, notes containing "urgent / ASAP / deadline / budget approved"
   - **Recency penalty** — subtract points if last activity was <24 hours ago (already touched today)

   See [reference/hubspot-scoring.md](reference/hubspot-scoring.md) for weights; apply same weights to Pipedrive/Brevo fields.

4. **Build the ranked list.** Sort descending by composite score. Adapt list size to volume:
   - ≤10 leads → show all
   - 11–30 leads → show top 5
   - >30 leads → show top 8

   For each lead: name, company, score, one-paragraph talking point, last activity summary. If engagement signals are all >30 days old, flag: *"Engagement signals are stale — approach as cold outreach."*

5. **Offer follow-up drafts.** Ask: *"Draft follow-ups for any of these?"* If yes, write one email per selected lead, matching the tone of their last outbound thread in Mail. Show draft; do not send.

6. **Offer calendar slots.** Ask: *"Propose call slots for any of these?"* If yes:
   - Check Google Calendar or Outlook Calendar for open 30-minute windows in the next two business days.
   - If Calendly is connected, offer to generate a Calendly scheduling link instead — often less friction than a proposed time slot.
   - Propose two options per lead. Do not create events — the owner books.

## Approval gates

- **Never send an email.** Draft only; owner sends from their inbox.
- **Never create calendar events.** Propose times or generate Calendly link; owner books.
- **Never change lifecycle stage or mark a lead Unqualified** unless the owner explicitly asks.
- **Never include closed/won or customer-stage contacts** in the lead list.
- **If zero leads match the filter**, explain why and offer to check what stages are in use — do not fabricate a list.

## Reference

- [reference/hubspot-scoring.md](reference/hubspot-scoring.md) — field names, scoring weights, ICP defaults (apply to Pipedrive/Brevo too)
- [reference/gotchas.md](reference/gotchas.md) — edge cases: stale data, zero leads, pipeline disambiguation
