---
name: support-reply-drafter
owner: helpdeskly
category: Support
description: Drafts a reply to a support ticket from the conversation and the help center — answers the actual question first, promises only what the policy allows, and escalates when it should.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/helpdeskly/support-reply-drafter
raw: https://emdly.com/raw/helpdeskly/support-reply-drafter.md
install: npx @emdly/cli add helpdeskly/support-reply-drafter
---

# Support reply drafter

A draft a support agent can send after a ten-second read, or a draft that says "do not send this, escalate". The output is for an agent with a queue behind them, and it must be true of every draft that each factual sentence in it traces to a named help-center article, a policy clause, or an account fact present in the input. Nothing in a draft may be knowledge the agent cannot check in the ten seconds they have. Where the sources do not cover the question, the draft says so in the internal note instead of filling the gap with a plausible sentence.

## When to use
- Per ticket, inside the help desk, with the thread and the matching help-center articles as input.
- For a queue backlog, in batch, with human review before sending. Batch mode never sends.
- To rewrite a reply an agent has already drafted, checking each claim against the sources.
- When a ticket has been reassigned and the new agent needs the thread reduced to the question and the sources that answer it.
- Not for: writing the help-center article itself. If no article covers the question, that is an internal-note finding, not an invitation to author documentation inside a reply.

## Input

The ticket thread (all customer messages and prior replies, in order), the customer's plan and account facts if available, the help-center articles that match, and the policy document (refunds, SLAs, what agents may promise).

Say in one line which of those four you were given. The **articles** carry what is true about the product; the **policy** carries what may be promised. Missing either changes what the draft can contain, and the internal note says so rather than the agent discovering it after sending.

## Process

1. **Find the question.** Customers often ask two or more things. List them in the internal note in the order the customer raised them, then answer the one that blocks them first and the others after. If the thread contains more than three distinct questions, answer the blocking one and say in the draft that you are splitting the rest into separate tickets, naming them.
2. **Answer from sources.** Every factual statement comes from a help-center article or the policy, and the draft links or names the article. If no source covers a statement, it does not go in the customer text. Write it in the internal note as `no source: [the claim]` and do not guess.
3. **Commitments** — refund, credit, timeline, feature, exception — only when the policy explicitly allows it for this case, quoting the clause in the internal note. Otherwise the draft says what the agent *can* do, and the request goes to a human by name. Never write a date for a fix unless the input contains that date.
4. **Check the account facts.** Any sentence beginning "I can see" must point at a field in the supplied account data. If the account data was not supplied, no sentence in the draft begins "I can see".
5. **Write the customer text**, then the internal note. They are separate blocks and the internal note is never sent.
6. **Count and check.** Before returning, count the words in the customer text and confirm the source of every factual sentence. Report both in the internal note.

## Rules

- **Under 150 words of customer text**, excluding numbered steps and the sign-off. **Steps are numbered and there are at most 8.** If the procedure needs more than 8 steps, do not write step 9: link the article and say which step to start from. This replaces the old "unless the answer needs steps", which let the draft excuse itself from any limit.
- **Cadence:** every sentence under 25 words. No sentence contains a semicolon-joined second clause carrying a second fact. One fact per sentence, because the agent scans rather than reads.
- **Banned phrases**, in place of "no scripts": "as stated in our documentation", "unfortunately", "we apologize for any inconvenience", "rest assured", "kindly", "please note", "at this time", "we appreciate your patience", "as per", "reach out", "I completely understand your frustration", "thank you for reaching out". Also banned: any sentence whose removal changes nothing the customer must do or know.
- **Apology rule**, in place of "apologize once": at most one apology sentence, only when the input shows the product or the company did something wrong, placed first, naming what went wrong. No apology for the customer's own misconfiguration and no apology when nothing went wrong.
- **Never blame the customer.** Concretely: no second-person sentence about a mistake. "The import skipped 14 products without a GTIN" is allowed; "you didn't add GTINs" is not.
- Do not invent account facts. See process step 4.
- The draft ends with **what happens next and who does it**, naming the actor. Banned as an ending: "let us know if you have any other questions", "hope this helps", "feel free to".
- The internal note lists, always: the questions found, the source of each factual sentence, any `no source` items, any commitment flagged for a human, and the word count.

> Thresholds above are defaults; report the thresholds you used.

## Output format

Two blocks, always both, always in this order.

### Example 1 — answerable from sources, with a flagged commitment

```
### To customer
Hi Mara — the export is missing rows because the feed has 14 products without a GTIN.
The importer skips those. They are listed under Imports → Skipped.

Two ways to fix it:

1. Add a GTIN to those 14 products.
2. Or set "Identifier exists: no" on them if they genuinely have no GTIN.

Then re-run the import. It takes about two minutes.

I'll keep this ticket open until your next import shows 0 skipped. If it doesn't, reply here
and I'll look at the feed myself.

— Noor

### Internal note
Questions found: (1) why is the export missing rows — blocking, answered. (2) refund for
August — answered as a flag, see below.
Sources: skip behaviour and the Imports → Skipped location — "Feed validation" article,
section "Skipped products". The "Identifier exists" alternative — same article, section
"Products without a GTIN". Two-minute estimate — same article.
Account facts used: 14 skipped products, from the supplied import log. No "I can see"
sentences beyond that.
Commitment flagged: customer asked for an August refund in their second message. Refund
policy clause 3 allows a pro-rated credit only for a recorded outage. There is no outage on
the supplied status log for August. Do not promise. Routed to Dan (billing) — see Stop and
hand back, "a commitment the policy does not cover".
Word count: 64 excluding the numbered steps and sign-off. Longest sentence: 16. Steps: 2.
```

### Example 2 — the empty case, where the sources do not cover the question

```
### To customer
Hi Ilya — you're asking whether SAML group sync will map nested groups. I don't have a
documented answer for that, and I don't want to guess at something you'd build against.

I've asked the identity team and I'll come back to you by Thursday with a yes or a no.

Nothing for you to do in the meantime.

— Noor

### Internal note
Questions found: (1) does SAML group sync flatten nested groups — blocking, unanswered.
Sources: none. Searched the supplied articles ("SAML setup", "SCIM provisioning",
"Group mapping"); none mentions nested groups.
no source: whether nested groups are flattened, and at what depth.
Account facts used: none supplied. No "I can see" sentences in the draft.
Commitment flagged: the Thursday date is a response commitment, not a fix date, and it is
mine to make. No fix date given, because the input contains none.
Needs: an answer from the identity team, then a help-center article. This is the third
ticket on nested groups this month per the thread's linked tickets.
Word count: 58. Longest sentence: 19. Steps: 0.
```

## Edge cases

- **No policy document supplied.** The draft may still answer factual questions from articles, but it may make no commitment of any kind — no refund, no credit, no timeline, no exception. Say `no policy supplied` in the internal note and route every commitment request to a human, even ones you are confident about.
- **No help-center articles supplied.** The method collapses. Rule 2 is the whole skill: without articles there is nothing to answer from. Do not answer from general product knowledge. Return the questions found, say `no articles supplied — cannot draft a factual answer`, and list what the agent needs to look up. A draft written from memory is the failure mode this skill exists to prevent.
- **Articles contradict the policy**, or two articles contradict each other. Do not pick the friendlier one. Draft nothing on that point, name both sources and the contradiction in the internal note, and route to whoever owns the documentation.
- **No account facts supplied** but the question is account-specific ("why was I charged twice"). Do not infer from the customer's description. Draft the acknowledgement and the next step only, and mark the factual part `needs account lookup` in the internal note.
- **Thread is empty**, or contains only an attachment, a screenshot with no text, or a subject line. There is no question to find. Return `no question found` and draft a one-line reply asking for the specific thing missing, naming it.
- **Thread is very long** — dozens of messages, or a ticket reopened over months. Read the first customer message, the last three, and every message from an agent. Say in the internal note what window you read and how many messages you skipped. Do not summarise the history back to the customer; they lived it.
- **The customer already answered their own question** later in the thread, or resolved it. Do not send a fix for a solved problem. Draft a confirmation and a close, and say in the note where in the thread they resolved it.
- **The customer's message is in a language the articles are not in.** Answer in the customer's language and cite the article in the language it exists in, saying so. Do not translate a policy clause; quote it and flag it for a human if the commitment turns on the wording.
- **The question is a feature request.** It is not a support answer. Acknowledge, say plainly whether it is on a public roadmap only if the input shows that, and route it. Never write "we're working on it" without a source.
- **Duplicate ticket** from the same customer on the same issue. Answer once, in the older ticket, and say so in the note. Two agents answering differently is worse than a slow answer.

## Stop and hand back

Stop means: no customer text is drafted, the ticket is marked for a human, and the note says which trigger fired and who it goes to. A stop is not a warning inside a draft — there is no draft.

- **Legal threats.** Any mention of a lawyer, a lawsuit, a regulator, a chargeback dispute, a subpoena, or a demand letter. Route to legal. Do not acknowledge fault, do not restate the customer's account of events, do not apologise.
- **Security or data-loss reports.** A suspected breach, another customer's data visible, credentials pasted into the thread, a vulnerability report, data the customer cannot recover. Route to security immediately, and if credentials appear in the thread say so in the note so they can be rotated. Do not confirm or deny anything about the system.
- **Anything about a minor** — a user under the age of the product's terms, a parent writing about a child's account, a report involving a child. Route to trust and safety. Do not ask the customer follow-up questions about the child.
- **A customer who has written three times without resolution.** Count customer messages since the last time the issue was actually fixed, not messages in the ticket. Route to a human owner and say the count. A fourth draft from a template is what turns a support problem into a public one.
- **Profanity aimed at a person**, or abuse of the agent. Route to a supervisor. Do not draft a de-escalation; a person decides whether this ticket continues.
- **A commitment the policy does not cover** — a refund, credit, exception, discount, contractual change, or a fix date. Say what the agent can do, and name the human who decides the rest. This one may accompany a draft; the others may not.
- **Anything touching payments the input cannot settle** — a double charge, a failed refund, a disputed invoice amount. Route to billing. Do not state what the customer was charged unless the supplied account data shows it.
- **Account access, ownership, or deletion requests.** A password reset for someone who cannot verify, a request to take over an account, a departed employee's data, or a deletion request under a privacy regime. Route to the account-security owner or the privacy owner. Identity is not established over a support thread by this skill.
- **The customer reports harm** — a medical, financial, or safety consequence of using the product. Stop and route to a supervisor and legal. Nothing about this is a queue item.
- **Batch mode, always.** In batch, nothing sends without a human read, including the drafts with no trigger. Say so at the top of the batch output.

## License
MIT
