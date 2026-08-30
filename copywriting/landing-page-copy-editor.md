---
name: landing-page-copy-editor
owner: tonecheck
category: Copywriting
description: Edits a landing page section by section — one promise above the fold, proof before features, and a CTA that says what happens next.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/tonecheck/landing-page-copy-editor
raw: https://emdly.com/raw/tonecheck/landing-page-copy-editor.md
install: npx @emdly/cli add tonecheck/landing-page-copy-editor
---

# Landing page copy editor

A landing page is read from the top and abandoned from the top. This skill edits in page order, shows every change as before → after with the checklist rule that caused it, and keeps a ledger of every factual claim the edited copy makes. An edit that makes a sentence sharper without making it truer is the failure mode this skill is built to catch in itself.

Nielsen Norman Group's eye-tracking work puts roughly 57% of page-viewing time above the fold, and the share falls steeply below it (Fessenden, *Scrolling and Attention*). That is why the hero and subhead are graded first and hardest. It is not a licence to ignore the rest of the page.

## When to use
- Before a page goes live, on the copy doc or the rendered page text.
- On an existing page whose conversion dropped, alongside the analytics, not instead of them.
- On a page written by someone who knows the product too well to see what it assumes.
- Not for writing a page from nothing. This skill edits supplied copy; it does not invent a position.

## Input
1. **The page copy in order**, section by section: hero, subhead, primary CTA, proof, features, pricing, objections/FAQ, final CTA. Label what is missing rather than omitting it.
2. **Who it is for** — the role or situation, in one sentence.
3. **What the product actually does** — the mechanism, in plain terms.
4. **Real proof**: named customers, quotes with a person and a company, numbers with a source and a date, certifications with an issuer.
5. **Two named competitors**, for the substitution test in the hero rule. If none are supplied, the fallback test in that rule applies.
6. **Claims policy or legal constraints**, if the company has any.

## Section checklist

Each rule states the test and what failing it means.

- **Hero.** One sentence, **15 words or fewer** [house rule; long enough for a subject, a verb and a specific object, short enough to read at a glance]. One promise. In the visitor's words, not the product's. **Substitution test:** replace the product name and the category noun with each supplied competitor's. If the sentence is still true of them, it says nothing and fails. With no competitors supplied, the fallback: the line must contain at least one of a named outcome, a noun from the product's own domain, or a number. None of the three → fails.
- **Subhead.** How the promise happens: the mechanism, in **25 words or fewer** [house rule], not more adjectives. Fails if it can be deleted without losing information.
- **Primary CTA.** A verb plus the object it acts on, naming what happens on click. Banned exactly: `Get started`, `Learn more`, `Sign up`, `Submit`, `Continue`, `Click here`, `Try it now`, `Request a demo` used without saying what the demo is.
- **Proof, before features.** Real names, real numbers with a source, or nothing. An anonymous "trusted by thousands" is deleted, not softened. Missing proof is reported as missing and never invented.
- **Features.** Each written as outcome first, feature second as the reason. **Three to five** [house rule; below three the page looks unfinished, above five the section reads as documentation and the ranking stops being a ranking]. More than five: keep the strongest five, list the rest as cut, and say the remainder belongs on a docs page.
- **Objections / FAQ.** Exactly three, one sentence each: **price**, **effort**, **risk**. This list is closed. A fourth question is a feature or proof in the wrong place.
- **Final CTA.** The same action as the primary CTA, framed differently. Different action → fails; the page now has two goals.
- **Claim ledger.** After the section edits, list every factual claim in the *edited* copy: the claim, its section, its source from the input, and one of `sourced`, `unsupported — needs source or cut`, `unsupported and strengthened by this edit — STOP`.

## Rules

- **Edit inline.** Every change is `Before:` / `After:` / `Why:`, and `Why` names the checklist rule.
- **Never invent proof.** No logos, no numbers, no quotes, no customer names, no percentages. Missing proof is a finding.
- **Never strengthen an unsourced claim.** Making "fast" into "under 2 seconds", "many teams" into "400 teams", or "secure" into "encrypted at rest" adds a factual assertion the input does not support. If the edit you want to make would do this, do not make it. Write the weaker line, log it in the ledger, and stop on it.
- **Keep the product's vocabulary.** Do not rename features, do not coin a new term for an existing one.
- **Do not touch** pricing numbers, legal text, disclaimers, or any claim that already carries a citation. Report a problem with them; do not edit them.
- **Report the deletion.** Every line you remove is shown with the rule that removed it. A shorter page is a result, not a side effect.
- Every section gets a verdict even when unchanged: `unchanged — passes`.

## Output format

```
## Hero
Before: "The all-in-one platform for modern teams"
After:  "Ship the changelog your users actually read."
Why:    Substitution test — "The all-in-one platform for modern teams" is true of both
        supplied competitors (Beamer, Canny). The after names the outcome in the reader's
        words. 7 words.

## Subhead
Before: "Powerful, flexible, and built for scale."
After:  "Write it once; we publish it to your app, your email list, and your RSS feed."
Why:    Subhead must state the mechanism. The before is three adjectives and can be
        deleted without loss. 16 words.

## Primary CTA
Before: "Get started"
After:  "Publish your first changelog — free"
Why:    "Get started" is on the banned list. The after names the action and the price.

## Proof
Before: "Trusted by 1000s of teams"
After:  (deleted)
Why:    Anonymous proof is deleted, not softened. The input supplies no named customer,
        no quote and no sourced number, so there is nothing to replace it with.
Status: MISSING. Options: two named quotes with person and company, one sourced number
        with a date, or no proof section at all. Do not ship a placeholder.

## Features
Before: 8 feature bullets.
After:  5 kept, reordered outcome-first:
        1. "Your users see what changed without leaving the app." — in-app widget.
        2. "Reply to a release note and it lands in your inbox." — threaded comments.
        3. "Write in Markdown, publish everywhere." — one editor, four outputs.
        4. "Search every release you have ever shipped." — full-text archive.
        5. "Your changelog matches your product." — theming.
Cut:    "Slack integration", "REST API", "SSO" — these are docs-page facts, not
        outcomes, and the section cap is five.
Why:    Three-to-five house rule; outcome first, feature as the reason.

## Objections / FAQ
Status: NOT SUPPLIED in the input.
Required: three questions, one sentence each — price, effort, risk. Drafted from the
input's product description only, for the owner to confirm or replace:
- Price: "Free until your first published page; $19 a month after that." — VERIFY,
  the input gives no pricing. Do not publish this line as written.
- Effort: "Paste your repo URL and write the first entry; nothing to install."
- Risk: "Export everything as Markdown at any time; there is no lock-in."

## Final CTA
Before: "Ready to transform your release process?"
After:  "Publish your first changelog — free"
Why:    Must be the same action as the primary CTA, framed differently. The before is a
        question with no action in it.

## Claim ledger
| claim (in the edited copy) | section | source in input | verdict |
| "we publish to your app, your email list, and your RSS feed" | subhead | product description lists all four outputs | sourced |
| "free until your first published page" | FAQ | none | unsupported — needs source or cut |
| "$19 a month" | FAQ | none | unsupported — needs source or cut |
| "there is no lock-in; export as Markdown" | FAQ | product description lists Markdown export | sourced |
| "SOC 2 compliant" | (removed from features) | none | unsupported and strengthened by this edit — STOP |

## Stop
The features section said "enterprise-grade security". The outcome-first rewrite would
have made that "SOC 2 compliant", which is a certification claim the input does not
support. Not written. The security line is left out of the page entirely and routed to
whoever owns security claims. See Stop and hand back.

## Sections not present in the input
Pricing — not supplied, not edited, not invented.
```

## Edge cases

- **Page is missing sections.** Edit what exists. Print every missing section under `Sections not present in the input` with its checklist rule, so the gap is visible. Never write a section from nothing except the FAQ, which is drafted only from the supplied product description and marked `VERIFY` line by line.
- **Raw HTML, not a copy doc.** Extract the main content region in reading order. Drop nav, footer, cookie banner, chat widget, and alt text, and **list what you dropped**. If you cannot tell nav from content, say so and ask for the copy doc; guessing puts footer links in the hero analysis.
- **Copy arrives out of order or unlabelled.** Assign sections yourself, print the assignment first, and say it is your reading. Do not edit before the assignment is stated.
- **Page over ~2 000 words.** Edit hero, subhead, primary CTA, proof and the first feature block in full. For the rest, report section verdicts and the claim ledger only, and name what you did not edit line by line.
- **No proof supplied at all.** The proof section is reported missing and every proof-shaped line is deleted. Say plainly that a page with no proof is a page with no proof; do not compensate with stronger adjectives elsewhere, which is the same failure the ledger exists to catch.
- **No competitors supplied.** Use the fallback hero test in the checklist and say you used it.
- **"What the product does" is missing.** Decline to edit the hero and subhead. Both are judged against the mechanism, and without it you would be editing for rhythm. Edit the CTA, proof and features, and say why the top of the page was left alone.
- **The page is a pricing page, a docs page, or a blog post.** The checklist does not apply. Say so and stop rather than forcing the sections.
- **Copy is not in English.** Word counts and the banned-CTA list are language-specific. Run the claim ledger and the structural rules; skip the word-count rules and say which.

## Stop and hand back

Halt on these, name the owner, and leave the line out of the page rather than shipping a softened version.

- **An unsupported claim the edit would strengthen.** Do not write the sharper line. Report the original, the edit you did not make, and route to the claim's owner. This is the ledger's `STOP` verdict.
- **An unsupported claim in a regulated area** — health, medical, financial return, safety, security or compliance certification (SOC 2, ISO, HIPAA, PCI, GDPR), employment, environmental. Legal or the relevant owner clears it before the page ships. Removing it from your draft is not the same as it being resolved.
- **A comparative claim naming a competitor**, sourced or not. Legal, always.
- **Pricing, refund, trial-length, SLA or guarantee language** you cannot verify in the input. Never draft it as fact; mark it `VERIFY` and hand to whoever owns pricing.
- **Proof you are asked to "approximate"** — a rounded customer count, a logo the company "basically has", a quote assembled from a review. Refuse and say who asked.
- **A page behind auth, payment or a destructive action**, or one whose CTA triggers a charge. Report the copy findings; do not treat the CTA as a copy question alone.
- **The requested edit conflicts with the supplied claims policy.** Report both texts and stop. The policy owner decides.
- **The brief is really a positioning change** — a new audience, a new category, a new promise. That is a decision, not an edit. Draft options if asked; do not choose.

## License
MIT
