# Onboard checklist

## The six interview questions

Ask one at a time. Wait for the full answer before moving on. One follow-up is fine if an answer is vague; do not drill further.

1. **Country.** "Which country is your business based in?" Store as an ISO code (NL, DE, BE, FR, ES, IT, etc.). This drives VAT skill behaviour, e-invoicing format, and tax routing.
2. **Industry and business type.** "What kind of business do you run? Give me the one-liner."
3. **Team size.** "How many people work with you, including yourself?"
4. **Top three headaches.** "What are your three biggest headaches right now — the things that eat your time or keep you up at night?"
5. **Tools already in use.** "Which tools do you already use day-to-day? Things like Exact Online, Mollie, HubSpot, Pipedrive, Gmail, Slack, Calendly…"
6. **Preferred cadence.** "How would you like me to check in — daily, weekly, or only when you ask?"

If the owner is short on time, compress to questions 1, 2, 4, and 5 — those four feed the most downstream skills.

---

## Connector priority matrix

Map the owner's stated headache to the best two connectors to link first.

| Primary headache | First connector | Second connector | Prove-value recipe |
|---|---|---|---|
| Cash flow / invoicing | Exact Online | Mollie or PayPal | `cash-flow-snapshot` |
| Customer follow-up | HubSpot or Pipedrive | Gmail or Outlook | `crm-maintenance` (read-only demo) |
| Getting more customers / marketing | Brevo or HubSpot | Canva | `lead-triage` demo |
| Hiring / job posts | Gmail or Outlook | Google Calendar or Outlook Calendar | `job-post-builder` |
| Staying organized / scheduling | Google Calendar or Calendly | Gmail | `business-pulse` |
| VAT / tax compliance | Exact Online | (none needed for second) | `vat-return-prep` |
| GDPR / data compliance | Gmail or Outlook | HubSpot or Brevo | `gdpr-audit` |
| General / unsure | Gmail or Outlook | Exact Online | `cash-flow-snapshot` |

If the owner names a connector not in this table, add it as the second connector and use `business-pulse` as the recipe.

---

## Recipe selection

Run the prove-value recipe immediately after the **first** connector is live — do not wait for the second. Priority order:

1. Exact Online → `cash-flow-snapshot`
2. Mollie → show last 7-day settlement summary
3. HubSpot or Pipedrive → `crm-maintenance` (log-a-note demo, read-only)
4. Brevo → show recent email campaign performance and CRM contact count
5. Gmail or Outlook → search for unread invoice-related emails, surface top 3
6. Google Calendar or Calendly → `business-pulse`

---

## EU sovereignty tier (optional)

For owners who explicitly want EU-only infrastructure (no US tech), mention these after the standard connectors:

- **Nextcloud** (replaces Google Drive + Calendar): self-hosted or EU-hosted; community MCP (`nextcloud-mcp-server`)
- **Proton Mail** (replaces Gmail): end-to-end encrypted, Switzerland; community MCP via Proton Bridge

Only raise these if the owner explicitly asks about data sovereignty or mentions avoiding US tech. Do not push them as the default.

---

## Owner profile — storage format

Write this block to the Cowork session memory directory under the heading `## Business context`. Every other skill reads this section by heading match. Do not rename the heading or change the field names.

```markdown
## Business context

- **Business:** <one-liner — industry, product/service>
- **Country:** <ISO country code, e.g. NL, DE, BE, FR>
- **VAT number:** <VAT registration number, e.g. NL123456789B01 — or "not registered" / "unknown">
- **Size:** <number of people, including owner>
- **Top headaches:** <headache 1> · <headache 2> · <headache 3>
- **Connected tools:** <comma-separated list of active connectors>
- **Weekly cadence:** <trigger phrase and day, e.g. "weekly check-in every Monday">
- **Onboarded:** <YYYY-MM-DD>
```

If a memory file already exists, append or update only the `## Business context` section. Do not touch other content.

**Country drives downstream behaviour:**
- EU country → `tax-prep` routes to `vat-return-prep`; `monday-brief` includes VAT deadline watch; `smb-router` surfaces GDPR and e-invoicing commands
- US → `tax-prep` routes to quarterly estimated tax / 1099 prep
- Unknown → `tax-prep` asks before routing
