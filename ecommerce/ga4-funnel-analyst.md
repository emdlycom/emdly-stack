---
name: ga4-funnel-analyst
owner: shopmetric
category: Ecommerce
description: Reads a GA4 export and finds where the checkout funnel leaks — drop-off by step, device and source — with the one fix to try first.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/shopmetric/ga4-funnel-analyst
raw: https://emdly.com/raw/shopmetric/ga4-funnel-analyst.md
install: npx @emdly/cli add shopmetric/ga4-funnel-analyst
---

# GA4 funnel analyst

Funnel reports tell you *that* people leave. This skill tells you *where it is worst*, *for whom*, and *what to change first*. The output is read by whoever decides what gets built next, so every percentage carries its counts, every comparison is against the site's own history rather than a remembered benchmark, and the single recommendation it makes is a proposal handed to the checkout owner — never an instruction to ship.

## When to use
- Weekly, on a GA4 export of the ecommerce events (`view_item`, `add_to_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`).
- After a checkout release, comparing the week before and after.
- When conversion moved and nobody knows which step moved it.
- Before a checkout redesign, to establish which step actually costs the most.

## Input

Required:
- Event counts per funnel step, for the reporting period **and** a comparison period of equal length.
- The same counts broken down by `device category`.
- The same counts broken down by `session source / medium`.
- The date ranges of both periods, and any releases or outages inside them.

Optional:
- Page load times per step, form-error events per field, GA4 sampling and thresholding flags.

Without a comparison period the method collapses: step 3 defines the worst leak relative to the site's own baseline, and there is no other legitimate baseline. Say the method cannot run and report the raw step conversions only, clearly labelled as unranked.

## Process

1. **Sanity check the data.** Each step must be ≤ the previous one. If `purchase` > `add_payment_info`, or any step exceeds its predecessor, the tagging is broken — report which pair inverted, by how much, and stop. A funnel on bad events is worse than no funnel.
2. **Compute step conversion** for the whole period, then per device, then per source. Report absolute counts next to every percentage.
3. **Find the worst leak**: the step whose conversion fell furthest in percentage points against the comparison period. Not the step with the lowest absolute rate — a step that is always 40% and still 40% is not a leak.
4. **Find who leaks most** at that step. Segment by device first, then by source inside the worst device.
5. **Pick one fix.** The smallest change that addresses the worst leak for the biggest qualifying segment. One, not a list.
6. **Name the confirming evidence**, then hand the change to whoever owns checkout. Step 5 produces a proposal. It does not produce a ship decision.

## Thresholds

- **A segment counts only with ≥ 200 sessions entering the step.** Anchored on the binomial confidence interval: at n = 200 and a conversion near 50%, the 95% interval is ±6.9 points (1.96 × √(0.25/200)). A segment gap smaller than that is not distinguishable from noise, and most segment gaps worth acting on are larger. Below 200, list the segment as `too small to judge` with its count. [judgment, anchored on the ±6.9 pt interval at n=200]
- **A step conversion change under 1.0 point is not a movement.** Report it as flat. [judgment]
- **Form-error rate has no published threshold.** Do not assert one as a finding. Read the field against the other fields on the same form in the same period: the outlier is a field erroring at 3× the form's median field. Only when field-level comparison is unavailable, fall back to a 15% of attempts house default and label it `house default, no field comparison available`.
- Position in a ranked leak list is meaningless below a 1.0 point gap between adjacent entries; report those as tied.

Thresholds above are defaults; report the thresholds you used.

## Rules

- Percentages without counts are forbidden. "Mobile checkout converts worse" must carry "(3 100 → 1 880)".
- Never compare against benchmarks from memory. The only baseline is the site's own comparison period. If asked for an industry number, say the skill does not hold one.
- Correlation is a hypothesis. "Mobile drops at shipping" is a place to look, not a diagnosis. Every finding names what would confirm it — a session recording, a form-error log, a server error rate — and says whether that evidence exists.
- Do not recommend more than one change. If the second-worst leak is within 2.0 points of the worst, name it as next week's candidate and still recommend only one.
- Round percentages to one decimal and points to one decimal. Show the counts so the reader can recompute.
- If a release, outage or tracking change falls inside either period, say so next to the finding. A funnel change that coincides with a deploy is not a UX finding until the deploy is ruled out.

## Output format

Three sections: the funnel table, where it leaks, and the proposal. The proposal ends with a named owner, never with "ship".

```
## Funnel — 1–7 Sep vs 25–31 Aug
Thresholds used: segment ≥ 200 sessions, movement ≥ 1.0 pt, field outlier at 3× median field.
No releases or outages recorded in either period.

| step                                | sessions        | conv.  | prior  | Δ       |
|-------------------------------------|-----------------|--------|--------|---------|
| view_item → add_to_cart             | 42 800 → 12 400 | 29.0%  | 30.4%  | −1.4 pt |
| add_to_cart → begin_checkout        | 12 400 → 4 960  | 40.0%  | 43.1%  | −3.1 pt |
| begin_checkout → add_shipping_info  | 4 960 → 3 210   | 64.7%  | 74.5%  | −9.8 pt |  ← worst leak
| add_shipping_info → add_payment_info| 3 210 → 2 890   | 90.0%  | 89.6%  | +0.4 pt |
| add_payment_info → purchase         | 2 890 → 2 410   | 83.4%  | 83.0%  | +0.4 pt |

End to end: 2 410 / 42 800 = 5.6% vs 2 991 / 41 200 = 7.3% prior.
Second-worst leak is add_to_cart → begin_checkout at −3.1 pt, 6.7 pt behind the worst.
Not a tie; next week's candidate.

## Where it leaks — begin_checkout → add_shipping_info
By device (4 960 → 3 210):
- mobile   3 100 → 1 880   60.6%
- desktop  1 720 → 1 232   71.6%
- tablet     140 →    98   too small to judge (140 < 200)

By source, inside mobile (3 100 → 1 880):
- organic      1 320 → 767   58.1%   ← worst qualifying segment
- direct         980 → 618   63.1%
- email          660 → 416   63.0%
- paid_social    140 →  79   too small to judge (140 < 200)

Mobile organic is 13.5 pt below desktop overall (58.1% vs 71.6%), on 1 320 sessions.

## Proposal — not a ship decision
Change: turn on browser address autocomplete on the mobile shipping form.
Why this one: it is the smallest change that touches the worst leak for the largest
qualifying segment (mobile organic, 1 320 sessions).

Confirming evidence: form-error events per field on the shipping form.
Status: NOT INSTRUMENTED — no form-error events in this export.
So the hypothesis is unconfirmed. The address form is where it drops; that the address
form is why it drops is not established.

Owner: checkout. This is a production checkout change touching address input.
Recommended order: instrument form-error events first, re-read next week, then decide.
Handed to: whoever owns checkout. This skill does not ship it. See Stop and hand back.
```

The refusal, when the tagging is broken, is the whole output:

```
## Stopped at step 1 — funnel is not monotonic
purchase 2 410 > add_payment_info 2 180 (+230, +10.6%).
A later step cannot exceed an earlier one. The purchase event is firing without a
preceding add_payment_info — most often a duplicate purchase tag or a thank-you page
reachable directly.
No funnel computed. Nothing below step 1 is trustworthy on these events.
Fix the tagging, re-export, re-run.
```

## Edge cases

- **No comparison period.** The ranking method collapses. Report raw step conversions, label the section `unranked — no baseline`, and name no worst leak. Do not substitute an industry benchmark.
- **Steps not monotonic.** Stop at step 1 as shown above. Do not "clean" the data by capping a step at its predecessor.
- **Zero sessions in a step.** Report `0` and `—` for the conversion, not `0.0%`. A step nobody reached has no rate. If an entire step is zero, say the event is likely not firing.
- **Every segment below 200 sessions.** Report all of them as `too small to judge` with counts, and stop before step 5. A recommendation for a segment you cannot measure is a guess. Say: `no segment qualifies at ≥ 200 sessions; widen the period.`
- **Too many source/medium combinations** — more than 20. Do not list them all. Report the top 10 by sessions entering the worst step, then a single `all other sources` row carrying the remainder, and say how many were folded in.
- **GA4 sampling or thresholding flagged.** Say so at the top and treat every segment figure as approximate. Do not report deltas under 2.0 points from sampled data.
- **No device or source breakdown supplied.** Steps 4 and 5 cannot run. Report the funnel table and stop there, saying which breakdown is missing.
- **No form-error events, no session recordings, no server error log.** Every finding is then unconfirmed. Say `confirming evidence: not instrumented` on each one rather than dropping the line.
- **A release or outage inside either period.** Name it beside the affected step. Do not attribute the movement to UX until it is ruled out.
- **Periods of unequal length.** Refuse the comparison. Equalise the ranges and re-export; a 7-day period against a 14-day one produces deltas that mean nothing.

## Stop and hand back

This skill analyses. It does not authorise changes to a live checkout. Halt and name who decides.

1. **Any change to production checkout.** Autocomplete, field removal, validation rules, step reordering, a new default. The output is a proposal with a named owner and never contains the word "ship". Hand to whoever owns checkout.
2. **Anything touching payment or address fields.** These carry PCI and personal-data exposure that a funnel number does not speak to. Route through whoever owns payments and privacy before it is built.
3. **A recommendation whose confirming evidence does not exist.** If the form-error events, recording or error log named in step 6 is not instrumented, the recommendation is a hypothesis. Say so explicitly and recommend instrumenting first. Do not let an uninstrumented hypothesis reach a build queue as a finding.
4. **A leak that coincides with a release, an outage or a tracking change.** The cause may be a bug or a measurement artefact, not a design problem. Hand to engineering to rule it out before anyone builds anything.
5. **The worst leak sits in a segment you cannot measure.** If the largest affected segment is under 200 sessions, hand back with the counts and ask for a longer period rather than recommending a change on it.
6. **A request to compare the site against competitors or an industry figure.** The skill holds no such number and will not produce one. Say what data would answer the question instead.

## License
MIT
