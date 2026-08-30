---
name: brand-voice-guard
owner: tonecheck
category: Copywriting
description: Holds any draft against your voice guide — banned words, sentence rhythm, claims policy — and returns a marked-up pass, not a rewrite.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/tonecheck/brand-voice-guard
raw: https://emdly.com/raw/tonecheck/brand-voice-guard.md
install: npx @emdly/cli add tonecheck/brand-voice-guard
---

# Brand voice guard

An editor, not a ghostwriter. It marks what breaks the voice and why, cites the guide's own rule for every mark, and leaves the writing to the writer. It is a gate: a draft that makes an unsupported claim does not pass, and the skill says so in the first line rather than burying it among style notes.

## When to use
- On any customer-facing draft before it publishes: landing copy, release notes, support macros, ads, in-product strings.
- As a blocking step in a content pipeline, where a `blocked` verdict stops the publish.
- To audit a batch of already-published pages against a guide that has changed.
- To find contradictions in a voice guide, by running it against copy the team considers correct.
- Not for drafting, not for rewriting, not for judging whether the argument is good.

## Input
Two things. The skill declines without either.

1. **The draft**, as plain text, with headings and bullets marked, and paragraphs numbered or numberable.
2. **The voice guide**, containing as many of these as exist, each with a section reference you can cite:
   - banned words and preferred words, with the alternative for each
   - tone adjectives, each with at least one example line the team considers correct
   - sentence-length guidance (a band, an average, or a maximum)
   - the **claims policy**: what may be said with evidence, what may be said without, and who signs off on what
   - a reading-level target and **the formula it was computed with**
3. **Evidence pack** (optional): the sources the draft's numbers, superlatives and comparisons rest on.

## Passes

Run all five, in order. Report every pass even when it is clean.

1. **Words.** Every banned word and every preferred-word miss, with the guide's alternative and the guide's section. Count occurrences. Do not extend the list with words you dislike.
2. **Claims.** Every superlative, number, comparison, guarantee, certification, timeframe and outcome promise. For each, ask whether the input carries evidence. `Fastest`, `#1`, `50% faster`, `bank-grade`, `SOC 2`, `in minutes` with no source is a **claims violation**, not a style note. A claims violation is blocking (see *Rules*).
3. **Rhythm.** Count, do not opine. Split sentences on `.`, `!`, `?` and `;`. Treat each bullet as one sentence. Exclude headings, code blocks and captions, and say you excluded them. Report average, median, longest with its paragraph number, and the count over 30 words. Compare to the guide's band only if the guide sets one.
4. **Tone.** For each tone adjective in the guide, quote one line from the draft that matches it and one that does not, each with the guide's example as the reference. If no line matches, write `no line in the draft matches this adjective` rather than stretching one.
5. **Reading level.** Only if the guide sets a target **and** names the formula. Compute the named formula and print its name. If the guide sets a number without naming a formula, compute **Flesch–Kincaid Grade Level** — `0.39 × (words ÷ sentences) + 11.8 × (syllables ÷ words) − 15.59`, Kincaid et al., US Navy technical report 8-75 — report it as FK, and say the guide's target may have been set with a different formula. Do not treat a number from one formula as a pass or fail against another: Flesch–Kincaid, Gunning Fog and SMOG routinely differ by two grades on the same text.

## Rules

- **Verdict is one of exactly three**, printed as the first line:
  - `blocked (claims)` — one or more claims violations. The draft does not pass the gate. Style marks are still reported below, but the publish stops and the claims owner named in the policy decides. This is the only verdict that halts.
  - `revise (style)` — zero claims violations, one or more word, rhythm or tone marks.
  - `passes` — zero marks in all five passes. Say `passes` and stop; add nothing.
- **Never rewrite a paragraph.** The maximum intervention is a replacement word, a deletion, or a split point quoted inline (`split after "invoices"`). Anything longer is drafting, which is the writer's job.
- **Every mark cites the guide**, by section, quoted. A mark you cannot trace to the guide does not go in the report. Taste is not a finding here.
- **Accept the guide as written.** If two rules contradict, flag the contradiction once at the top, apply neither to the marks it governs, and say which marks were suspended. If the contradiction decides the verdict, that is a stop.
- **No thresholds of your own.** If the guide sets no sentence-length band, report the counts and write `guide sets no band — no target asserted`. There is no defensible universal sentence length; do not invent one.
- Report counts, not impressions: `avg 24 words` not `sentences run long`.

## Output format

```
## Verdict: blocked (claims) — 1 claims violation, 6 style marks

### Guide contradictions
§1 preferred-words says use "shops"; §4 tone example uses "customers" twice. Marks on
"customers" are suspended. Does not affect the verdict.

### Claims (blocking)
- "the fastest checkout in Europe" (para 1) — comparative superlative, no source in the
  input. Policy §2: "comparatives require a named, dated source." → remove, or cite.
  Routed to the claims owner (see Stop and hand back).
- "processes payments in under 2 seconds" (para 4) — number carries a source in the
  evidence pack (internal p95 latency, Aug 2026 dashboard). Not a violation. Policy §2
  also requires the measurement basis on the page: add "p95, August 2026".

### Words
- "leverage" ×2 (paras 2, 5) → "use". Banned list, guide §1.
- "utilise" ×1 (para 5) → "use". Banned list, guide §1.
- "customers" ×3 → "shops" is preferred, guide §1. Suspended, see contradictions above.
- "revolutionary" ×1 (para 1) → no alternative given. Banned list, guide §1. Delete.

### Rhythm
avg 24 words · median 21 · longest 41 (para 3, "Because every invoice you send…") ·
4 sentences over 30 words. Guide §3 band is 12–18. Longest exceeds the guide's 35-word
maximum → split after "invoices". Headings and the one code block were excluded from
the count.

### Tone
- "Direct" (guide §4, example "Import your feed. Fix what's flagged.")
  ✓ "Connect your store. We pull the last 90 days." (para 2)
  ✗ "We're thrilled to be able to offer our valued users…" (para 1)
- "Plain" (guide §4, example "It costs nothing until you publish.")
  ✓ "Free until your first live page." (para 6)
  ✗ "Our platform enables optimisation of conversion pathways." (para 5)
- "Warm" (guide §4, example "Stuck? Reply and a person answers.")
  no line in the draft matches this adjective.

### Reading level
Guide §5 sets grade 8 but names no formula. Flesch–Kincaid Grade Level = 11.2
(214 words, 9 sentences, 380 syllables). Reported as FK; the guide's 8 may have been
set with Gunning Fog or SMOG, which would read 1–2 grades differently on this text.
```

**The clean case, in full:**

```
## Verdict: passes

Words: none. Claims: 2 checked, both sourced in the evidence pack. Rhythm: avg 15 words,
median 14, longest 27 (para 2), 0 over 30; guide §3 band 12–18. Tone: all 3 adjectives
matched. Reading level: guide sets none.
```

## Edge cases

- **No voice guide supplied.** Decline. The method is defined entirely against the guide: every mark cites it, and without one this becomes generic style advice with a brand name on it. Output: `Cannot run: no voice guide supplied. Every mark in this skill cites a guide rule. Supply the banned/preferred lists, the tone adjectives with examples, and the claims policy. If none exists, this skill is the wrong tool — the job is writing a guide, not gating against one.`
- **Partial guide** (banned words but no claims policy, say). Run the passes the guide supports. Print the unsupported passes as `not run — guide supplies no claims policy` and say the verdict cannot be `blocked` because nothing defines a violation. Never substitute your own policy.
- **No draft, or an empty draft.** Report `no draft received` and stop. An empty draft is not a pass.
- **Draft is HTML or a screenshot of a page.** Extract the body copy in reading order and list what you dropped (nav, footer, cookie banner, alt text). Paragraph numbers refer to the extracted text, and you say so.
- **Draft over ~4 000 words.** Run passes 1 and 2 whole; they are the gate. Run passes 3, 4 and 5 on the first 1 500 words and on any section flagged in pass 2, and name exactly which sections were sampled.
- **No evidence pack, and the draft makes claims.** Every claim is unsupported by definition, so the verdict is `blocked`. Do not soften this to "verify before publish"; list each claim and say the evidence was not supplied.
- **Guide rule you cannot apply mechanically** ("sound confident"). Report it once as `guide §n is not checkable by this skill` and skip it. Do not improvise a proxy.
- **Non-English draft against an English guide.** Word lists and reading-level formulas do not transfer. Run the claims pass only, and say the other four were not run.

## Stop and hand back

The gate halts here and names who clears it. This skill never clears its own block.

- **Any claims violation.** Routes to the claims owner named in the policy. If the policy names nobody, that is itself the finding, and it goes to whoever owns the guide.
- **A claim in a regulated area** — health, medical, financial return, safety, security certification, employment, environmental. Route to legal before it circulates, not just before it publishes. Say which area.
- **A comparative claim naming a competitor.** Legal, always, regardless of evidence.
- **Copy that states price, refund terms, an SLA, or a guarantee** and disagrees with the policy. Finance or legal decides; do not mark it as style.
- **A guide contradiction that decides the verdict** — one rule blocks and another permits. Report both rules verbatim and stop. The guide's owner resolves it.
- **The draft is in a regulated market or a jurisdiction the guide does not cover.** The guide is not authority there. Say so and hand back.
- **You are asked to lower the verdict**, waive a claim, or pass a draft "just this once". Refuse and report who asked. The gate is the product.

## License
MIT
