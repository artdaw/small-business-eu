# Small Business Plugin — EU Edition

Pre-built EU-native small business workflows for [Cowork](https://claude.com/product/cowork) and Claude Code. Install it once and you get **34 skills** spanning finance, compliance, sales, customer ops, hiring, and business intelligence — plus a router that understands plain English.

Built for European SMBs: Exact Online for accounting, Mollie / PayPal / SumUp for payments, HubSpot / Pipedrive / Brevo for CRM, Calendly for scheduling, and EU-first compliance tools for VAT returns, GDPR, and e-invoicing.

You don't need to memorize anything. Just tell Claude what you need — "I need to file my VAT return," "a customer wants their data deleted," "what should I charge?" — and the router figures out the right skill and walks you through it. Every workflow that touches money or customers pauses for your approval, so nothing happens without your say-so.

> **Important**: This plugin assists with small business workflows but does not provide financial, tax, legal, or privacy advice. All outputs should be reviewed by you and, where appropriate, a qualified professional before use.

**Author:** Gleb Galkin · [artdaw.com](https://artdaw.com) · me@artdaw.com

---

## Installation

This repo doubles as its own plugin marketplace (`.claude-plugin/marketplace.json`). Install it locally from the repo root.

### Claude Code

In an interactive session:

```
/plugin marketplace add /Users/gleb/Developer/smb-claude
/plugin install small-business-eu@smb-claude
/reload-plugins
```

Or from the CLI:

```bash
claude plugin marketplace add /Users/gleb/Developer/smb-claude
claude plugin install small-business-eu@smb-claude
```

Because the marketplace `source` is the repo itself, any edits you make to the skills here are picked up by the installed plugin — handy while developing.

### Cowork

Add the marketplace from the repo (or a git remote, once published) and install the `small-business-eu` plugin.

Once installed, say **"set me up"** to run the `smb-onboard` skill — it'll help Claude understand your business, your country, your VAT situation, and the tools you already use.

---

## What you'll need to connect

Connectors are configured in [`.mcp.json`](.mcp.json). Run `/smb-onboard` or ask Claude to "set me up" to walk through it.

**Core tools** (connect these first for the best experience):
- **Exact Online** — powers all financial workflows (cash forecasts, margins, month-end close, VAT prep, e-invoicing). *Requires the CData MCP server — see setup below.*
- **Mollie** — payment data, invoice settlements, SEPA/iDEAL/Bancontact refunds, and recurring mandates
- **PayPal** — transaction data, invoices, disputes, and refunds (EU legal entity)
- **SumUp** — card and point-of-sale payment data

**CRM & marketing:**
- **HubSpot** — CRM, leads, campaigns, and customer support tickets
- **Brevo** (formerly Sendinblue) — EU-native alternative to HubSpot for email, contacts, and CRM deals
- **Pipedrive** — EU-native (Estonia) CRM, widely used in northern and western Europe

**Scheduling:**
- **Calendly** — meeting scheduling for leads and customers; generates booking links from the call-list workflow

**Communication:**
- **Gmail / Microsoft 365 (Outlook)** — email drafts, ticket handling, contract review, DSAR responses
- **Google Calendar / Outlook Calendar** — meeting prep, call blocking, weekly commitments
- **Slack** — brief delivery and notifications
- **Canva** — generates on-brand social assets

**File storage & notes:**
- **Google Drive / OneDrive** — file storage and templates
- **Notion** — docs, wikis, and databases (SOPs, project notes, CRM-adjacent tracking)

**Contracts & signatures:**
- **DocuSign** — contract review from pending envelopes

You don't need all of these to start. Connect one or two and you'll immediately see value — the plugin tells you when connecting another tool would unlock more.

---

## Configuring connectors (`.mcp.json`)

Most connectors need **no developer setup** — you authenticate with a normal in-session sign-in.

- **OAuth-in-session (just click sign-in):** Mollie, PayPal, SumUp, HubSpot, Brevo, Canva, DocuSign, Calendly, Slack, Microsoft 365, Notion. Nothing to configure.
- **Google (Gmail / Calendar / Drive):** connect these through your **Claude / Cowork account connector settings** (Settings → Connectors → Google), not through this plugin. It's a one-click "Sign in with Google" — no Google Cloud project or OAuth credentials required. The skills automatically use whichever Google connector your account exposes.
- **Needs credentials (only these two):** Exact Online and Pipedrive use stdio servers that require your own keys. Set them via environment variables using `${VAR}` expansion, then restart Claude Code and run `/reload-plugins`.

| Server | Env vars to set |
|---|---|
| `exact-online` | `EXACTONLINE_CLIENTID`, `EXACTONLINE_CLIENTSECRET`, `EXACTONLINE_DIVISION` |
| `pipedrive` | `PIPEDRIVE_API_TOKEN` |

Example `~/.zshrc`:

```sh
# Exact Online — register an OAuth app at https://apps.exactonline.com
export EXACTONLINE_CLIENTID="…"
export EXACTONLINE_CLIENTSECRET="…"
export EXACTONLINE_DIVISION="…"          # your division code

# Pipedrive
export PIPEDRIVE_API_TOKEN="…"
```

> **Why Google isn't in `.mcp.json`:** Google's OAuth servers don't support dynamic client registration, so wiring Google's raw MCP endpoints directly would force every user to create their own Google Cloud OAuth app — not practical. The managed account connector handles this for you with a normal sign-in.

### EU Sovereignty Tier (optional)

For owners who want to avoid US-headquartered tech entirely:

- **Nextcloud** (replaces Google Drive/Calendar/Mail): self-hosted or EU-hosted; community MCP available (`npx nextcloud-mcp-server`). Set `NEXTCLOUD_URL`, `NEXTCLOUD_USERNAME`, and `NEXTCLOUD_APP_PASSWORD`.
- **Proton Mail** (replaces Gmail): end-to-end encrypted; community MCP via Proton Bridge running locally.

Add these to `.mcp.json` with `${VAR}` references — they're optional and used automatically if connected.

---

## How it works

Two layers working together:

1. **Skills** — the building blocks and workflows. Some skills do one thing well (forecast cash, prep a VAT return, audit GDPR compliance); others chain those together into multi-step recipes with checkpoints where you approve before anything happens. There are 34 in total.

2. **The Router** (`smb-router`) — the front door. You talk to Claude in plain English. The router listens, figures out which skill fits, and gets you there. You never need to memorize a skill name.

---

## All 34 skills

Every skill that touches money or customers pauses for your approval before acting. Skills marked **▶ workflow** chain several building blocks together; the rest are focused building blocks. The router and onboarding skills sit on top.

### Front door

| Skill | What it does |
|---|---|
| **smb-router** | The front door. Routes plain-English requests to the right skill and suggests what to try next. |
| **smb-onboard** | Walks you through connecting tools, runs a demo recipe, captures your EU business context (country, VAT number, headaches), shows a GDPR data notice, sets a weekly cadence. |

### Money & finance

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **cash-flow-snapshot** | 30/60/90-day EUR cash forecast with confidence bands and VAT-deadline flag; CSV fallback. | One of Exact Online / Mollie / PayPal / SumUp | Others as secondary |
| **invoice-chase** | Drafts overdue-invoice reminders with SEPA references; sends via Mollie or PayPal with approval. | Exact Online | Mollie, PayPal, Gmail |
| **margin-analyzer** | Unit economics by product in EUR with inflation benchmarks and pricing scenarios. | Exact Online | Mollie, PayPal, SumUp |
| **month-end-prep** | Reconciles Exact Online vs Mollie/PayPal/SumUp, VAT check, P&L narrative, close packet. | Exact Online | Mollie, PayPal, SumUp |
| **vat-return-prep** | EU VAT return: categorise by rate, flag reverse-charge, check OSS/IOSS, per-country summary (NL/DE/BE v1). | Exact Online | Mollie, PayPal |
| **tax-season-organizer** | EU VAT mode (delegates to vat-return-prep) or US mode (quarterly estimated tax + 1099-NEC prep). | Exact Online | Mollie, PayPal |
| ▶ **plan-payroll** | Cash forecast + overdue-invoice chase so you know payroll is covered. | Exact Online | Mollie, PayPal, SumUp, Mail |
| ▶ **month-heads-up** | 30-day cash outlook with early risk flags. | Exact Online | Mollie, PayPal |
| ▶ **close-month** | Month-end close: reconcile, flag gaps, write P&L, export packet. | Exact Online | Mollie, PayPal, SumUp |
| ▶ **price-check** | Margin-by-product table and three pricing scenarios. | Exact Online | Mollie, PayPal |
| ▶ **tax-prep** | Routes EU owners to VAT return prep; non-EU to estimated tax / 1099s. | Exact Online | Mollie, PayPal |

### Compliance (EU)

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **gdpr-audit** | RoPA draft, gap checklist (privacy policy, DPAs, retention, DSARs), DSAR response drafting. | — (standalone) | HubSpot, Brevo, Gmail |
| **einvoice-export** | Structured XML invoice (ZUGFeRD / XRechnung / PEPPOL BIS), EN16931 validation, plain-English error flags. | — (standalone) | Exact Online |
| **contract-review** | Plain-English contract review with risk flags, severity ratings, and a marked-up redline DOCX. | — (works with file upload) | Gmail, DocuSign |
| ▶ **review-contract** | Red-flag review with severity ratings and a marked-up docx/PDF of suggested redlines. | — (file upload) | DocuSign |

### Sales & marketing

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **lead-triage** | Scores leads by engagement/fit/urgency; ranked call list with talking points. | HubSpot or Pipedrive or Brevo | Gmail, Calendar, Calendly |
| **content-strategy** | Analyzes sales to find top performers; prioritized 30-day content brief. | Exact Online or Mollie | HubSpot, Brevo |
| **canva-creator** | Executes an approved brief: posting calendar, Canva assets, caption copy, CRM staging. | Canva, HubSpot or Brevo | — |
| ▶ **call-list** | Top-5 leads to call today with talking points, calendar blocks, or Calendly links. | HubSpot or Pipedrive or Brevo | Mail, Calendar, Calendly |
| ▶ **run-campaign** | End-to-end campaign: sales analysis → content brief → Canva assets → CRM send. | HubSpot or Brevo, Canva | Exact Online, Mollie |
| ▶ **sales-brief** | Top and bottom sellers with a 2-week content brief. | Exact Online or Mollie | HubSpot, Brevo |

### Customers & operations

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **customer-pulse** | Aggregates disputes, CRM tickets, and email sentiment into a themes report. | — (degrades gracefully) | Mollie, PayPal, HubSpot, Brevo, Gmail |
| **ticket-deflector** | Reads a customer email, pulls Mollie/PayPal status, drafts a tone-matched reply, issues refund with approval. | Mollie or PayPal, HubSpot or Brevo, Mail | — |
| **crm-maintenance** | Keeps the CRM current: creates/updates contacts and deals, logs calls and notes, flags stale records. | HubSpot or Pipedrive or Brevo | Gmail, Calendar |
| ▶ **customer-pulse-check** | Feedback themes synthesized into a top-3 fixable-issues list with response templates. | Mollie or PayPal or HubSpot | — |
| ▶ **handle-complaint** | End-to-end complaint resolution: pull context, draft response, suggest fix. | — (pasted text) | Gmail, HubSpot, Brevo, Mollie, PayPal |
| ▶ **crm-cleanup** | CRM hygiene: stale deals, duplicates, missing fields — fixes what you approve. | HubSpot or Pipedrive or Brevo | — |

### Hiring

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **job-post-builder** | Builds a hiring packet: job post, interview guide with scoring rubric, offer letter template. | — (standalone) | DocuSign, Google Drive |

### Business intelligence

| Skill | What it does | Required | Optional |
|---|---|---|---|
| **business-pulse** | One-page snapshot: cash (EUR), sales trend, pipeline, VAT-deadline watch, commitments, #1 priority. | — (degrades gracefully) | Exact Online, Mollie, PayPal, HubSpot, Pipedrive, Brevo, Calendar, Calendly, Gmail, Slack |
| ▶ **monday-brief** | Monday briefing: cash, sales, pipeline, VAT watch, week ahead, top 3 to-dos. | — (degrades gracefully) | All connectors |
| ▶ **friday-brief** | Friday end-of-week pulse: revenue vs last week, wins, things to watch. | Mollie or PayPal or HubSpot | — |
| ▶ **quarterly-review** | Full QBR narrative: revenue, margin, customer health, opportunities, risks. | Exact Online | Mollie, PayPal, HubSpot |

---

## Customizing

- **Add business context** — Drop your country, VAT number, industry, products, and customers into the skill files (or let `smb-onboard` capture it once).
- **Adjust thresholds** — Tune the alert thresholds in `business-pulse` and `cash-flow-snapshot` for your scale (amounts are in EUR).
- **Swap connectors** — The CRM skills support HubSpot, Pipedrive, or Brevo — configure whichever you use.
- **EU sovereignty** — Add Nextcloud or Proton Mail to `.mcp.json` if you prefer EU-only infrastructure.
- **Iterate live** — Because the marketplace source is this repo, edits to `skills/*` take effect after `/reload-plugins` with no reinstall.
