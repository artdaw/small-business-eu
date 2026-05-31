---
name: einvoice-export
version: 1.0.0
description: >
  Exports EU-compliant structured electronic invoices from Exact Online or
  manual input. Supports ZUGFeRD/XRechnung (Germany), UBL 2.1 / PEPPOL BIS
  (Netherlands, Belgium, and pan-EU). Validates against the EN16931 European
  e-invoicing standard, flags validation errors in plain English, and produces
  a downloadable XML file ready for submission to the recipient or a government
  portal. Mandatory in Germany for B2B since 2025; mandatory across the EU
  for large companies, rolling out to SMBs by 2028.

  Trigger when the user says: "e-invoice," "electronic invoice," "ZUGFeRD,"
  "XRechnung," "UBL invoice," "PEPPOL," "structured invoice," "EN16931,"
  or "my customer needs an XML invoice."
---

# E-Invoice Export

> **Scope:** Germany (ZUGFeRD 2.x / XRechnung 3.x), Netherlands (PEPPOL BIS 3.0 / UBL 2.1), Belgium (PEPPOL BIS 3.0 / UBL 2.1). Other EU member states: general UBL 2.1 output — check local requirements with your accountant.

## Quick start

```
User: "export invoice #2024-042 as an e-invoice"
→ Pull invoice from Exact Online (or ask for manual input)
→ Detect country and select the right schema
→ Validate all required fields
→ Flag any missing or invalid data with plain-English fix instructions
→ Generate the XML file
→ Offer to save to Google Drive or download
```

## Step 1 — Identify the invoice

**From Exact Online (preferred):** Ask the user for the invoice number or let them describe it (customer name, approximate date/amount). Pull the invoice record including:
- Seller: company name, address, VAT number, IBAN
- Buyer: company name, address, VAT number (required for B2B e-invoices)
- Line items: description, quantity, unit price excl. VAT, VAT rate, VAT amount, line total
- Invoice header: invoice number, issue date, due date, currency (EUR), payment reference

**Manual input (fallback):** If Exact Online is not connected or the invoice isn't found, collect the fields above via a structured interview. Present a checklist and ask for missing fields one at a time.

## Step 2 — Detect country and select schema

Read `## Business context` for country. Apply:

| Country | Default format | Notes |
|---|---|---|
| Germany (DE) | XRechnung 3.x | Mandatory for B2G since 2017; B2B since 2025 |
| Netherlands (NL) | PEPPOL BIS 3.0 / UBL 2.1 | Government portals accept PEPPOL; many large buyers require it |
| Belgium (BE) | PEPPOL BIS 3.0 / UBL 2.1 | Mandatory for B2G; B2B rollout underway |
| Other EU | UBL 2.1 (EN16931) | Generic; check recipient requirements |

If the customer's country differs from the owner's (cross-border invoice), use the buyer's country requirements if they are more specific. Ask if unclear.

For Germany, confirm whether XRechnung or ZUGFeRD is required:
- **XRechnung**: pure XML, required for German federal and state government buyers
- **ZUGFeRD 2.x (Comfort/Extended)**: hybrid PDF+XML, accepted for most B2B; also satisfies XRechnung requirements at the Comfort+ profile

Ask: "Does your customer or their system specify XRechnung or ZUGFeRD, or is either acceptable?"

## Step 3 — Validate required fields

Run through the EN16931 mandatory field checklist:

**Seller:**
- [ ] Full legal name
- [ ] Full registered address (street, postcode, city, country code)
- [ ] VAT registration number (format: NL999999999B01 / DE999999999 / BE0999999999)
- [ ] IBAN (for payment)

**Buyer:**
- [ ] Full legal name
- [ ] Full registered address
- [ ] VAT number (mandatory for reverse-charge invoices; strongly recommended for all B2B)

**Invoice header:**
- [ ] Unique invoice number
- [ ] Issue date (ISO 8601: YYYY-MM-DD)
- [ ] Due date
- [ ] Currency code (EUR)
- [ ] Payment terms or payment reference

**Line items (each line):**
- [ ] Item description
- [ ] Quantity and unit of measure
- [ ] Unit price excl. VAT (EUR)
- [ ] VAT rate (%) and VAT category code (S = standard, Z = zero, E = exempt, K = intra-EU reverse charge)
- [ ] Line total excl. VAT

**Invoice totals:**
- [ ] Sum of line totals excl. VAT
- [ ] Total VAT amount
- [ ] Invoice total incl. VAT

For each missing or invalid field, surface a plain-English fix instruction:
> "⚠ Buyer VAT number is missing. For B2B e-invoices, this is required to apply reverse charge or for the buyer to recover input tax. Add it from your CRM or ask the customer for it."

Do not proceed to XML generation until all mandatory fields are present or the owner explicitly overrides a non-mandatory field.

## Step 4 — Generate the XML

Produce the invoice in the selected format:

**XRechnung (Germany):** Generate UBL 2.1 XML conforming to the XRechnung 3.x specification. Include the `CustomizationID` element:
```
urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0
```

**ZUGFeRD Comfort (Germany):** Generate the CrossIndustryInvoice XML conforming to ZUGFeRD 2.1 Comfort profile (UN/CEFACT CII D22B syntax). Note: embedding in a PDF requires a PDF/A-3 compliant viewer — Claude produces the XML; PDF embedding is outside scope.

**PEPPOL BIS 3.0 (NL/BE/pan-EU):** Generate UBL 2.1 XML conforming to PEPPOL BIS Invoice 3.0. Include:
```
CustomizationID: urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0
ProfileID: urn:fdc:peppol.eu:2017:poacc:billing:01:1.0
```

**Output filename:** `invoice-[InvoiceNumber]-[YYYY-MM-DD]-[format].xml`
Example: `invoice-2024-042-2024-03-15-xrechnung.xml`

## Step 5 — Offer delivery options

After generating the XML:

1. **Download:** "The XML file is ready. Want me to save it to Google Drive / OneDrive, or download it locally?"
2. **Email attachment:** "Want me to attach this to a draft email to [buyer name]?" — draft only; owner sends.
3. **Peppol network:** "If your customer is reachable via the PEPPOL network, you can submit directly through your accounting software or a PEPPOL access point. I can't submit directly, but your Exact Online instance may support PEPPOL send."

## Approval gates

- **Never send the invoice file** without explicit owner approval.
- **Never modify Exact Online invoice records.** Read-only access only.
- **Always show the field validation summary** before generating XML. The owner must acknowledge any missing fields.

## Reference files

- [reference/field-specs.md](reference/field-specs.md) — full EN16931 field list with data types, formats, and EU country-specific notes
- [reference/xrechnung-schema.md](reference/xrechnung-schema.md) — XRechnung 3.x structure and mandatory elements
- [reference/peppol-bis.md](reference/peppol-bis.md) — PEPPOL BIS 3.0 structure for NL/BE
- [reference/vat-categories.md](reference/vat-categories.md) — VAT category codes (S, Z, E, K, AE, O)
- [reference/gotchas.md](reference/gotchas.md) — common errors: wrong VAT number format, missing buyer address, incorrect VAT category for reverse charge
