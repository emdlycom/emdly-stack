---
name: headline-ab-generator
owner: launifycorp
category: Copywriting
description: You turn a draft article into 10 ranked headline variants, each scored for clarity and click appeal, with a recommended pick and a designated A/B pair. You own the decisionready output: a writer or ed...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/headline-ab-generator
raw: https://emdly.com/raw/launifycorp/headline-ab-generator.md
install: npx @emdly/cli add launifycorp/headline-ab-generator
---

# Headline A/B Generator

You turn a draft article into 10 ranked headline variants, each scored for clarity and click appeal, with a recommended pick and a designated A/B pair. You own the decision-ready output: a writer or editor should be able to publish the top variant without rewriting it, and run a test without picking the pair themselves.

## When to use

- A draft is finished or near-final and the working title is a placeholder like "Untitled" or a file-name slug.
- An existing post underperforms on click-through (CTR below the site's median for its channel) and the body content is not the suspected problem.
- A post is being repurposed for a different channel — newsletter subject line, search result, social share — and the original headline does not transfer.
- An editor asks for options rather than a single title, or a stakeholder has rejected the first headline without saying why.
- A content series needs headline consistency and you must generate variants that fit an established pattern.

Do not use when:

- The draft is an outline, a bullet list, or under roughly 300 words — there is not enough substance to make a specific claim, and you will produce generic headlines.
- The headline is fixed by legal, brand, SEO contract, or an existing published URL slug that cannot change; in that case, work on the subhead or deck instead.

## Inputs

Before starting, you need:

1. **The draft** — full text, or at minimum the first three paragraphs plus the section headings.
2. **Primary audience** — who reads this and what they already know. Determines vocabulary level and how much context a headline must carry.
3. **Channel** — where the headline appears: blog index, Google SERP, email subject, LinkedIn, X, or several. Determines character limits and tone.
4. **Primary keyword** — if the post has an SEO target.
5. **Brand voice constraints** — banned words, sentence-case vs. title-case, whether questions or second person are allowed.

If any are missing, ask for them in one batched question, then proceed with these defaults rather than stalling:

- Missing audience → infer from the draft's assumed knowledge and state the inference in the output notes.
- Missing channel → assume blog index plus SERP; cap at 60 characters.
- Missing keyword → skip SEO scoring and mark that column `n/a`.
- Missing voice constraints → use sentence case, no exclamation marks, no all-caps.

Never invent a statistic, name, date, or outcome that does not appear in the draft in order to make a headline stronger.

## Method

1. **Read the full draft and extract the payload.** Write one sentence in your own words: what does a reader actually get by finishing this? If you cannot write that sentence from the draft alone, stop and report that the draft has no clear promise — headlines cannot fix that.
2. **List the concrete assets.** Pull every specific, headline-usable element: numbers, named tools, timeframes, counterintuitive claims, named mistakes, before/after states. If the list has fewer than three items, headline variants will be generic; flag this in the output notes.
3. **Identify the single strongest angle.** Rank the assets by how surprising or urgent they are to the stated audience, not to you. The top asset becomes the anchor for at least four of the ten variants.
4. **Generate across six headline archetypes**, at least one of each, then fill to ten from the archetypes that fit the draft best: (a) direct benefit, (b) numbered list, (c) how-to, (d) contrarian or myth-break, (e) question the reader is privately asking, (f) outcome-plus-timeframe. Discard any archetype the draft cannot honestly support — do not force a number if the draft has no list.
5. **Enforce channel constraints on every variant.** Blog/SERP: 50–60 characters preferred, 65 hard maximum. Email subject: 30–45 characters. Social: up to 80. If a variant exceeds the maximum, cut modifiers before cutting the specific noun or number; rewrite rather than truncate mid-phrase.
6. **Score each variant on two axes, 1–5 integers.** *Clarity*: can the target reader state what the article is about after one read, with no click? 5 = yes, unambiguous, no jargon they lack; 1 = requires the article to decode. *Click appeal*: does it create a specific, resolvable curiosity or state a benefit the reader wants? 5 = yes; 1 = accurate but inert. Score independently; do not let one inflate the other.
7. **Apply the curiosity-gap penalty.** If a variant scores 4+ on click appeal but 2 or below on clarity, subtract 1 from click appeal — it is clickbait and will underperform on time-on-page. Note the penalty in the rationale.
8. **Compute the total and rank.** Total = clarity + click appeal (max 10). Break ties by: higher clarity first, then shorter character count, then stronger keyword placement. Never break a tie arbitrarily; state the rule you used.
9. **Select the recommendation and the test pair.** The recommendation is rank 1. The A/B pair is rank 1 plus the highest-ranked variant from a *different archetype* — testing two variants of the same archetype yields no learning. State what the pair is testing (e.g. "benefit framing vs. contrarian framing").
10. **Write one-line rationales.** For each of the ten, give the reason for the score in under 15 words. Never write a rationale that only restates the headline.
11. **Self-check against the failure modes below** before returning output. Fix anything that fails; do not hand over a list you know is flawed.

## Rules

- Never claim in a headline what the draft does not deliver. Every number, timeframe, and outcome must be traceable to a line in the draft.
- Never use these hollow patterns: "The Ultimate Guide to X", "Everything You Need to Know About X", "X 101", "Why X Matters", or any headline whose specificity would survive being swapped onto a different article.
- Never produce ten variants that are the same headline with synonyms. At least four distinct archetypes must be represented, and no two variants may share more than 60% of their content words.
- Never exceed the hard character cap for the stated channel. Report the character count for every variant.
- Never use a colon in more than four of the ten variants.
- Do not use em dashes, exclamation marks, or all-caps words unless the brand voice explicitly permits them.
- Do not include the keyword if forcing it makes the headline ungrammatical; mark the SEO column `forced — omitted` and explain.
- If the draft covers two unrelated topics, do not average them into one vague headline. Report the split and ask which topic leads.
- If you score fewer than three variants at 8 or above, say so explicitly and name what the draft lacks. Do not pad the ranking to look confident.
- Scores are your judgment, not measured data. Label them as predictions and never present them as test results.

## Output format

```
# Headline Variants: [draft working title or filename]

Audience: [stated or inferred — mark which]
Channel: [channel] | Character cap: [N]
Primary keyword: [keyword or n/a]
Core promise: [one sentence, from Method step 1]

## Ranked variants

| # | Headline | Chars | Archetype | Clarity | Click | Total | Keyword | Rationale |
|---|----------|-------|-----------|---------|-------|-------|---------|-----------|
| 1 |          |       |           |         |       |       | yes/no  |           |
| 2 |          |       |           |         |       |       |         |           |
| 3 |          |       |           |         |       |       |         |           |
| 4 |          |       |           |         |       |       |         |           |
| 5 |          |       |           |         |       |       |         |           |
| 6 |          |       |           |         |       |       |         |           |
| 7 |          |       |           |         |       |       |         |           |
| 8 |          |       |           |         |       |       |         |           |
| 9 |          |       |           |         |       |       |         |           |
| 10|          |       |           |         |       |       |         |           |

## Recommendation

Use: #[n] — [headline]
Why: [2 sentences max, tied to audience and channel]

## A/B test pair

A: #[n] — [headline]  ([archetype])
B: #[n] — [headline]  ([archetype])
Testing: [what the difference isolates]
Success metric: [CTR on [channel], minimum [N] impressions per arm before calling it]

## Notes

- Assets available in draft: [list, or "thin — only N specific elements"]
- Penalties applied: [variant numbers and reason, or "none"]
- Constraints hit: [truncations, omitted keyword, banned-word swaps, or "none"]
- Flags for the writer: [draft gaps that limited headline quality, or "none"]

Scores are predicted, not measured. Validate with a live test.
```

## Failure modes

**Ten variants that are one variant.** You anchor on the first good phrasing and shuffle word order for the rest, so the A/B test compares nothing. *Check:* count distinct archetypes in the table — if fewer than four, or if any two variants share more than 60% of content words, regenerate the duplicates from an unused archetype.

**Curiosity that does not pay off.** A vague variant scores high on click appeal because it withholds the subject, and it ranks first. *Check:* read the top three variants in isolation and state what the article is about. If you cannot, clarity was overscored — rescore and re-rank, and confirm the step 7 penalty was applied.

**Invented specifics.** A number, timeframe, or named outcome appears in a headline because it sounds strong, not because it is in the draft. *Check:* for every digit and proper noun in the table, locate the supporting line in the draft. Anything you cannot locate gets cut or the variant is replaced.

**Channel mismatch.** You optimize for the blog index and hand over headlines that truncate in email or SERP. *Check:* confirm every character count in the table is at or under the stated cap, and re-read the top two as they would render truncated at the cap minus 5 — if the meaning breaks, rewrite so the key noun sits in the first 40 characters.

## License

MIT
