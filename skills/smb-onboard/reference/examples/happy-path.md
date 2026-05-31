# Happy path: Retail shop owner, first session

## Setup

- **Owner:** Maria, sole operator of a hardware store with 3 part-time employees, based in the Netherlands
- **Session type:** First session ("get me started")
- **Connectors before start:** None connected

---

## Transcript

**Claude:** Welcome! I'm going to walk you through getting set up — takes about 15 minutes and ends with me knowing enough about your business to be immediately useful. Ready?

**Maria:** Sure.

**Claude:** First — what country are you based in? That'll shape which tools and workflows make most sense.

**Maria:** Netherlands.

**Claude:** Perfect. We'll start with Exact Online — that's the standard accounting system here and lets me pull your cash position anytime. Here's how to authorize the connection…

*[Maria sets up Exact Online via CData MCP — ~5 minutes]*

**Claude:** Exact Online is live. Let me pull your last 30 days of cash flow right now.

*[runs cash-flow-snapshot]*

**Claude:** Here's your snapshot: €4,200 net cash in for the month, but €6,100 in outstanding invoices, mostly from two contractor accounts. Your biggest gap looks like Net-30 customers paying Net-45. Does that match what you're seeing?

**Maria:** Yes, exactly. I didn't know I could see that so fast.

**Claude:** That's available anytime. Second tool: let's connect Mollie for payment data, and Gmail for cross-referencing invoices with emails.

*[Maria connects Mollie and Gmail — ~3 minutes each]*

**Claude:** Good. A few questions so I know your business — one at a time. What's the one-liner for what you do?

**Maria:** Hardware store. Retail and light contractor supply.

**Claude:** Do you have a VAT number registered in the Netherlands?

**Maria:** Yes, NL123456789B01.

**Claude:** How many people work with you, including yourself?

**Maria:** Four total — me and three part-timers.

**Claude:** What are your top three headaches right now?

**Maria:** Cash flow, slow-paying contractors, and scheduling the part-timers around school.

**Claude:** Which tools do you already use day-to-day?

**Maria:** Exact Online and Gmail, which we just connected. Also SumUp for in-store payments.

**Claude:** Got it. Last one: how often would you like me to check in — daily, weekly, or only when you ask?

**Maria:** Weekly's good. Monday mornings.

**Claude:** Before I save this, here's what the data notice covers: your business info and the transaction data I pull from Exact Online, Mollie, and Gmail are used only to answer your questions — not stored externally or used to train AI models.

Here's what I'm about to save as your business profile — let me know if anything's off:

```
Country: Netherlands
VAT number: NL123456789B01
Business: Hardware store — retail and light contractor supply
Size: 4 people (including owner)
Top headaches: cash flow gaps · slow-paying contractors · scheduling part-timers
Connected tools: Exact Online, Mollie, Gmail, SumUp
Weekly cadence: weekly check-in every Monday
Onboarded: 2026-04-23
```

Look right?

**Maria:** Yep, that's it.

**Claude:** Saved. Every skill from here will know your business. Each Monday, just say "weekly check-in" and I'll pull your cash snapshot, flag urgent invoices, and remind you what's due that week. Want to try the invoice reminder skill right now? Say "invoice chase."

---

## Pass criteria

- [ ] First question is country, before any connector recommendation
- [ ] Asked headache question before pitching additional connectors
- [ ] Recommended Exact Online first (NL-based owner + cash-flow headache)
- [ ] Ran `cash-flow-snapshot` immediately after Exact Online connected, before interview
- [ ] Asked for VAT number during interview
- [ ] Connected additional tools (Mollie, Gmail) before starting interview
- [ ] Asked all interview questions one at a time
- [ ] Showed GDPR data notice before saving profile
- [ ] Showed profile draft before writing
- [ ] Wrote context under `## Business context` in `~/.claude/CLAUDE.md`
- [ ] Named the weekly cadence trigger phrase explicitly
- [ ] Offered a next skill naturally at the end

---

# Branch B: Owner uses an unsupported CRM

## Setup

- **Owner:** Dev, freelance design studio (2 people), based in Germany
- **Session type:** First session
- **Primary headache:** Customer follow-up
- **Current CRM:** Salesforce (not in the EU-native stack)

## Transcript

**Claude:** What country are you based in?

**Dev:** Germany.

**Claude:** Great. What are your biggest day-to-day headaches — money stuff, keeping up with customers, scheduling, or something else?

**Dev:** Customer follow-up. Leads slip through.

**Claude:** For customer follow-up, the two pieces I'd want are a CRM and your inbox. Are you on HubSpot, Pipedrive, or Brevo today, or something else?

**Dev:** Salesforce.

**Claude:** Got it — there's no Salesforce connector in the current setup. If you stayed on Salesforce, you'd still get cash-flow work and calendar work from Claude, but I wouldn't be able to score leads or draft follow-ups from inside the CRM. Pipedrive (Estonian, EU-native) and HubSpot both have connectors — if you're open to trying one, here's what'd unlock: a top-5 call list every morning, drafted follow-ups after every meeting, and stale-deal alerts. Up to you — want to try it, or skip the CRM for now?

**Dev:** Skip for now. I'll look at Pipedrive later.

**Claude:** No problem. Then let's connect Gmail — that'll still let me surface customer threads waiting on a reply and draft replies for you. Sound good?

**Dev:** Yeah.

*[Dev connects Gmail — ~2 minutes]*

**Claude:** Gmail is live. Let me pull the three oldest unread customer threads…

## Pass criteria

- [ ] First question is country
- [ ] Named the function ("a CRM and your inbox"), not the platform
- [ ] Asked what the owner uses before pitching anything
- [ ] Listed EU-native CRM options first (Pipedrive, HubSpot, Brevo)
- [ ] On unsupported tool, listed gain/loss concretely without pushing
- [ ] Accepted the owner's decision and pivoted to the next supported tool
- [ ] No paragraph-long descriptions of the tools
