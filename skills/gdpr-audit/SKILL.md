---
name: gdpr-audit
version: 1.0.0
description: >
  Helps EU SMB owners understand and document their GDPR obligations. Conducts
  a structured interview about data flows, cross-references what's visible in
  connected tools (HubSpot, Brevo, Gmail), produces a draft Record of Processing
  Activities (RoPA), flags compliance gaps, and can draft a DSAR (Data Subject
  Access Request) response email. Works standalone via interview — no connectors
  required. Not legal advice.

  Trigger when the user says: "GDPR audit," "data compliance," "privacy check,"
  "RoPA," "record of processing," "DSAR," "subject access request," "data
  deletion request," or "is my data handling compliant."
---

# GDPR Audit

> **Framing:** This skill produces prep material and a starting point for
> compliance — not legal advice. For binding decisions, consult a privacy lawyer
> or a certified Data Protection Officer (DPO).

## Quick start

```
User: "GDPR audit"
→ Explain what we'll cover; confirm the owner wants to proceed
→ Interview: what data do you hold, where, how long, for what purpose?
→ Cross-reference connected tools for visible data categories
→ Produce draft RoPA + gap checklist
→ Offer to draft a DSAR response if a request is pending
```

## Step 1 — Frame the session

Briefly explain what a GDPR audit covers:

> "I'll ask you about the personal data your business handles — customers, employees, suppliers — and produce a draft Record of Processing Activities (RoPA). This is legally required for most EU businesses. I'll also flag any obvious gaps. This takes about 15–20 minutes and ends with a document you can refine with your accountant or privacy adviser."

Ask: "Ready to start?"

## Step 2 — Interview: data mapping

Ask the following questions one at a time. Wait for the full answer before moving on.

**Block A — Customer data**
1. "What personal data do you collect from customers? (e.g. name, email, address, phone, payment details, purchase history)"
2. "Where is this data stored? (e.g. HubSpot, Brevo, Exact Online, email, spreadsheet, paper)"
3. "How long do you keep customer data after a relationship ends?"
4. "Do you share customer data with any third parties? (e.g. email marketing platforms, payment processors, accountants, delivery companies)"

**Block B — Employee data (if applicable)**
5. "Do you have employees? If yes, what personal data do you hold about them? (e.g. name, address, BSN/national ID, salary, contract, performance records)"
6. "Where is employee data stored, and for how long after they leave?"

**Block C — Website and tracking**
7. "Do you have a website? Do you use cookies, analytics (e.g. Google Analytics), or a contact form?"
8. "Do you have a privacy policy and cookie notice on your website?"

**Block D — Supplier and partner data**
9. "Do you hold personal data about contacts at suppliers or partners? (e.g. names, email addresses)"

## Step 3 — Cross-reference connected tools

For each connected tool, look for categories of personal data that are being processed:

- **HubSpot / Brevo / Pipedrive**: contacts list — check for: email addresses, phone numbers, job titles, company names, notes with personal details. Flag how many contacts exist and when they were last updated.
- **Gmail / Outlook**: scan subject lines for common DSAR-related phrases ("cancel my account," "delete my data," "send me my data") in the last 90 days. Flag any that may be unanswered DSARs.
- **Exact Online**: customer and supplier data in the AR/AP ledger — names, addresses, VAT numbers, bank account numbers.

Note which tools are connected and what categories of data appear. Do not read or surface individual personal records beyond counts and categories.

## Step 4 — Identify third-party processors

Based on the connected tools and interview answers, list likely third-party data processors:

- **Mollie / PayPal / SumUp** — payment data (names, amounts, email); EU data processors
- **HubSpot** — CRM data; US-headquartered; transfers to US covered by DPA + SCCs
- **Brevo** — email/CRM; EU-headquartered (France); GDPR-native
- **Canva** — design assets; may contain customer names in designs; US-headquartered
- **DocuSign** — signed contracts with personal data; US-headquartered
- **Slack** — internal communications; US-headquartered
- **Google Workspace** — email, calendar, drive; US-headquartered; EU data residency available
- **MS365** — email, calendar, files; US-headquartered; EU data residency available

Flag US-headquartered processors: "This tool is US-headquartered. Transfers to the US require either an adequacy decision, SCCs (Standard Contractual Clauses), or the owner's active DPA. Confirm with your adviser that the appropriate safeguard is in place."

## Step 5 — Produce the draft RoPA

Structure the Record of Processing Activities as a table:

| Processing activity | Categories of data | Lawful basis | Retention period | Recipients / processors | Transfer outside EEA? |
|---|---|---|---|---|---|
| Customer relationship management | Name, email, phone, purchase history | Legitimate interest / Contract | 3 years after last purchase | HubSpot (US, SCCs) | Yes — US |
| Invoice and payment processing | Name, address, bank details, VAT number | Contract / Legal obligation | 7 years (tax law) | Exact Online (EU), Mollie (EU) | No |
| Email marketing | Name, email, marketing preferences | Consent | Until opt-out | Brevo (EU) | No |
| Employee records | Name, address, national ID, salary | Contract / Legal obligation | 5 years post-employment | Payroll provider | Confirm |
| Website analytics | IP address, browsing behaviour | Consent (cookie) | 26 months | Google Analytics (US, SCCs) | Yes — US |

Populate based on the interview answers and connected tool scan. Flag cells where information is missing as "**[confirm with owner]**".

## Step 6 — Gap checklist

Flag the most common EU SMB GDPR gaps:

- [ ] **Privacy policy**: does the owner have a publicly accessible, up-to-date privacy policy?
- [ ] **Cookie notice**: if the website uses cookies, is there a compliant cookie banner with opt-in for non-essential cookies?
- [ ] **Data retention policy**: are retention periods defined and actually enforced?
- [ ] **DPA agreements**: are Data Processing Agreements (DPAs) signed with all third-party processors?
- [ ] **US transfers**: are SCCs or equivalent safeguards in place for US-headquartered tools?
- [ ] **DSAR process**: is there a process for responding to Data Subject Access Requests within 30 days?
- [ ] **Breach notification**: is there a process for identifying and notifying the supervisory authority of a data breach within 72 hours?
- [ ] **Consent records**: if relying on consent as a lawful basis, are consent records kept?

For each gap, note its severity: 🔴 High (material non-compliance), 🟡 Medium (should fix), 🟢 Low (best practice).

## Step 7 — DSAR mode (if requested)

If the owner has received a Data Subject Access Request or data deletion request:

1. Ask: "What did the customer request — a copy of their data, deletion, or correction?"
2. Ask: "What is the customer's name and email?"
3. Confirm the 30-day response deadline from the date of receipt.
4. Draft a response email:
   - **Access request**: acknowledge receipt, confirm identity verification step, state the 30-day timeline, list what data categories you hold.
   - **Deletion request**: acknowledge receipt, confirm deletion timeline, note any data you must retain by law (e.g. invoice records for tax purposes).
   - **Correction request**: acknowledge receipt, confirm the correction will be made, state the timeline.

Show the draft. Do not send without owner approval.

## Guardrails

- **Not legal advice.** State this clearly at the start and in every output header.
- **No individual personal records.** Scan for categories and counts — never surface individual people's data from connected tools.
- **Approval before any action.** If the DSAR mode drafts an email, always show it first.
- **Flag, don't fix.** Surface gaps and explain them; the owner and their adviser decide what to do.

## Reference files

- [reference/lawful-basis.md](reference/lawful-basis.md) — six GDPR lawful bases explained for SMBs
- [reference/retention-periods.md](reference/retention-periods.md) — common data types and typical retention periods (EU)
- [reference/dsar-templates.md](reference/dsar-templates.md) — response templates for access, deletion, and correction requests
- [reference/gotchas.md](reference/gotchas.md) — common SMB GDPR mistakes and how to address them
