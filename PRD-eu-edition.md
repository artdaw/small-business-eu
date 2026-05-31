# PRD: Small Business Plugin — EU Edition

**Status**: Draft  
**Author**: Gleb  
**Date**: 2026-05-17  
**Version**: 0.2  

---

## Problem

The existing `small-business` plugin is built around a US-centric stack: QuickBooks for accounting, Stripe and Square for payments, DocuSign for contracts. European SMBs face three structural mismatches:

1. **Wrong accounting software.** QuickBooks has very low market share in continental Europe. German businesses use Datev or Lexware; Dutch and Belgian ones use Exact Online; French ones increasingly use Pennylane; Spanish ones use Holded. Wiring the plugin to QuickBooks means most EU owners can't connect their actual books.

2. **Wrong payment stack.** Stripe is available in the EU but Mollie dominates Dutch, Belgian, and German e-commerce. iZettle/PayPal Zettle and SumUp own the POS market. Square barely exists here. SEPA bank transfers — not cards — are the default B2B payment rail.

3. **Missing EU obligations.** EU SMBs deal with GDPR data-handling rules, per-country VAT returns (or the EU OSS/IOSS scheme for cross-border e-commerce), Intrastat declarations, and mandatory e-invoicing (already live in Germany and rolling out across the bloc). None of these are in the current skill set.

The goal is a drop-in EU variant of the plugin: same 15-command structure, EU-native connectors, and three new EU-specific skills.

---

## Target Market

Pan-EU SMBs, 1–50 employees, prioritizing:
- **Netherlands, Belgium, Germany** (Mollie + Exact Online strongholds)
- **France** (Pennylane, strong YouSign adoption)
- **Spain / Italy** (Holded, Fattura PA e-invoicing)

Language: English interface, but connector data arrives in EUR and local formats.

---

## Service Substitutions

### 1. Accounting: QuickBooks → Exact Online (primary)

| | QuickBooks | Exact Online |
|---|---|---|
| EU coverage | Low | Netherlands, Belgium, UK, Germany, France, Spain |
| API | Yes (Intuit) | Yes (REST, well-documented) |
| MCP status | Live (ai-inc.quickbooks.intuit.com) | Read-only via CData (`exact-online-mcp-server-by-cdata`); no official native MCP yet |
| VAT | US-centric | EU VAT codes, BTW, MwSt native |

**Secondary options** (same skill, degraded path if Exact not connected):
- **Pennylane** — France, growing API, no MCP
- **Holded** — Spain, REST API, no MCP
- **Datev** — Germany, API exists but restricted; accountant-controlled

**Decision**: Use the CData read-only MCP to start (unblocks financial skill reads immediately); build or wait for a full read-write MCP to enable actions like issuing invoices. Skills fall back to CSV upload when not connected.

### 2. Payments: Stripe → Mollie

| | Stripe | Mollie |
|---|---|---|
| EU headquarters | US (Stripe Inc.) | Netherlands (Mollie B.V.) |
| GDPR storage | US-based, SCCs required | EU-native |
| SEPA Direct Debit | Yes | Yes (core feature) |
| iDEAL, Bancontact, SOFORT | Yes | Yes (primary methods) |
| MCP status | Live (mcp.stripe.com) | **Live — `https://mcp.mollie.com/mcp` (launched Jul 2025)** |

Mollie's MCP server is official and launched. Auth requires an Advanced access token passed as `MOLLIE_API_OAUTH_ORG_TOKEN`. Covers payment links, payment status, customer profiles, and recurring mandates. Future roadmap includes invoicing and subscription management.

### 3. POS / In-person Payments: Square → SumUp

| | Square | SumUp |
|---|---|---|
| EU availability | UK only (limited) | 35+ countries, EU-native |
| MCP status | Live (mcp.squareup.com) | **Live — `https://mcp.sumup.com/mcp` (official, Cloudflare Worker)** |
| Key markets | US | DE, FR, NL, BE, ES, IT, UK |

SumUp's MCP is official (github.com/sumup/sumup-mcp), hosted as a Cloudflare Worker. Exposes `/mcp` (Streamable HTTP) and `/sse` (legacy). Auth: `Authorization: Bearer <access-token>`.

### 4. E-signatures: DocuSign → YouSign

| | DocuSign | YouSign |
|---|---|---|
| HQ | US | France (Paris) |
| GDPR compliance | SCCs required | EU-native, eIDAS-qualified |
| MCP status | Live (mcp.docusign.com) | Third-party only (viaSocket); no official MCP yet |
| eIDAS QES support | No | Yes |

YouSign supports Qualified Electronic Signatures under eIDAS — legally binding across all 27 EU member states without additional paperwork. This is a material upgrade for EU contract workflows, not just a swap. Until an official MCP lands, use DocuSign as the active connector and flag YouSign as the preferred alternative once available.

### 5. PayPal — Keep

PayPal operates an EU legal entity (Luxembourg), supports SEPA, and the `paypal` MCP is already live. Keep as-is.

### 6. Google Workspace (Gmail, Calendar, Drive) — Keep as default; EU alternatives available

Google Workspace has strong EU data residency options and all three services have official MCPs. Keeping as default. However, a significant portion of EU SMBs — particularly in DE, NL, FR — are actively moving off US tech stacks for data sovereignty reasons. Add a documented "EU sovereignty tier" as an opt-in alternative path.

**EU Workspace alternatives and MCP status:**

| Service | Replaces | HQ | MCP status |
|---|---|---|---|
| **Proton Mail** | Gmail | Switzerland | Community MCPs via Bridge (PulseMCP, multiple impls); not official |
| **Proton Drive** | Google Drive | Switzerland | Community MCP (`anyrxo/proton-drive-mcp` on GitHub) |
| **Nextcloud** | Gmail + Drive + Calendar combined | Germany | Community MCP on PyPI (`nextcloud-mcp-server`, 110+ tools, v0.86.3 May 2026); comprehensive |
| **Zeeg** | Google Calendar / Calendly | Germany | No MCP yet; GDPR-native scheduling |

**Decision**: Don't swap Google Workspace in the default connector set. Add Nextcloud and Proton as documented alternative paths in `smb-onboard` for owners who explicitly want EU-only infrastructure. Skills degrade to the same CSV/manual path when neither is connected.

### 7. Calendly — Add as new scheduling connector

Calendly has an **official MCP at `https://mcp.calendly.com`** with Dynamic Client Registration (no OAuth secrets needed in config). Not in the current plugin at all — adding it unlocks meeting scheduling in `call-list` and `monday-brief`. Calendly is US-based but offers EU data residency; Zeeg (German, GDPR-native) is the EU-sovereign alternative but has no MCP yet.

### 8. Pipedrive — Add as second CRM path

Pipedrive (HQ: Estonia, EU-native) is the most widely used CRM in northern and western Europe alongside HubSpot. Multiple community MCPs exist (most complete: `iamsamuelfraga/mcp-pipedrive` on GitHub). Add as an alternative connector in `lead-triage`, `crm-maintenance`, and `crm-cleanup` — same pattern as Brevo alongside HubSpot.

### 9. HubSpot — Keep

Widely used in EU, strong GDPR compliance, official MCP. Keep as primary CRM. EU owners now have two documented fallbacks: Brevo and Pipedrive.

### 10. PayPal — Keep

PayPal operates an EU legal entity (Luxembourg), supports SEPA, and the `paypal` MCP is already live. Keep as-is.

---

## New EU-Specific Services

### Brevo (formerly Sendinblue) — Optional CRM/Email Alternative

French company, GDPR-native, popular in FR/DE/ES as a HubSpot alternative for micro-SMBs. **Official MCP live at `https://mcp.brevo.com/v1/brevo/mcp`** — covers contacts, email campaigns, transactional email, lists, templates, automations, SMS campaigns, and CRM deals (55+ tools). Auth: Bearer token. Add as an optional connector from day one alongside HubSpot, not as a post-MVP item.

### SEPA / Banking

No single SEPA MCP exists today. The `cash-flow-snapshot` and `close-month` skills should mention SEPA batch file export as a manual step, with a note that future versions may connect to Open Banking APIs (PSD2) when an MCP becomes available.

---

## Connector Map (EU Edition)

```
.mcp.json changes:
  REMOVE:  stripe      → replace with mollie    (https://mcp.mollie.com/mcp)            ✓ live
  REMOVE:  square      → replace with sumup     (https://mcp.sumup.com/mcp)             ✓ live
  KEEP:    docusign    → active connector; note YouSign as preferred alternative in copy
  ADD:     exact       → use CData read-only now; upgrade to native MCP when available
  ADD:     brevo       → https://mcp.brevo.com/v1/brevo/mcp                             ✓ live
  ADD:     calendly    → https://mcp.calendly.com  (DCR, no OAuth secrets needed)       ✓ live
  ADD:     pipedrive   → community MCP (iamsamuelfraga/mcp-pipedrive)                   ✓ live
  OPTIONAL nextcloud   → community MCP (nextcloud-mcp-server on PyPI)  for EU-sovereign owners
  OPTIONAL proton      → community MCP via Bridge                       for EU-sovereign owners
  KEEP:    paypal, hubspot, canva, slack, ms365, gmail, google-calendar, google-drive
```

**Full MCP status:**

| Service | Status | URL / Source |
|---|---|---|
| Mollie | Official | `https://mcp.mollie.com/mcp` |
| SumUp | Official | `https://mcp.sumup.com/mcp` |
| Brevo | Official | `https://mcp.brevo.com/v1/brevo/mcp` |
| Calendly | Official | `https://mcp.calendly.com` |
| Pipedrive | Community | `iamsamuelfraga/mcp-pipedrive` (GitHub) |
| Nextcloud | Community | `nextcloud-mcp-server` (PyPI, 110+ tools) |
| Proton Mail | Community | via Proton Bridge IMAP/SMTP (PulseMCP) |
| Proton Drive | Community | `anyrxo/proton-drive-mcp` (GitHub) |
| Exact Online | Community (read-only) | `CDataSoftware/exact-online-mcp-server-by-cdata` |
| YouSign | Third-party only | viaSocket bridge; no official MCP yet |

**Still needs building:** Exact Online read-write MCP (blocks invoice issuance and reconciliation).

---

## New EU Skills

### `vat-return-prep` (new — replaces `tax-season-organizer` for EU)

US tax prep (1099s, quarterly estimates) is irrelevant in the EU. EU SMBs file periodic VAT returns (monthly or quarterly depending on country and turnover).

**What it does**: Pulls transaction data from Exact Online (or CSV), categorizes by VAT rate (0%, reduced, standard), separates intra-EU B2B (reverse charge) from domestic, outputs a filled-in summary matching the owner's country's VAT form format. Flags if the business likely qualifies for / is enrolled in OSS/IOSS (for cross-border e-commerce).

**Required**: Exact Online  
**Optional**: Mollie, PayPal  
**Replaces**: `tax-season-organizer` (which remains for non-EU fallback)

---

### `gdpr-audit` (new)

No equivalent in the current plugin. EU businesses are legally required to maintain a Record of Processing Activities (RoPA) and honor subject access / deletion requests within 30 days.

**What it does**: Interviews the owner about data flows (what customer data is stored, where, how long), cross-references what's visible in HubSpot and Gmail, produces a draft RoPA and a checklist of gaps. Flags any connected tool that appears to transfer data outside the EEA without an adequacy decision. Can draft a DSAR response email.

**Required**: None (works standalone via interview)  
**Optional**: HubSpot, Gmail  
**New command**: `/gdpr-audit`

---

### `einvoice-export` (new)

E-invoicing is now mandatory in Germany (ZUGFeRD/XRechnung as of 2025 for B2B), rolling out in France, Belgium, and Italy. The EU is mandating structured XML invoices by 2028 for all member states.

**What it does**: Takes an invoice from Exact Online or manual input, validates it against the country-specific e-invoice schema (UBL 2.1 / EN16931), and exports a compliant XML file ready for submission. Flags invoices that fail validation with plain-English fix instructions.

**Required**: None (works with manual input or Exact Online)  
**Optional**: Exact Online  
**New command**: `/einvoice-export`

---

## Skills Modified for EU Context

| Skill | Change |
|---|---|
| `cash-flow-snapshot` | QuickBooks → Exact Online as primary; SEPA-aware cash position; EUR default currency |
| `invoice-chase` | Stripe → Mollie for payment lookup; PayPal as fallback; SEPA reference numbers in reminder emails |
| `month-end-prep` | QuickBooks → Exact Online; Stripe → Mollie; add VAT reconciliation step |
| `tax-season-organizer` | Rename to `tax-prep` (EU) — redirects to `vat-return-prep` for EU owners; retains 1099 path for non-EU |
| `ticket-deflector` | Add Mollie refund path alongside PayPal refund path |
| `contract-review` | Keep DocuSign as active connector; add note that YouSign (eIDAS QES) is preferred if owner uses it |
| `smb-onboard` | EU-specific connector priority list; GDPR data notice before storing business context; mention VAT number field |
| `business-pulse` | EUR currency throughout; add VAT-due date to watch-list |
| `content-strategy` | Remove Square integration reference; add SumUp path |

---

## Plugin Metadata Changes

```json
{
  "name": "small-business-eu",
  "version": "0.1.0",
  "description": "EU-native small business workflows — accounting, VAT, GDPR, SEPA payments, and growth — using Exact Online, Mollie, PayPal, HubSpot, YouSign, and more. Every step that touches money or customers requires your approval.",
  "keywords": ["smb", "eu", "europe", "vat", "gdpr", "mollie", "exact", "finance", "operations"]
}
```

---

## Decisions

1. **Exact Online read-write**: Build a read-write MCP. CData read-only unblocks reporting; write actions (invoice issuance, reconciliation) need a native MCP. Exact's REST API is OAuth2, well-documented — build it.

2. **YouSign**: Keep DocuSign as the active connector. Update skill copy to prefer YouSign where the owner already uses it. Revisit when an official MCP ships.

3. **Country targeting**: **Pan-EU launch.** No country-first phasing — Mollie, SumUp, and Brevo are all pan-EU from day one.

4. **Tax skill scope**: **Implement v1 now** — NL/DE/BE (BTW/MwSt framework, same structure). FR and ES in v2.

5. **Brevo**: Add as a **second CRM/email path alongside HubSpot** — not a replacement. Skills accept either connector; degrade gracefully if neither is connected.

---

## What Stays the Same

The three-layer architecture (skills → commands → router), the approval-gate philosophy, and the skill file format are unchanged. The EU edition is a remapping of connectors and addition of three skills — not a rewrite.

---

## Next Steps

1. Confirm this PRD
2. Swap `.mcp.json`: add Mollie, SumUp, Brevo, Calendly (all official/live); add Exact CData (read-only); add Pipedrive community MCP; keep DocuSign; add Nextcloud/Proton as optional
3. Adapt existing skill files: QuickBooks → Exact Online, Stripe → Mollie, Square → SumUp; add Brevo + Pipedrive paths alongside HubSpot; add Calendly to `call-list` and `monday-brief`
4. Write `vat-return-prep` skill (NL/DE/BE scope for v1)
5. Write `gdpr-audit` skill
6. Write `einvoice-export` skill
7. Update `smb-onboard` connector priority list for EU; add VAT number field; add GDPR data notice; add EU sovereignty tier path (Nextcloud/Proton)
8. Update `plugin.json` metadata
9. Update `README.md`
10. (Post-MVP) Build read-write Exact Online MCP; add YouSign MCP when official; expand VAT skill to FR/ES