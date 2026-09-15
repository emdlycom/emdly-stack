---
name: competitor-landscape-brief
owner: launifycorp
category: Research
description: You produce a decisiongrade brief on a defined set of competitors: how each positions itself, what it charges, and what it has done in the last 612 months. The outcome you own is a CEOreadable documen...
version: v1
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/competitor-landscape-brief
raw: https://emdly.com/raw/launifycorp/competitor-landscape-brief.md
install: npx @emdly/cli add launifycorp/competitor-landscape-brief
---

# Competitor Landscape Brief

You produce a decision-grade brief on a defined set of competitors: how each positions itself, what it charges, and what it has done in the last 6-12 months. The outcome you own is a CEO-readable document that states where the company stands relative to rivals and names the two or three moves that change that standing. You do not produce a feature matrix dump or a market-size report.

## When to use

- The CEO is preparing a board or investor update and needs a current read on rival positioning and pricing.
- A competitor has announced a raise, launch, acquisition, or price change and leadership needs to know whether to respond.
- Sales is losing deals to a named rival and the loss reason is not yet understood at the positioning or pricing level.
- Annual or quarterly planning is starting and the strategy inputs are stale (last competitive review older than two quarters).
- The company is entering a new segment or geography and needs to know who already owns it.

Do not use when:

- The question is about a single deal or a single feature gap — that is a battlecard or product spec, not a landscape brief.
- No competitor set can be named or derived and the request is really "who should we worry about" — run a market map first, then come back.

## Inputs

Before starting, you need:

1. **Competitor set** — 4 to 8 named companies. If fewer than 4 are given, ask for more or derive candidates from the requester's category, buyer, and price band, then confirm the list before researching.
2. **Our own positioning baseline** — one-sentence positioning, current list pricing, and target buyer. If missing, ask for it. Do not infer our positioning from our website alone; state the source if you do.
3. **Decision this brief feeds** — pricing change, roadmap bet, board narrative, GTM entry, or general awareness. If missing, ask which one. The decision determines which section you weight.
4. **Time window** — default to the last 12 months for "recent moves." Ask if the requester wants a shorter window.
5. **Access constraints** — whether you may use only public sources or also internal win/loss notes, CRM data, and sales call transcripts. Default to public-only and label it.

If input 1 or 3 is missing, stop and ask. If 2, 4, or 5 are missing, proceed with the defaults above and state the assumption at the top of the brief.

## Method

1. **Lock the competitor set and classify each entry.** Tag every company as Direct (same buyer, same job), Adjacent (same buyer, different job), or Emerging (smaller, growing fast, or newly funded). If more than 8 make the list, cut to 8 by dropping the lowest-threat Adjacent entries and note what you dropped.
2. **Pull positioning from primary sources only.** For each competitor, capture the homepage headline, the self-described category, and the named target buyer. Use the company's own words verbatim in quotes. If a competitor's stated category differs from how buyers describe them, record both and flag the gap — that gap is usually the story.
3. **Capture pricing with a confidence tag.** Record published list price, packaging tiers, and the unit of pricing (seat, usage, platform fee). Tag each price point as *Published*, *Reported* (third-party source, name it), or *Inferred* (state the basis). If pricing is not public and you have no reported figure, write "Not public" — never estimate a number without labeling it Inferred and showing your reasoning.
4. **Build the recent-moves timeline.** For each competitor, list dated events in the window: funding, acquisitions, exec hires or departures, product launches, pricing changes, partnerships, market entries or exits. Include the date and source for each. If a competitor has zero events in the window, write "No material public moves" — an empty row is itself a signal.
5. **Score threat, and justify the score.** Rate each competitor High / Medium / Low on threat to us, using two factors: overlap with our buyer and momentum in the window. A competitor with high overlap and 3+ material moves is High. High overlap with no moves is Medium. Low overlap is Low regardless of momentum, unless their moves indicate a move toward our buyer — then Medium and say so.
6. **Identify the two or three patterns across the set.** Look for convergent pricing moves, a shared category shift in language, a buyer migration, or a gap nobody occupies. Write each pattern as a claim with the evidence that supports it. If you cannot find a pattern with at least two supporting data points, do not manufacture one — write "No cross-set pattern observed."
7. **Translate to implications for us.** For each pattern, state what it means for our positioning, our pricing, or our roadmap. Each implication must be specific enough to accept or reject; "we should monitor this" does not qualify.
8. **Write the So What.** Three or fewer recommended actions, each with an owner-type (CEO, CRO, CPO), a decision the action forces, and a timeframe. If the evidence does not support three, write fewer.
9. **Verify every price and date before shipping.** Re-open each cited source. Remove or re-tag anything you cannot confirm. State the research date at the top of the brief.

## Rules

- Never present an inferred price as a published one. Every number carries a Published / Reported / Inferred tag.
- Never cite a source you have not opened. No "according to reports" without naming the report.
- Never include a competitor claim about their own performance (customer counts, growth rates, ARR) without marking it as self-reported.
- Cap the brief at two pages of prose equivalent. If the competitor set is large, shorten per-competitor detail, not the So What.
- Do not use internal win/loss or CRM data unless input 5 explicitly permits it. If permitted, label every internal-derived claim as such.
- Missing data is written as "Not public" or "Not found" with a one-line note on what was searched. Never leave a cell blank and never fill a gap with a plausible guess.
- Do not rank competitors by feature count, headcount, or funding as a proxy for threat. Threat is overlap plus momentum, per step 5.
- Keep our own company out of the competitor table. Our position appears only in the Implications and So What sections.
- If two sources conflict on a price or date, present both with sources and flag the conflict rather than picking one silently.
- Date-stamp the brief. A landscape brief without a research date is unusable in three months.

## Output format

```
# Competitor Landscape Brief: [Category / Segment]

Research date: [YYYY-MM-DD]
Window for recent moves: [last N months]
Sources: [Public only | Public + internal (specify)]
Decision this feeds: [pricing | roadmap | board narrative | GTM entry]
Assumptions: [any defaults applied, or "None"]

## So What (read this first)

1. [Action] — Owner: [CEO/CRO/CPO] — Forces decision: [what must be decided] — By: [timeframe]
2. [Action] — Owner: [ ] — Forces decision: [ ] — By: [ ]
3. [Action] — Owner: [ ] — Forces decision: [ ] — By: [ ]

## Landscape at a glance

| Competitor | Type | Positioning (their words) | Target buyer | Pricing | Price tag | Threat |
|---|---|---|---|---|---|---|
| [Name] | Direct/Adjacent/Emerging | "[quote]" | [buyer] | [figure or "Not public"] | Published/Reported/Inferred | High/Med/Low |

## Positioning

**[Competitor]** — Claimed category: [X]. Headline: "[quote]". Buyer: [X].
Gap between claimed and perceived positioning: [note, or "None observed"].
[Repeat per competitor. 2-3 lines each.]

## Pricing

**[Competitor]** — Model: [seat/usage/platform]. Tiers: [list]. Entry price: [figure] ([tag], source: [link/name]).
Notable structure: [discounting, minimums, free tier, enterprise-only].
[Repeat per competitor.]

Cross-set pricing observation: [one paragraph, or "No pattern observed."]

## Recent moves ([window])

**[Competitor]**
- [YYYY-MM] [Event]. Source: [name/link]. Significance: [one line]
- [Or: "No material public moves in window."]
[Repeat per competitor.]

## Patterns

1. **[Pattern claim]** — Evidence: [data point 1]; [data point 2].
2. **[Pattern claim]** — Evidence: [ ]; [ ].
[Or: "No cross-set pattern observed."]

## Implications for us

- Positioning: [specific, acceptable-or-rejectable statement]
- Pricing: [ ]
- Roadmap: [ ]

## Gaps and open questions

- [What could not be confirmed, what was searched, what would resolve it]
```

## Failure modes

**The brief becomes a feature matrix.** You list capabilities per competitor and the CEO learns nothing about positioning or threat. *Check:* every competitor row must contain a positioning quote and a threat rating; if a row contains only features, rewrite it.

**Invented pricing.** Enterprise pricing is opaque, so you produce a plausible-looking number that survives into a board deck. *Check:* scan the pricing section for any figure without a Published / Reported / Inferred tag and a named source. Zero untagged numbers may ship.

**Stale moves presented as current.** You pick up an 18-month-old funding round or a launch that was later reversed. *Check:* every timeline entry has a YYYY-MM date inside the declared window, and any entry older than 6 months is confirmed still in effect.

**No So What.** The research is sound but the brief ends in description, leaving the CEO to derive the decision. *Check:* the So What section contains at least one action naming an owner and a decision it forces; if every line reads "monitor" or "consider," the brief is not finished.

**Competitor set drift.** You research adjacent players who share a category label but not a buyer, and the threat ratings become meaningless. *Check:* every Direct entry must sell to the buyer named in input 2; if not, reclassify as Adjacent or drop.

## License

MIT
