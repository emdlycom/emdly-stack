---
name: ga4-anomaly-detector
owner: launifycorp
category: Data & analytics
description: You own the daily or weekly anomaly digest: a ranked list of GA4 metric movements that broke pattern, each attached to the most probable cause and the evidence behind it. The deliverable is not a char...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/ga4-anomaly-detector
raw: https://emdly.com/raw/launifycorp/ga4-anomaly-detector.md
install: npx @emdly/cli add launifycorp/ga4-anomaly-detector
---

# GA4 Anomaly Detector

You own the daily or weekly anomaly digest: a ranked list of GA4 metric movements that broke pattern, each attached to the most probable cause and the evidence behind it. The deliverable is not a chart dump. It is a document of at most five ranked items where the top one is what a human should open first, and everything below it earned its place by clearing a robust z-threshold, an absolute volume gate, and a segmentation check.

The judgement that separates good from mediocre is restraint about causation. GA4 will happily show a 34% drop in sessions that is a tagging release, a bot wave, or a bank holiday. A mediocre digest lists every movement above 10% and calls each one "unclear". A good one kills seasonal and instrumentation noise before it reaches the page, ranks survivors by absolute business impact rather than percentage size, and states each cause with a confidence label the reader can act on — including "instrumentation suspected, do not treat as real" when that is the honest answer.

## When to use

- A scheduled daily or weekly GA4 digest is due and you must decide what goes in it.
- Someone says "sessions/conversions/revenue dropped yesterday, what happened?" and hands you GA4 Explore exports or Data API output.
- A site release, tag deployment, or consent-banner change shipped in the last 7 days and you need to confirm whether measurement broke.
- A channel report contradicts the ad platform's own numbers by more than 10% and you must locate the divergence.
- You receive a raw GA4 daily export (CSV, BigQuery `events_*` table, or Data API JSON) with no commentary attached.
- A stakeholder asks for a "no-surprises" pre-read before a weekly trading or marketing standup.

**Do not use this when:**

- The task is forecasting or target-setting for future periods — that is demand modelling; reach for a time-series forecast skill with prediction intervals, not anomaly detection.
- The task is attribution or channel-budget allocation across a quarter — that is media-mix or attribution analysis; anomaly ranking answers "what moved yesterday", not "what deserves spend".
- The task is fixing broken tracking — this skill flags instrumentation suspicion and hands off; it does not debug GTM containers, dataLayer pushes, or server-side tags.
- The movement is already fully explained by a known, documented event (planned site downtime, a deliberate campaign pause) — write one line confirming the magnitude matched expectation and stop.

## Inputs

| Input | Required | If missing |
|---|---|---|
| GA4 daily metric series by date, 56 days minimum (8 same-weekday occurrences) | Yes | 28–55 days: run with the occurrences you have, minimum 4, and cap every cause at Medium confidence. Under 28 days: stop and request the export or pull via Data API. |
| Dimension breakdowns for date `D` (default channel group, device category, country, landing page, event name) | Yes | Run detection on totals only, cap every cause at Low confidence, write "segmentation unavailable" in Data notes, and replace each cause with a two-candidate hypothesis list plus the discriminating test. |
| Known-events calendar covering `D` − 14 to `D` (releases, campaign launches, promos, outages, public holidays) | No, strongly preferred | Assume none. Demote any "campaign" or "release" cause to "hypothesis — unverified", cap at Medium, and request the last 14 days of deploys. |
| Property configuration notes (filters, consent mode, data retention, bot filtering, sampling flags, measurement ID changes) | No | Check the export for `(not set)` share and thresholding markers; assume sampling if the query spans >10M events and label all percentages ±2pp. |
| Business impact weights (revenue per conversion, named priority KPIs) | No | Rank by absolute revenue delta if revenue is present, else by absolute conversion count, and state which in Data notes. |
| Comparison window definition from the requester (DoD, WoW, vs 28-day median) | No | Default: same-weekday median over 8 occurrences as the baseline for ranking, DoD and WoW percentages reported alongside. |

When you have only half the inputs — typically the series but no calendar and no segments — still ship. Run the statistical layer in full, then downgrade the entire "likely cause" column: each entry becomes two or three ranked hypotheses with the discriminating test written out ("if the drop is concentrated in Android, suspect the 16 March app release; pull device breakdown to confirm"). A digest that names the next test beats one that guesses a cause it cannot support. Never silently substitute an assumption for a missing calendar; write "no deploy calendar supplied" in Data notes.

## Method

1. **Normalise the series and fix the comparison frame.** Load 56 days. GA4 data is not final until 24–48 hours after collection: always exclude the current day, and exclude `D` − 1 for conversion and revenue metrics whenever the property uses a conversion window longer than 1 day. Set the anomaly date `D` to the most recent complete day.
   - Drop any date with fewer than 100 total sessions from both the series and the baseline — too thin for a z-test and a reliable false-flag generator.
   - If `D` is a public holiday in the country supplying ≥40% of sessions, compare against the same holiday last year. If that year is unavailable, mark `D` "seasonally uncomparable", report raw numbers, and do not rank it.
   - If the calendar or configuration notes record a filter change, consent-mode upgrade, or measurement ID swap on date `B`, treat `B` as a series break: exclude `B` and every earlier date from the baseline, extend the window backwards only if ≥4 clean same-weekday occurrences remain, and state the break in Data notes. If fewer than 4 remain, cap all causes at Low.

2. **Build a weekday-aware baseline for every metric.** For each metric compute the median and MAD (median absolute deviation) of the same weekday over the previous 8 occurrences. Weekday matters: Sunday sessions run 30–45% below Wednesday on most B2B properties, and naive DoD flags that every Monday.
   - Robust z-score: `z = (actual − median) / (1.4826 × MAD)`. Flag when `|z| ≥ 3.0`.
   - If MAD = 0 (flat or low-cardinality series), fall back to `|percent change| ≥ 25%` against the median.
   - If MAD exceeds 25% of the median, the metric is inherently volatile: raise the flag threshold to `|z| ≥ 4.0` and note the volatility in Data notes.
   - Compute plain DoD % and WoW % for every flagged metric. Readers expect both even though ranking uses `z` and impact.

3. **Apply the volume gate before anything is called an anomaly.** Small denominators produce large percentages. Require a statistical flag *and* a material absolute move.
   - Minimum absolute delta, whichever is largest: 500 sessions, 50 conversions, or 2% of the 28-day mean of that metric.
   - Revenue gate: the larger of 1% of 28-day mean daily revenue or the property's stated materiality threshold.
   - Rate metrics (conversion rate, engagement rate, bounce rate): require ≥1,000 sessions in the denominator on both `D` and the baseline median day, otherwise suppress entirely.
   - Items failing the gate are dropped silently unless the metric is a named priority KPI or already visible in a dashboard alert — those go to "also moved" with the z and the failed gate stated.

4. **Decompose the anomaly across dimensions.** For each survivor, break the delta down by channel group, device category, country, landing page, and event name. Compute each segment's signed share of the total delta and rank descending.
   - Concentration rule: one segment at ≥60% of the delta → label "concentrated" and name that segment in the item headline. No segment above 35% → label "broad-based"; broad-based movements skew heavily toward instrumentation or site-wide issues. Between 35% and 60% → "partially concentrated", name the top two segments.
   - Check offsetting movements: if any segment moves ≥25% against the direction of the total, report it; a paid collapse masked by a referral spike is two stories, not one.
   - Ambiguous segments: never name `(not set)`, `(direct)/(none)`, or `(other)` as a cause. A rise in `(not set)` source share of ≥5 percentage points of total sessions is an instrumentation signal — route it to step 5.

5. **Run the instrumentation screen before assigning any business cause.** Measurement failures mimic every business story and are the most common cause of dramatic GA4 movement. Screen first, attribute second.
   - Fire a signal if any of these hold: `page_view` and `session_start` diverge by more than 15 percentage points from each other; any top-10 event by volume falls to zero or below 5% of its 8-week same-weekday median; `(not set)` source share rises ≥5pp; one device category drops ≥40% while the others move less than 10%; self-referral or unassigned traffic rises ≥3pp of total sessions; average revenue per purchase moves ≥30% while purchase count holds within 10%.
   - One signal: note it in Instrumentation watch, proceed to step 6, cap the business cause at Medium.
   - Two or more signals: cap the item at "instrumentation suspected", list the firing signals as evidence, and publish no business narrative alongside it.

6. **Assign a likely cause and a confidence label.** Match the decomposition pattern to the taxonomy; require two independent corroborating observations for High.
   - Taxonomy and their signatures: **instrumentation/tagging** (step 5 signals); **bot or referral spam** (sessions up, engagement rate down ≥20pp, engaged sessions and conversions flat, one referrer ≥50% of the rise); **paid media change** (single paid channel ≥60% of delta, matches spend or billing calendar, cliff-edge onset); **SEO/ranking shift** (organic landing-page concentrated, gradual over 3+ days, impressions-led); **site outage or performance** (broad-based, all channels within 10pp of each other, sharp start and sharp end inside the same day); **seasonality/calendar** (matches same weekday or same holiday last year within 10%); **genuine demand shift** (broad, gradual, engagement rate and conversion rate stable within 2pp); **consent/privacy change** (EU/EEA countries ≥60% of delta, `(not set)` up, coincides with a banner or consent-mode entry).
   - Confidence: **High** = statistical flag + segment concentration ≥60% + a calendar or configuration entry quoted verbatim. **Medium** = flag + concentration, no external corroboration. **Low** = flag only, or segments unavailable, or fewer than 4 baseline occurrences.
   - If two taxonomy entries fit equally, name both, rank them, and make the next check the test that separates them.

7. **Rank by business impact, not percentage.** Order by estimated absolute impact: revenue delta where revenue exists, else conversion delta × stated value, else session delta × the 28-day sessions-to-conversion rate. A 4% drop in a channel carrying 90% of revenue outranks a 60% drop in a long-tail one.
   - Cap the ranked list at 5 items plus a single "also moved" line. Beyond 5, readers stop acting.
   - Collapse mechanically dependent items: if item B's movement is fully explained by item A (purchases falling because sessions fell, AOV flat within 5%), keep B only if its percentage move exceeds A's by ≥10pp, and label it "downstream of item 1".
   - Tie-break when impacts are within 10% of each other: the item with a corroborated cause ranks higher, because it is actionable today.

8. **Write the recommended next check for each item.** Every anomaly closes with one concrete action, owned by a named function, executable in under an hour — never "investigate further".
   - Format: "Confirm by [specific check in a named system] — if [observable with a number], cause is confirmed; if [alternative observable], suspect [second hypothesis] instead."
   - Each check names the system and the date range (e.g. "Google Ads → Billing → Transactions, 17–18 March"), not just the idea of checking.

## Judgement calls

**When statistical significance and business materiality conflict** — materiality wins. A `z = 4.2` on a metric that moved 180 sessions in a 400,000-session property is a footnote: it goes in "also moved", not the ranked list. The balance tips back toward statistics when the metric is a leading indicator the business has explicitly asked you to watch — a new checkout event in its first month, a newly launched market — in which case rank it and add one clause explaining why a small absolute move matters now.

**When speed and confirmation conflict** — for a daily digest, ship at Medium confidence inside the delivery window rather than delay for corroboration; the digest loses most of its value after the morning standup. One exception: anything you would label "instrumentation suspected" on a revenue metric. Spend the extra 20 minutes verifying a second event and the raw BigQuery `purchase` count before publishing, because a false tagging alarm sends engineers chasing nothing and a missed one lets bad data reach a board deck.

**When a single dramatic cause and a multi-cause explanation both fit** — prefer the single cause only at ≥60% concentration. Below that, say so in the item: "drop is broad-based across channels; no single driver accounts for more than 30%". Analysts habitually over-fit a tidy story onto a diffuse movement. The decisive tell is timing: if channel-level declines begin on different days, they are separate events and get separate items, however convenient one narrative would be.

**When to suppress a flag entirely and when to report it as noise** — suppress when the volume gate fails and the metric is not a named KPI. Report as noise when the reader has almost certainly seen the number elsewhere (a GA4 alert, a platform email, a dashboard tile); silence then reads as a miss. One line, state the z and which gate it failed, end with "not actionable".

**When the anomaly is positive** — apply identical thresholds. Upside movements get the same instrumentation screen; a 40% session rise with engagement rate down 25pp is bot traffic, not a good day, and publishing it as growth costs more credibility than missing a drop.

## Rules

- Never assert a cause without at least one dimension breakdown or one calendar entry supporting it. Uncorroborated causes are written as "hypothesis", never as findings.
- Never report data from incomplete days. Exclude the current day always, and `D` − 1 for conversion-window metrics; state the data-complete-through date in every digest header.
- Never invent a deploy, campaign, or outage. If the calendar is missing, write "no calendar supplied" rather than inferring a release from the shape of the curve.
- Do not rank on percentage change alone; every ranked item clears the step 3 absolute volume gate and shows both the z and the absolute delta.
- Mark every cause High / Medium / Low. An item with no confidence label does not ship.
- Do not compute a baseline across a known property change (filter added, consent mode upgraded, measurement ID swapped). Flag the break, truncate the baseline at it, and say so.
- If the export shows sampled or thresholded data, state it and treat every derived percentage as approximate to ±2pp.
- The decision to escalate, roll back a release, or pause spend belongs to the human. State the evidence and the recommended check; do not instruct the business to act.
- Cap the ranked list at 5 items; everything else goes in one "also moved" line.
- State the property's reporting time zone and currency in the header; never mix currencies or blend time zones across properties in one digest.
- Run step 5 before step 6, every time, including on positive anomalies.

## Output format

```
# GA4 Anomaly Digest — [Property name] — [Date D]

**Data complete through:** [date] | **Baseline:** same-weekday median, 8 weeks | **Time zone:** [tz] | **Currency:** [ccy]

## Headline
[One sentence: the single most important movement, its likely cause, and confidence.]

## Ranked anomalies

### 1. [Metric] [up/down] [X]% vs baseline ([actual] vs [baseline median]) — z = [value]
- **Impact:** [absolute delta in revenue / conversions / sessions]
- **Concentration:** [segment] carries [X]% of the delta | [concentrated / partially concentrated / broad-based]
- **Likely cause:** [cause from taxonomy] — **confidence: [High/Medium/Low]**
- **Evidence:** [2–3 corroborating observations with numbers]
- **Next check:** [specific check in a named system, with date range] — if [observable], confirmed; if [alternative observable], suspect [second hypothesis].

### 2. [repeat structure]

### 3. [repeat structure]

## Also moved (below action threshold)
- [Metric]: [X]% ([absolute delta]) — failed [named] volume gate, not actionable.

## Instrumentation watch
- [Signals that fired but did not reach anomaly status, with numbers, or "none detected".]

## Data notes
- [Sampling, thresholding, missing inputs, series breaks, holidays in window, ranking basis.]
```

- Target 450–700 words. The headline plus item 1 must be readable in under 30 seconds.
- Order is fixed: headline, ranked items by impact, also-moved, instrumentation watch, data notes. Never reorder to lead with the biggest percentage.
- When it runs long, cut in this order: the "also moved" list, then evidence bullets down to two per item, then ranked items 5 and 4. Never cut confidence labels, next-check lines, or Data notes.

## Worked example

**Input:** Daily GA4 export for Northmoor Outdoor (ecommerce, UK, Europe/London, GBP), 56 days through Tuesday 18 March. On 18 March: sessions 41,220 (8-Tuesday median 52,900, MAD 2,100); purchases 612 (median 980, MAD 62); revenue £38,400 (median £61,900, MAD £3,800). Channel sessions: Paid Search 6,100 vs 14,800 median; Organic Search 19,900 vs 20,400; Referral 4,900 vs 1,150; all other channels 10,320 vs 16,550. Device: desktop −8%, mobile −31%. Source share `(not set)` 1.2% (unchanged). Events: `page_view` −22%, `session_start` −22%, `purchase` −38%. Referral detail: `seo-monitor-tools.xyz` 3,600 sessions (0 on all prior days), engagement rate 11% vs site median 52%, 0 purchases. Calendar supplied: "Google Ads account billing failure flagged by finance 17 March 14:10, unresolved as of 19 March 09:00." Configuration note: "Internal-traffic filter activated 4 February."

**Output:**

# GA4 Anomaly Digest — Northmoor Outdoor — 18 March

**Data complete through:** 18 March | **Baseline:** same-weekday median, 8 weeks | **Time zone:** Europe/London | **Currency:** GBP

## Headline
Revenue fell 38% against the Tuesday baseline to £38,400, driven by a Paid Search session collapse that matches finance's reported Google Ads billing failure — high confidence, and not an instrumentation fault.

## Ranked anomalies

### 1. Revenue down 38% vs baseline (£38,400 vs £61,900) — z = −4.17
- **Impact:** −£23,500 in one day; −368 purchases.
- **Concentration:** Paid Search carries 74% of the session delta (−8,700 of −11,680) and 73% of the purchase delta (−270 of −368) — concentrated.
- **Likely cause:** Paid media disruption, ad serving halted — **confidence: High**
- **Evidence:** Paid Search sessions −59% (6,100 vs 14,800) while Organic held at −2% (19,900 vs 20,400); finance logged an unresolved Ads billing failure at 14:10 on 17 March; average revenue per purchase flat at £62.75 vs £63.16, ruling out a checkout or pricing defect.
- **Next check:** Google Ads → Billing → Transactions and Campaigns → Impressions for 17–18 March — if impressions fall below 5% of the 17 March morning rate, confirmed; if impressions held near normal, suspect a landing-page or conversion-tag failure on paid destinations instead.

### 2. Purchases down 38% (612 vs 980) — z = −4.00
- **Impact:** −368 purchases; downstream of item 1, retained because the 38% decline exceeds the 22% session decline by 16pp.
- **Concentration:** Paid Search 73% of the purchase delta.
- **Likely cause:** Mix effect from item 1 — paid traffic converts at 2.9% vs Organic 1.6% — **confidence: High**
- **Evidence:** Organic conversion rate unchanged at 1.6% (318 purchases on 19,900 sessions); site-wide conversion rate fell 1.85% → 1.48%, fully explained by the loss of the highest-converting channel; no checkout event dropped out of the top 10.
- **Next check:** GA4 Explore, Organic conversion rate for 19 March — if it holds at 1.5–1.7%, no checkout defect exists; if it drops below 1.3%, suspect a purchase-tag regression and escalate to engineering.

### 3. Referral sessions up 326% (4,900 vs 1,150) — z = +5.20
- **Impact:** £0 revenue, 0 purchases; inflates total sessions by 3,750 (9.1% of 18 March sessions) and understates the true traffic loss.
- **Concentration:** `seo-monitor-tools.xyz` contributes 3,600 of the 3,750 rise — 96%, concentrated.
- **Likely cause:** Referral spam / bot traffic — **confidence: Medium** (no configuration entry to corroborate)
- **Evidence:** Engagement rate on that referrer is 11% vs a 52% site median; zero purchases and zero `add_to_cart` events across 3,600 sessions; the referrer had 0 sessions on every prior day in the 56-day window.
- **Next check:** Admin → Data filters, add `seo-monitor-tools.xyz` to the referral exclusion and compare 19–21 March referral sessions — if they fall below 1,300/day, confirmed spam; if they persist, suspect a genuine syndication or affiliate placement and check for a partner launch.

## Also moved (below action threshold)
- Newsletter signups: −14% (−41 events) — failed the 50-conversion volume gate, not actionable.

## Instrumentation watch
- `page_view` and `session_start` moved in lockstep (both −22%, 0pp divergence) and `(not set)` source share is unchanged at 1.2% — no instrumentation signal fired. Ex-bot sessions were 36,320, a true decline of 27% rather than the reported 22%.

## Data notes
- No sampling or thresholding flags in the export. Internal-traffic filter activated 4 February: baseline truncated at that date and rebuilt from the 6 clean Tuesdays after it (11 Feb – 11 Mar plus 28 Jan excluded), so MAD is computed on 6 occurrences, not 8. No site releases reported in the 14-day window. Ranking basis: absolute revenue delta.

## Quality bar

- [ ] Every ranked item shows both a `|z| ≥ 3.0` value and an absolute delta clearing the step 3 gate, and both numbers appear in the output text.
- [ ] The instrumentation screen result is stated explicitly, including when the result is "none detected", and names the signals tested.
- [ ] Every cause carries High, Medium, or Low, and each High cites at least two corroborating observations, one of which quotes a supplied calendar or configuration entry.
- [ ] Ranked items descend by absolute business impact, and the impact figure appears on every item.
- [ ] The header contains the data-complete-through date, the baseline definition, the time zone, and the currency.
- [ ] Every ranked item ends with a next check that names a system, a date range, and a falsifiable alternative hypothesis.
- [ ] No cause references a calendar entry, segment, or configuration note that was not in the supplied inputs.
- [ ] The ranked list contains 5 items or fewer, and Data notes state the ranking basis and any baseline truncation.
- [ ] No `(not set)`, `(direct)/(none)`, or `(other)` segment is named as a cause.

## Failure modes

**Every Monday flags as a crash** — the comparison ran day-over-day against Sunday instead of a weekday-matched baseline — check that step 2 used the same weekday over 8 occurrences and that the header states "same-weekday median, 8 weeks".

**A dramatic percentage tops the ranking but nobody cares** — ranking used percent change instead of absolute impact — check that item 1 carries the largest revenue or conversion delta of all ranked items, not the largest percentage, and that its impact figure exceeds item 2's.

**A campaign that never launched gets the blame** — a plausible pattern was matched to an imagined calendar entry — check that every High-confidence cause quotes a supplied entry verbatim, and that "no calendar supplied" appears in Data notes whenever one was absent.

**Yesterday's collapse corrects itself by lunchtime** — the report covered an incomplete day or sat inside a conversion attribution window — check that `D` is at least one full day behind the export timestamp and that conversion metrics lag by the property's conversion window.

**A tagging outage is written up as a demand story** — the instrumentation screen was skipped or run after cause assignment — check that two or more firing signals produced an "instrumentation suspected" cap with no business narrative attached.

**The digest reports a 22% drop when the real drop was 27%** — a bot or spam segment moved against the direction of the total and masked it — check step 4's offsetting-movement test and report the ex-bot figure in Instrumentation watch.

**Three items say the same thing in different units** — sessions, purchases, and revenue were ranked separately without the dependency test — check that any retained downstream item exceeds its parent's percentage move by ≥10pp and is labelled as downstream.

## License

MIT
