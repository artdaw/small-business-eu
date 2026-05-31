---
name: margin-analyzer
description: >
  Analyzes unit economics by product or service using Mollie or PayPal
  payment data and Exact Online cost data, benchmarks against inflation and
  cost changes, and shows pricing-scenario data. Surfaces analysis only —
  does not recommend a price. Use when the user asks about raising prices,
  pricing, margin analysis, what to charge, whether costs are eating into
  profit, or how a price change might affect their business.
---

# Margin Analyzer

> **Version:** 1.2.0 · **Category:** Finance & Ops

## Quick start

When an SMB owner asks "should I raise my prices?" or "are my margins okay?", this skill:

1. **Identifies what to analyze** — which products/services are in scope
2. **Pulls cost data** from Exact Online (COGS, direct expenses)
3. **Pulls revenue data** from Mollie or PayPal (transaction history)
4. **Computes unit economics** — revenue, COGS, gross margin, margin % per item — all in EUR
5. **Benchmarks against context** — inflation, cost changes, industry norms
6. **Builds pricing scenarios** — what happens to revenue and margin at +5%, +10%, +15% price changes
7. **Presents the analysis** — no price recommendation; the owner decides

---

## Workflow

### Step 1: Pre-flight check

**Exact Online:** Verify the industry/sector field is populated in business context. If missing, ask: "I need your business category to pull relevant benchmarks. What industry are you in?" Store the answer.

**No connectors:** Offer CSV upload as a fallback. Expected CSV schema: product/service name, date, revenue, COGS. See `reference/csv-schema.md`.

### Step 2: Clarify scope

Ask the owner two questions:

1. **"Which products or services do you want to analyze?"**

2. **"What metric matters most to you?"**
   - Gross margin (revenue minus direct costs)?
   - Net margin (after all expenses)?
   - Revenue per unit?

### Step 3: Pull cost data (Exact Online)

Fetch from Exact Online:
- **Date range:** Last 12 months (or full history if less is available)
- **Extract:** Cost of goods sold by product/service line, direct expenses — all in EUR

If Exact Online isn't connected, ask the owner for a cost breakdown by product/service line.

If Exact Online is connected but COGS = €0 across all periods, surface this to the owner:

> "Exact Online shows no cost of goods sold recorded for this period. To compute meaningful margins, I need a cost breakdown by product or service line — not a single average for the whole business. For each item you want analyzed, what does it cost you to deliver it?"

Flag this limitation in the Data Quality Notes section of the final output.

### Step 4: Pull revenue data (Mollie / PayPal / SumUp)

Fetch transaction history:
- **Date range:** Match the cost data window (last 12 months)
- **Extract:** Transaction amount (EUR), item/service name, date, quantity if available

Try in order: Mollie → PayPal → SumUp. If you hit PayPal rate limits, pause 30 seconds and retry once. If still blocked: "PayPal is temporarily rate-limited. Want to switch to Mollie or SumUp, or upload a CSV instead?"

If only one data source is available, note the limitation in the output.

### Step 5: Compute unit economics

For each product/service in scope, calculate:

| Metric | Formula |
|---|---|
| **Revenue** | Sum of transaction amounts in EUR |
| **COGS** | Cost data from Exact Online or owner-provided |
| **Gross Profit** | Revenue − COGS |
| **Gross Margin %** | (Gross Profit ÷ Revenue) × 100 |
| **Units Sold** | Count of transactions (if available) |
| **Revenue per Unit** | Revenue ÷ Units Sold |
| **Cost per Unit** | COGS ÷ Units Sold |

Flag any item where margin is below 20% — as a data point, not a recommendation.

### Step 6: Benchmark

- **Inflation:** Note relevant cost trends. "Your input costs rose ~X% over this period while your prices held flat — that compressed margin by Y points."
- **Industry benchmarks:** Use the owner's industry from business context. See `reference/industry-benchmarks.md`.
- **Historical comparison:** If 24+ months of data is available, compare this year's margins to last year's.

If fewer than 6 months of transactions exist, omit the elasticity section and note: "You need at least 6 months of pricing history to estimate how volume responds to price changes. I'll show scenario math instead."

### Step 7: Pricing scenarios

Build a table for each product/service showing three price-change scenarios:

| Scenario | New Price | Projected Revenue* | Gross Margin % |
|---|---|---|---|
| +5% | €X | €Y | Z% |
| +10% | €X | €Y | Z% |
| +15% | €X | €Y | Z% |

**If 6+ months of history:** Estimate volume response using historical data. Compute observed elasticity if a past price change exists.
**If insufficient history:** Show three volume assumptions (−0%, −5%, −10%) and let the owner pick what seems realistic.

Add a note: *"These are projections based on available data, not guarantees."*

### Step 8: Present the analysis

Structure: H2 header (business name + date range), Unit Economics Summary table, Context and Benchmarking (2–4 sentences), Pricing Scenarios (table per product), Data Quality Notes.

Keep it factual. Do not say "you should raise prices." The owner decides.

---

## Scope boundary

**This skill surfaces data. It does not recommend a price.**

If the owner asks "so what should I do?" — respond: "I can show you what the data suggests, but the pricing decision is yours. Would you like me to model any additional scenarios?"

---

## Connectors

**Primary:** Exact Online, Mollie  
**Also supported:** PayPal, SumUp · CSV upload

## Reference files

- `reference/gotchas.md` — common pitfalls (data gaps, elasticity traps, margin math errors)
- `reference/industry-benchmarks.md` — gross margin ranges by SMB category
- `reference/csv-schema.md` — expected columns when the owner uploads a CSV
