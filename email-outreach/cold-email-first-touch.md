---
name: cold-email-first-touch
owner: outboundlab
category: Email & outreach
description: Writes first-touch e-mails that sound like a person — one observation, one sentence of value, one soft ask. Kills every buzzword on sight.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/outboundlab/cold-email-first-touch
raw: https://emdly.com/raw/outboundlab/cold-email-first-touch.md
install: npx @emdly/cli add outboundlab/cold-email-first-touch
---

# Cold e-mail, first touch

The only cold e-mail that gets answered is the one that could not have been sent to anyone else. This skill enforces that. The output is one mail to one named person who did not ask to hear from you, and it must be true of every mail that a reader who deleted it would still agree the sender had read something specific about their company before writing. That means the observation is a real artefact with a place you can name, the proof carries a number, the ask is small enough to say yes to in a sentence, and where the research is thin the skill returns nothing rather than a mail that opens with flattery.

## When to use
- Per prospect, with research notes as input, when you have one specific artefact about their company and not just a firmographic row.
- For a sequence, this is mail 1 only. Follow-ups reference a prior mail and are a different skill.
- To rewrite a template that has gone stale, by feeding the template plus one prospect's research and keeping only what survives the observation rule.
- Not for: an inbound lead, a warm intro, a renewal, or anyone who has replied before. Those have context this skill deliberately ignores.
- Not for: a list. Run it once per prospect, or it will produce the interchangeable mail it exists to prevent.

## Input

Supply all three. Say in one line which you were given.

1. **Prospect** — name, role, company, and **one verifiable observation** from this closed list: a job post, a product or pricing change, a talk or podcast appearance, a public number (funding, headcount, a filing, a published metric), a published engineering or company blog post, a conference session, a documented integration or migration. Each observation carries where you found it and roughly when.
2. **Sender** — who you are and the one thing you do, in one sentence.
3. **Proof** — a single named case in one sentence with a number, if you have one. If you do not, say so; the proof line changes shape rather than being invented.

The seven observation types above are the whole list. Anything else — a funding round three years old, an industry trend, "you're in ecommerce", a LinkedIn profile photo, an anniversary — is not an observation and does not unlock a mail.

## The four lines

Every mail is four lines in this order, one sentence each. No fifth line.

1. **Observation.** Name the artefact from the Input list and where it is. Quote it or cite it. The test is not "did it take effort to know" — that is unfalsifiable and the skill does not use it. The test is: does this sentence name one of the seven Input types, with a locator (the job post, the changelog entry, the talk, the filing)? If it does not, the mail does not get written.
2. **Bridge.** One sentence connecting the observation to a problem you solve, stated as a question or a hypothesis. Use "so I assume", "if the blocker is", "which usually means". Never "we help companies like yours".
3. **Proof.** One sentence: who else, what changed, one number. Named customer or a named segment, not "our clients".
4. **Ask.** Small and reversible: "worth a 15-minute look?", "should I send the two-pager?", "is this the wrong read?". Never "book a time on my calendar", never a link to a scheduler.

## Rules

- **Body under 60 words**, counting lines 1 to 4 and the greeting, excluding the signature block. `[judgment, anchored on Boomerang 2016]` — Boomerang's analysis of ~40 million e-mails found bodies of 50–125 words drew the highest response rates. 60 is the low end of that band, chosen because four sentences fit in it and a fifth does not. It is not a measured optimum; it is a house rule that enforces the four-line structure. Anywhere in 50–125 is defensible if you report the count you used.
- **Subject 5 words or fewer**, lowercase is fine. `[judgment, anchored on Boomerang 2016]` — the same analysis put the best-performing subject lines at 3–4 words. 5 is one word of slack, not a finding. Banned outright: "Quick question", "Following up", "Touching base", the prospect's first name alone, and any subject that would read identically to another company.
- **Banned words and phrases**, in subject or body: "I hope this finds you well", "I'd love to", synergy, leverage, streamline, solution, innovative, revolutionary, game-changer, circle back, reach out, "at your convenience", "picking your brain", "crushing it", "in the space".
- No flattery of any kind. No fake familiarity. No reply-bait ("did you see my last mail?") on a first touch.
- **One link at most**, and only in the proof line. Zero links is fine.
- Sign with a name and a role. No banners, no calendar links, no PS, no images, no tracking pixel note.
- **If the input lacks an observation from the Input list, do not write the mail.** Return `no observation — research first` and name which of the seven types to go looking for on this specific company.
- Do not soften the refusal by writing a mail anyway with a placeholder. A bracketed `[observation]` in a draft becomes a sent mail.

> Thresholds above are defaults; report the thresholds you used.

## Output format

A subject, a body, a signature, then a one-line count so the limits are checkable.

### Example 1 — the normal case

```
Subject: your PostgreSQL 13 migration

Hi Mara — your team posted a Postgres 16 migration role last week, so I assume the 13 → 16 jump is on the roadmap.

If the blocker is the downtime window, that is what we do: Lumen cut theirs from 6 h to 40 min last quarter.

Worth a 15-minute look?

Sean Callahan · Outbound Lab

[subject 4 words · body 50 words · observation: job post, careers page, last week · 0 links]
```

### Example 2 — no proof case supplied

The proof line does not disappear and is not invented. It becomes a specific offer of evidence.

```
Subject: eu data residency in 2.4

Hi Tomas — your 2.4 changelog says EU data residency shipped for new tenants only, which usually means the back-fill for existing ones is the hard part.

We have done that migration twice; I can send what the cutover looked like, no call.

Is that the wrong read?

Priya Raman · Outbound Lab

[subject 5 words · body 47 words · observation: changelog entry 2.4, 12 Aug · 0 links · no proof case supplied, proof line offers evidence instead]
```

### Example 3 — the refusal

```
no observation — research first

Prospect: Dana Whitfield, VP Operations, Corvid Logistics.
Supplied: company size (240), industry (3PL), "growing fast".
None of these is one of the seven Input types. Company size and industry are firmographics;
"growing fast" is not sourced to anything.

Look for, in this order:
1. Job posts — Corvid's careers page. An ops or WMS role names the system they run.
2. A product or pricing change — their site's changelog or pricing page in the Wayback Machine.
3. A public number — 3PLs of this size often publish warehouse counts or square footage in
   press releases.

No mail written.
```

## Edge cases

- **No observation supplied.** Refuse as above. This is the common case and it is not a failure of the skill.
- **An observation that is not on the list** — a trend, a competitor's news, a firmographic, an old funding round. Treat as no observation. Say which type it was mistaken for and refuse.
- **Several observations supplied.** Use one. Pick the most recent that connects to what you sell; a stale but relevant artefact beats a fresh irrelevant one. Name the one you used and say you discarded the others.
- **The observation is older than about six months.** Still usable, but say when it was ("your March pricing change"), never imply it is news. If it is the only observation and it is over two years old, refuse: it no longer distinguishes this prospect.
- **No sender line supplied.** The bridge and proof cannot be written; both depend on knowing what you do. Return `no sender — cannot write the bridge` rather than guessing from the observation.
- **No proof case supplied.** Do not invent a customer, a number, or "companies like yours see". Use the Example 2 shape: name what you have done in plain terms and offer the evidence. Report `no proof case supplied` in the count line.
- **Research notes are long** — a full account dossier, several pages. Do not read it all into the mail. Extract at most three candidate observations, pick one, and say which. The mail does not get longer because the research did.
- **Research notes are empty or are only the prospect's name and title.** Refuse. Do not enrich from general knowledge about the company; general knowledge is by definition not an observation.
- **The prospect's name is a placeholder** (`{{first_name}}`, "there", "team"). Refuse. A merge field in the input means this is a list, and this skill does not write to lists.
- **Two people at the same company.** Write two mails with different observations, or write one and say who not to also mail. Two near-identical mails into one company reads as automation and costs both.
- **The company has no public surface** — no careers page, no changelog, no press. Say so plainly: the observation-first method does not work on this prospect and no amount of writing fixes it. Report `no public surface — not a fit for cold first touch` and suggest a different channel rather than degrading to a generic mail.

## Stop and hand back

Stop means: no mail is written and none is queued. Say which trigger fired and who it went to.

- **The observation came from somewhere non-public.** A leaked or forwarded internal document, a private Slack or community, a scraped internal directory, an intent-data vendor's page-level browsing signal, or anything you would not be willing to name in the mail itself. If you cannot write "your careers page says" and have it be both true and comfortable, stop and hand to whoever owns your data sourcing.
- **The observation is about a person rather than a company** — their health, their departure, a lawsuit naming them, a personal social post. Stop. Company artefacts only.
- **The observation is a distress event the company has not itself announced** — layoffs, a breach, an outage, insolvency, a death. Stop and hand to the account owner. Mailing on an unannounced distress event is the fastest way to be the vendor that did that.
- **Consent-regime recipients.** Canada (CASL), Germany, and jurisdictions requiring prior consent for commercial mail, or any recipient who is an individual rather than a business contact. Do not send on this skill's authority. Hand to whoever owns consent and suppression, and say which regime you think applies rather than deciding it.
- **No suppression list was supplied**, or you cannot confirm the address is not on one — an unsubscribe, a customer, an open opportunity, a past "do not contact". Stop. Sending twice into a suppression is a compliance event, not a style problem.
- **A personal address** (a consumer mail domain) where a work address exists or is unknown. Stop and hand back; ask for the work address.
- **The prospect is at a company under an active contract, negotiation, or dispute with you.** Hand to the account owner. A cold first touch into a live relationship is not cold.

## License
MIT
