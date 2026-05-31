---
name: content-strategy
description: >
  Analyzes sales data from Mollie, PayPal, and Exact Online to find top
  performers and slow movers, layers in seasonality, and produces a prioritized
  30-day content brief: what to push, what offers to run, what to hold.
  Strategic output only — no calendars or assets. Use when the user asks what
  to post, wants a content plan, asks what's selling, or what to promote this month.
---

# Content Strategy

> **Version:** 0.3.0 · **Category:** Marketing & Sales

## Quick start

When an SMB owner asks "what should I post this month?" or "what's my content plan?", this skill:

1. **Pulls sales data** from Exact Online or Mollie/PayPal (transaction history, product/service revenue by date)
2. **Identifies patterns** — top-selling products, slow movers, seasonal trends
3. **Layers in context** — seasonality (user-provided or industry benchmarks), past performance
4. **Produces a 30-day brief** — ranked recommendations of what to push, what to hold, what offers to consider
5. **Gets owner approval** before the brief feeds into `canva-creator` for asset generation

The output is strategic only — no calendar scheduling, no creative assets.

---

## Workflow

### Step 1: Pre-flight check (Exact Online)

If using Exact Online, verify the business profile is set up:

1. Check if industry/sector is populated in the owner's business context.
2. If missing: ask "I need your business category to pull the right seasonality benchmarks. What industry are you in?" Store the answer in business context.
3. If set, proceed to Step 2.

**Note:** Mollie and SumUp do not require profile setup.

### Step 2: Clarify priorities & metrics

Ask the user:

- **"How do you want me to measure 'top performers'?"**
  - By total revenue?
  - By profit margin?
  - By sales velocity?
  - Combination?

- **"Do you have seasonality patterns in mind?"**
  - If yes: capture them
  - If no: "I'll use industry benchmarks for your category"

### Step 3: Pull and analyze sales data

Fetch data from the authenticated connector (Exact Online, Mollie, or PayPal):

- **Date range:** Last 90 days (or full history if <90 days available)
- **Extract:** Product/service name, date sold, revenue, quantity

**Connector-specific notes:**

- **Exact Online:** Fetch invoice line items via P&L and sales invoice API. Industry context from business profile drives seasonality benchmarks.
- **Mollie:** Fetch payments via the Mollie MCP. Group by product description or metadata tags where available.
- **PayPal:** Fetch merchant transactions via PayPal MCP. *Rate-limiting:* If you hit rate limits, pause 30 seconds and retry once. If still blocked: "PayPal is rate-limited. Want to switch to Exact Online or Mollie instead, or I can continue with historical data I already pulled?"
- **SumUp:** Fetch POS transactions and group by product/item sold.

**Fallback:** If <3 months of data, use industry seasonality benchmarks for the owner's category (e.g., retail, services, e-commerce).

Identify:
- **Top 3–5 performers** (by user's chosen metric)
- **Bottom 3–5 slow movers** (consider holding or repositioning)
- **Trending up** (gaining momentum in last 30 days)
- **Trending down** (losing momentum)

### Step 4: Layer in seasonality

- **User-provided:** Weight recommendations against stated seasonal patterns
- **Industry benchmarks:** For categories without strong user data
- **Timing:** Flag products that should ramp up/down in the next 30 days

### Step 5: Build the 30-day brief

Structure:
- **Executive summary** (1–2 sentences)
- **Push hard** (Top 2–3 products/services + recommended content angle)
- **Hold steady** (Middle performers; maintain visibility)
- **Reposition or pause** (Slow movers; consider discounting, bundling, or pausing)
- **Seasonal opportunities** (What's coming next month to position for now)
- **Recommended offers** (Bundle, discount, or free-trial strategy based on data)

Example length: **200–400 words** (brief and actionable).

### Step 6: Owner approval & iteration

Present the brief to the owner. Ask:
- "Does this match your gut?"
- "Anything to adjust?"
- "Ready to feed this to canva-creator for asset generation?"

Iterate if needed; once approved, return the final brief as structured JSON (ready for downstream tools).

---

## Gotchas & edge cases

See [`reference/gotchas.md`](reference/gotchas.md) for common pitfalls.

## Examples

See [`reference/examples/`](reference/examples/) for worked examples (SaaS, retail, services).
