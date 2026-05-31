# Gotchas

## Gotcha: Skipping the prove-value step when a connection takes too long

**Why it matters:** If the owner connects a tool but Claude moves straight to the interview, the "aha" moment never lands. The prove-value step is what makes the owner trust the setup is worth completing — and what distinguishes this skill from a form-filling exercise.

### ✗ Bad

> "Great, Exact Online is connected! Now let me ask you a few questions about your business."

Skips the recipe entirely. Owner leaves not knowing what they just enabled.

### ✓ Good

> "Exact Online is live. Let me pull your last 30 days of cash flow — takes about 10 seconds."
> *[runs cash-flow-snapshot, shows results]*
> "That's what we can do anytime you want a number check. Now, a few questions about your business…"

The demo runs before the interview, every time, without exception.

---

## Gotcha: Asking country too late

**Why it matters:** Country determines which connectors to recommend (NL → Exact Online; DE → Exact Online; FR → Exact Online), which VAT regime applies, and which CRM the owner likely uses. Recommending the right connector stack requires knowing the country first.

### ✗ Bad

> "Let's connect Exact Online first…" (without first asking what country they're in)

Assumes the owner is NL-based without asking — may be wrong for a DE/FR/BE owner.

### ✓ Good

> "First — what country are you based in? That'll shape which tools and workflows make most sense."

Ask country before recommending any connector. EU owners go to Exact Online; adjust CRM recommendations by region.

---

## Gotcha: Dumping all interview questions at once

**Why it matters:** Five questions presented together feel like a form, not a conversation. Owners either skim-answer or drop off. Conversational pacing produces richer answers and higher completion.

### ✗ Bad

> "To get you set up, I need: (1) What country? (2) What kind of business? (3) How many employees? (4) What are your top headaches? (5) What tools do you use?"

### ✓ Good

Ask each question, receive the full answer, then ask the next. Compress to three (country, headaches, tools) if the owner signals they're in a hurry — never skip those three.

---

## Gotcha: Writing context without showing the profile first

**Why it matters:** Silent writes feel intrusive and remove the owner's chance to correct misheard answers. Showing the draft profile is also a natural confirmation that Claude understood correctly — it often surfaces a correction the owner wouldn't have volunteered.

### ✗ Bad

> "Got it, I've saved your business profile!"

No preview, no approval — owner doesn't know what was written.

### ✓ Good

> "Here's what I'm about to save as your business profile — let me know if anything needs fixing:"
>
> ```
> Country: Netherlands
> VAT number: NL123456789B01
> Business: Lakewood Hardware — retail hardware store
> Size: 4 people (including owner)
> Top headaches: cash flow gaps · slow-paying contractors · scheduling part-timers
> Connected tools: Exact Online, Mollie, Gmail
> Weekly cadence: weekly check-in every Monday
> Onboarded: 2026-04-23
> ```
>
> "Look right? I'll save it once you confirm."

Show the block, wait for approval, then write.

---

## Gotcha: Pitching the platform instead of the capability

**Why it matters:** Owners already know what HubSpot, Exact Online, Gmail, and Calendar do. When the recommendation reads like a product page, it lands as an ad, not advice.

### ✗ Bad

> "1. Exact Online (accounting) — One place for all your invoices, expenses, P&L, and VAT. Once it's in, I can pull your cash position instantly, prepare your BTW return…"

Reads like marketing copy. The owner is being sold to.

### ✓ Good

> "For cash flow visibility, the first thing I'd want is your accounting system. Are you on Exact Online, or something else?"
>
> *(Owner: "Exact Online.")*
>
> "Perfect — let's connect it. Here's how…"

States the function, checks what the owner uses, gives a clear next step. If the owner asks "what does Exact Online actually do?" — that's an explicit invitation; answer it directly.
