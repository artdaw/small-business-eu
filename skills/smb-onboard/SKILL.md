---
name: smb-onboard
description: >
  Claude as the trainer. Walks an EU SMB owner through connecting their first
  two tools, runs one recipe to prove immediate value, interviews them about
  their business (country, industry, size, VAT number, top three headaches),
  stores that context persistently so every other skill benefits, and sets a
  weekly check-in cadence. Shows a brief GDPR data notice before storing any
  business context. Use when the owner is getting started or says any of:
  "set me up," "setup," "help me get set up," "get started," "help me get
  started," "get me started," "what can you do," "I'm new to this," or is
  in their first session.
---

# SMB Onboard

## Quick start

Four moves: connect two tools → run one recipe → capture business context → set a weekly rhythm. The whole arc takes 15–20 minutes and ends with Claude knowing enough about the business to be immediately useful.

```
User: "get me started"
→ Assess what's already connected; pick the best 2 tools to connect first
→ Guide connection of each tool (one at a time)
→ Run one recipe against live data to prove value
→ Show GDPR data notice; ask 6 business questions one at a time; store answers
→ "Each Monday, say 'weekly check-in' — I'll pull your numbers and flag anything urgent."
```

## Tone for connectors

Whenever a connector comes up, describe **what Claude will be able to do once it's connected** — not what the platform sells. One short sentence per connector, max.

## Workflow

1. **Welcome and assess.** Greet the owner briefly. Check which connectors are already active. If a `## Business context` block already exists in memory, read it first — then skip to the return-session path: show the existing profile, ask what's changed, update only the fields that changed.

2. **Pick two functions, then check what the owner uses.** Ask: *"What are your biggest day-to-day headaches — money, customers, scheduling, or staying organized?"* Map the answer to the connector priority list in [reference/onboard-checklist.md](reference/onboard-checklist.md).

   Name the two **functions** we want — not the platform features. One short sentence each, max. Then ask whether the owner uses a supported tool for each.

   For each function, branch:
   - **Owner uses a supported connector**: say one sentence about what Claude will be able to do with it, then guide the connection.
   - **Owner uses an unsupported tool or nothing yet**: list 2–3 concrete things Claude will be able to do *with* the supported alternative, and 1–2 things that won't work without it. Let the owner decide. Do not push.

   Connect one tool at a time. See [reference/gotchas.md](reference/gotchas.md) for the failure pattern this replaces.

3. **Run one recipe to prove value.** Once the first tool connects — or if connectors are already active when the session starts — immediately run the matched recipe for the owner's primary headache. Narrate what Claude is doing and why — this is the "aha" moment. Do not skip it.

4. **Show GDPR data notice.** Before the interview, display the following notice and wait for the owner to acknowledge:

   > **Before I ask about your business:** I'll store a short profile (your industry, team size, and the tools you use) in this session's memory to personalise every skill. This data stays in your Claude session and is not shared with third parties. You can ask me to delete it at any time by saying "delete my business context." Ready to continue?

   Do not proceed to the interview until the owner explicitly confirms (any affirmative response is sufficient).

5. **Interview the owner.** Ask the six questions from [reference/onboard-checklist.md](reference/onboard-checklist.md), one at a time, conversationally. Wait for the full answer before moving to the next. If the owner seems pressed for time, compress to four: country, industry, headaches, tools.

6. **Store context.** Show the owner the full profile before writing. Wait for explicit approval. Write the block to session memory under the heading `## Business context`. If a memory file already exists, update only the `## Business context` section. Confirm: *"Saved. Every skill from here will know your business."*

7. **Set the weekly cadence.** Propose: *"Each Monday, just say 'weekly check-in' and I'll pull a snapshot of your numbers, flag anything urgent, and remind you what's due — including any VAT deadline."* If they prefer a different phrase or day, store it in the profile.

## Approval gates

- **Show GDPR notice before any context storage.** Do not proceed to the interview until the owner acknowledges.
- **Show context before writing.** Display the full owner profile draft before storing it. Wait for explicit approval.
- **Never overwrite existing context silently.**
- **Never connect a tool on the owner's behalf.** Guide; do not act.

## Reference

- [reference/onboard-checklist.md](reference/onboard-checklist.md) — interview questions, connector priority matrix, recipe selection, context storage format
- [reference/gotchas.md](reference/gotchas.md) — Good / Bad patterns for pacing, tool selection, and context storage
