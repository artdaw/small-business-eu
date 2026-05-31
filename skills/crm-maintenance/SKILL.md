---
name: crm-maintenance
description: >
  Keeps the CRM current without the owner opening it: creates and updates
  contacts and deals from email and calendar context, logs notes and calls,
  and flags stale records. Works with HubSpot (primary), Pipedrive (secondary),
  or Brevo CRM (tertiary). The "stop doing data entry" skill. Use when the user
  asks to update the CRM, log a call, clean up contacts, or add context to a deal.
---

# CRM Maintenance

## Quick start

Pull context from the referenced email or calendar event, resolve the right CRM contact and deal, log the activity, and surface what changed. For a deal cleanup, audit the deal against recent email/calendar activity and propose updates — never apply them without approval.

```
User: "log this call to the Acme deal"
→ Read the most recent completed calendar event
→ Confirm attendees map to the Acme deal's contacts in HubSpot / Pipedrive / Brevo
→ Write a call activity on the Acme deal
→ Report: "Logged call to Acme Q2 Expansion. [deal link]"
```

## Workflow

### Step 0 — Select CRM connector

Try in this order:
1. **HubSpot** — check if connected; use if available
2. **Pipedrive** — fall back if HubSpot is not connected
3. **Brevo CRM** — fall back if neither HubSpot nor Pipedrive is connected

Announce which CRM is active at the start of the session if it's ambiguous. Do not mix data between CRMs in a single operation.

### Step 1 — Identify intent

Decide which of three paths applies from the user's message and context:
- **Email path** — "update my CRM", "add this to the deal", or any reference to an email thread
- **Call path** — "log this call", "log the meeting", or any reference to a calendar event
- **Cleanup path** — "clean up the CRM", "is this deal up to date", or any request to audit a specific deal

If the intent is ambiguous, ask which path before proceeding.

### Step 2 — Gather context

- Email path: read the thread (subject, participants, last 1–3 messages). Identify the primary external contact.
- Call path: read the calendar event (title, attendees, time, description). If no event was specified, use the most recent completed meeting in the last 24 hours and confirm with the user.
- Cleanup path: pull the deal (stage, amount, close date, next-step, associated contacts, activities in last 60 days), plus the last 14 days of email threads and calendar events involving the deal's contacts.

### Step 3 — Resolve the CRM contact and deal

For email/call paths:
- Search CRM contacts by email address. If a contact is missing, create it from email signature or calendar invite data — announce creation in chat before writing.
- Find the right deal in this order: (a) explicit match if the user named one, (b) the contact's sole open deal, (c) fuzzy match against the email subject or meeting title — confirm before writing, (d) ask the user if no match. **Never auto-create a deal.**

**HubSpot:** Read [reference/hubspot-fields.md](reference/hubspot-fields.md) for field names, activity types, and association rules before writing.

**Pipedrive:** Use Pipedrive's activity types (call, meeting, email, task). Associate activities to the deal via deal ID. Note Pipedrive's stage names differ from HubSpot — map carefully.

**Brevo:** Use Brevo CRM deal and contact APIs. Activity logging is via notes on the contact or deal record.

If deduplication or deal-resolution feels ambiguous, check [reference/gotchas.md](reference/gotchas.md) before proceeding.

### Step 4 — Execute the action

- Email path: write an email activity with the thread subject as the title and a concise summary as the body. Timestamp to the latest message.
- Call path: write a call activity with the event title, duration, and any notes available. Timestamp to the event start.
- Cleanup path: walk each field per [reference/cleanup-checklist.md](reference/cleanup-checklist.md) and assemble a proposed-changes list. Show current → proposed side-by-side. Write only what the user approves.

### Step 5 — Approval gate

For contact creation and activity logging, announce before writing and surface the result after. For cleanup edits, do not write anything until the user approves the specific changes.

### Step 6 — Report what happened

Tell the user what was written and what's pending. Include a CRM link to the affected deal when possible.

## Approval gates

- **Never delete records.** Not contacts, not deals, not activities. Direct the user to the CRM if they want to delete.
- **Never change deal stage or close a deal without explicit user approval.** Flag and defer.
- **Never create a new deal unprompted.** Ask if the right deal can't be resolved.
- **Announce contact creation before writing.**
- **Side-by-side diffs for cleanup.** Show current value and proposed value; wait for approval per item.

## Reference

- [reference/hubspot-fields.md](reference/hubspot-fields.md) — HubSpot activity types, field names, association rules
- [reference/cleanup-checklist.md](reference/cleanup-checklist.md) — fields checked during a deal cleanup
- [reference/gotchas.md](reference/gotchas.md) — contact resolution, activity summaries, cleanup proposals
