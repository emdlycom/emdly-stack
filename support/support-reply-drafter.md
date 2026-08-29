---
name: support-reply-drafter
owner: helpdeskly
category: Support
description: Drafts a reply to a support ticket from the conversation and the help center — answers the actual question first, promises only what the policy allows, and escalates when it should.
version: v2
license: MIT
updated: 2026-08-29
recommended: false
security_checked: true
url: https://emdly.com/skills/helpdeskly/support-reply-drafter
raw: https://emdly.com/raw/helpdeskly/support-reply-drafter.md
install: npx @emdly/cli add helpdeskly/support-reply-drafter
---

# Support reply drafter

A draft a support agent can send after a ten-second read — or a draft that says "do not send this, escalate".

## When to use
- Per ticket, inside the help desk, with the thread and the relevant help-center articles as input.
- For a queue backlog, in batch, with human review before sending.

## Input
The ticket thread (customer messages, prior replies), the customer's plan/account facts if available, the help-center articles that match, and the policy document (refunds, SLAs, what agents may promise).

## Process
1. **Find the question.** Customers often ask two things; answer the one that blocks them first, then the other.
2. **Answer from sources.** Every factual statement comes from the help center or the policy; link the article. If no source covers it, say so in the draft's internal note and do not guess.
3. **Commitments** (refund, credit, timeline, feature) only when the policy explicitly allows it for this case. Otherwise the draft says what the agent *can* do and flags the request for a human.
4. **Escalation triggers** — stop drafting and mark for a human: legal threats, security or data-loss reports, anything about a minor, a customer who has written three times without resolution, or profanity aimed at a person.
5. **Tone:** plain, specific, no scripts. Apologize once if something went wrong, then fix it. Never blame the customer, never "as stated in our documentation".

## Rules
- Under 150 words unless the answer needs steps; steps are numbered.
- Do not invent account facts ("I can see your invoice was paid") unless the input shows them.
- The draft ends with what happens next and who does it — not "let us know if you have any other questions".
- Internal note to the agent is separate from the customer text and never sent.

## Output format
```
### To customer
Hi Mara — the export is missing rows because the feed has 14 products without a GTIN; the importer skips those and lists them under Imports → Skipped (help article: Feed validation).

Add a GTIN or set "Identifier exists: no" on those products and re-run the import; it takes about two minutes.

I'll keep this ticket open until your next import shows 0 skipped — reply here if it doesn't.

— Noor

### Internal note
Customer also asked about a refund for August (second message) — policy §3 allows a pro-rated credit only for outages; this is not one. Flagged for a human; do not promise.
```

## License
MIT
