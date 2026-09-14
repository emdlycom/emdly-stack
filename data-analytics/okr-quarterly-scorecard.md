---
name: okr-quarterly-scorecard
owner: launifycorp
category: Data & analytics
description: Drafts a monthly investor update email from a founder/CEO to existing investors, combining hard metrics, narrative context, honest risk disclosure, and specific asks. You own the finished draft: metri...
version: v1
license: MIT
updated: 2026-09-14
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/okr-quarterly-scorecard
raw: https://emdly.com/raw/launifycorp/okr-quarterly-scorecard.md
install: npx @emdly/cli add launifycorp/okr-quarterly-scorecard
---

# Investor Update Email

Drafts a monthly investor update email from a founder/CEO to existing investors, combining hard metrics, narrative context, honest risk disclosure, and specific asks. You own the finished draft: metrics table, three-to-five-sentence commentary per section, and asks that name a person, company, or intro target. The CEO should be able to send it after a factual review, not a rewrite.

## When to use

- The monthly or quarterly investor update is due and the month's numbers are closed.
- A material event (term sheet signed, key hire, churn of a top-10 customer, pivot) requires an off-cycle note to the cap table.
- The CEO is behind on updates and needs to restart the cadence, possibly covering multiple months.
- A fundraise is 3-6 months out and updates need to build momentum with existing investors and warm the round.
- A board meeting produced decisions that non-board shareholders should hear directly.

Do not use when:

- The message is a single-purpose ask (one intro, one signature, one question) — send a direct email instead.
- The recipient list includes prospective investors who have not invested; that is a fundraising deck or teaser, governed by different disclosure norms.

## Inputs

Collect before drafting:

- **Period** — month/quarter and year, and the date of the last update sent.
- **Core metrics** — current value, prior period value, and target for each. At minimum: revenue (MRR/ARR or equivalent), growth rate, net/gross retention, cash balance, net burn, runway in months. Add 2-4 company-specific metrics (active users, GMV, pipeline, gross margin, units shipped).
- **Wins** — 3-5 concrete events with names: customers closed, hires made, product shipped, partnerships signed, press.
- **Lowlights / risks** — what missed plan, what broke, what is at risk in the next 90 days.
- **Asks** — 2-4 items, each with a named target or a precise profile.
- **Team changes** — headcount now vs. prior period, key departures and starts.
- **Fundraise status** — if applicable: stage, target amount, timeline, current status.
- **Recipient list and tone precedent** — prior updates, if any.

If inputs are missing, ask for them in this priority order: (1) core metrics with prior-period comparison, (2) asks, (3) risks, (4) wins. Do not proceed past a draft skeleton without (1). If the CEO cannot supply a metric, mark it `Not tracked` or `Not available this period` in the draft — never estimate, interpolate, or carry forward a stale number silently.

## Method

1. **Confirm the period and the comparison baseline.** If the last update was more than one period ago, set the comparison to the last reported period and state the gap explicitly in the opening line. If this is the first update, use company inception or the last fundraise as baseline.
2. **Build the metrics table first, before writing prose.** Every row gets: metric, current, prior, change (% or absolute), and plan/target if a target exists. If a metric moved more than 20% in either direction, it must be explained in the commentary — no unexplained swings.
3. **Write the TL;DR after the table, not before.** Three to four sentences maximum: the single most important number, the single most important event, the single biggest risk, and the headline ask. If you cannot name one most important number, the metrics set is too diffuse — flag it to the CEO.
4. **Draft wins with proof, not adjectives.** Each win gets a name and a number: "Closed Acme Corp, $48K ARR, 14-month sales cycle" not "landed a major enterprise logo." Drop any win that cannot carry a name or a number.
5. **Draft lowlights before you draft asks.** Rule: the lowlights section must be at least as long as the wins section is short — minimum two items, and at least one must be something the CEO controls (execution, hiring, pricing), not only market conditions. If the CEO supplied zero lowlights, ask again; a zero-lowlight update destroys credibility.
6. **Convert each ask into an action a reader can complete in under 10 minutes.** Test: does it name a target (person, company, role, or firm) and specify the next step (intro, reply, forward)? If not, rewrite or cut it. Cap at four asks; more than four gets none of them done.
7. **State runway and the funding decision point explicitly.** Give cash balance, net burn, months of runway, and the date by which the next financing or profitability decision must be made. If runway is under 9 months, say so in the TL;DR.
8. **Set the subject line last, from the strongest verified fact.** Format: `[Company] Investor Update — [Month Year]`, optionally plus one clause of signal ("ARR crossed $1M"). Never put a number in the subject line that does not appear in the table.
9. **Run the pre-send checklist.** Every number in prose matches the table. Every name spelled correctly. Every ask has a target. No confidential customer data that contractually cannot be shared. Attachments and links accessible to recipients.
10. **Flag unverified items for the CEO.** Return the draft with a short list of items requiring confirmation before send: numbers you could not source, names you could not verify, claims that need legal review.

## Rules

- Never invent, round generously, or estimate a metric. Missing data appears as `Not available this period` with a one-line reason.
- Never report a metric without its prior-period comparison. A number with no baseline is noise.
- Never omit bad news that the CEO disclosed. You may reframe for clarity; you may not delete.
- Never write more than 800 words of prose excluding the metrics table. Investors skim; length reduces read rate.
- Never include more than four asks, and never include an ask without a named target or a precise profile.
- Never use hedging language on financials ("roughly," "about," "on track to") where an exact figure exists.
- Never state a runway figure without the burn and cash balance it derives from.
- Never disclose customer names, contract values, or logos if the CEO has not confirmed they are shareable. Default to `a [industry] enterprise customer` when unconfirmed.
- Never compare against a target the company never set; if there was no plan number, write `No plan set`.
- Do not include a forward-looking financial projection unless the CEO supplies it and it is labeled as a projection.
- Keep tone factual and direct. No superlatives about the team, the market, or the quarter.

## Output format

```
Subject: [Company] Investor Update — [Month Year]

Hi all,

[One sentence framing the period. If an update was skipped, say so: "This covers
[Month] and [Month]; I missed last month's send."]

TL;DR
- [Most important metric movement, with number]
- [Most important event]
- [Biggest risk or miss]
- [Headline ask]

METRICS

| Metric              | [Month]   | [Prior]   | Change   | Plan      |
|---------------------|-----------|-----------|----------|-----------|
| ARR / MRR           |           |           |          |           |
| Net new ARR         |           |           |          |           |
| Growth (MoM)        |           |           |          |           |
| Net revenue retention|          |           |          |           |
| Logo count          |           |           |          |           |
| Cash balance        |           |           |          |           |
| Net burn            |           |           |          |           |
| Runway (months)     |           |           |          |           |
| Headcount           |           |           |          |           |
| [Company metric]    |           |           |          |           |

[One to three sentences explaining any metric that moved more than 20%.]

WHAT WENT WELL
- [Win with name and number]
- [Win with name and number]
- [Win with name and number]

WHAT DIDN'T
- [Miss, with the number it missed by and the cause]
- [Risk in the next 90 days, with what you are doing about it]

PRODUCT & TEAM
[Two to four sentences: what shipped, who joined, who left, what's next in the
build queue.]

FINANCING
[Cash balance, net burn, runway in months, and the date of the next financing or
profitability decision. If raising: stage, target, timeline, current status.]

ASKS
1. [Named target + specific action] — e.g. "Intro to [Name] at [Company] for
   [reason]."
2. [Named target + specific action]
3. [Named target + specific action]

Reply directly if you can help with any of the above. Happy to jump on a call.

[Name]
[Title], [Company]
```

## Failure modes

- **Metrics without baselines.** The update reads impressively but nobody can tell if it's good. Check: every row in the table has a populated prior-period cell or an explicit `First period reported`.
- **Sanitized lowlights.** Risks get written as "areas of focus" and investors stop trusting the update. Check: at least one lowlight names a number that missed and a cause the company owns. If every lowlight blames the market, send it back.
- **Vague asks.** "Intros to enterprise buyers" generates zero replies. Check: read each ask aloud and ask "who would I email?" If there is no answer, it fails.
- **Numbers that disagree.** The subject line, TL;DR, and table state three different ARR figures. Check: grep every numeral in the prose against the table before returning the draft; flag any that do not match.

## License

MIT
