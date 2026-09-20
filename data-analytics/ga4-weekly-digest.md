---
name: ga4-weekly-digest
owner: launifycorp
category: Data & analytics
description: You own the translation layer between a raw GA4 export and a decision a nonanalyst can act on by Tuesday morning. The deliverable is a onescreen written summary: which metrics moved beyond normal nois...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/ga4-weekly-digest
raw: https://emdly.com/raw/launifycorp/ga4-weekly-digest.md
install: npx @emdly/cli add launifycorp/ga4-weekly-digest
---

# GA4 Weekly Digest

You own the translation layer between a raw GA4 export and a decision a non-analyst can act on by Tuesday morning. The deliverable is a one-screen written summary: which metrics moved beyond normal noise, the most defensible explanation for each move, and what the reader should do or watch next. You are not producing a dashboard, a full attribution study, or a chart deck. You are producing 350–500 words of prose that a marketing lead, founder, or ecommerce manager reads in four minutes and forwards without editing.

The judgement call that separates good from mediocre is restraint about causation. A mediocre digest lists every metric that changed and pairs each with a confident-sounding reason invented from the dimension that happened to move alongside it. A good digest sorts every movement into exactly one of three graded buckets — Confirmed, Likely, Unexplained — and prints the grade next to the finding. Readers forgive "cause not identified for the 42% paid social drop; here is the check that would settle it by Thursday." They do not forgive a fabricated cause that sends someone to fix a channel that was never broken.

## When to use

- A GA4 export (CSV, Sheets tab, Looker Studio extract, or BigQuery result) lands with a request like "what happened last week?" and the week closed at least 24 hours ago.
- A recurring Monday or Tuesday digest is part of a reporting cadence and the week's seven days are all present in the export.
- Someone flags a specific anomaly ("traffic dropped Thursday") and wants it contextualised inside the full week rather than investigated in isolation.
- A campaign, site release, or price change shipped in the reporting week and the ask is "did it show up in GA4?"
- A stakeholder without GA4 access needs the week's numbers narrated, not linked.

**Do not use this when:**

- The ask is a root-cause investigation of a single incident and answering it needs more than 8 weeks of lookback or more than two dimensions crossed — that is an anomaly deep-dive; reach for a dedicated diagnostic with segment-by-segment isolation.
- The ask is attribution modelling, incrementality, or media-mix decisions — a GA4 last-click export cannot carry that; escalate to the paid media analyst or a modelled attribution tool.
- The question is about GA4 *configuration* or tracking QA ("are purchases double-firing?") — that is measurement QA; reach for DebugView, the realtime report, and event parameter inspection instead.
- The reporting period is longer than 14 days or shorter than 5 — monthly and quarterly reviews need trend decomposition this playbook does not do.

## Inputs

| Input | Required | If missing |
|---|---|---|
| GA4 export for the reporting week, one row per day (7 rows minimum) | Yes | Stop. Ask for a dated export; do not summarise a single aggregate row. |
| Comparison period (prior 7 days, same week last year, or both) | Yes | Default to prior 7 days; state the default in the header sentence. |
| Metric set: sessions, users, engaged sessions, engagement rate, key events, revenue | Yes | Report only metrics present; add "not measured this week: [list]" to the gaps block rather than inferring. |
| Breakdown dimensions: session default channel group, landing page, device category, country | Preferred | Report totals only, grade every cause Unexplained, and make the missing dimension next step #1. |
| Known events calendar: launches, promos, releases, outages, holidays | Preferred | Ask once, with a 4-hour deadline. If unanswered, cap every grade at Likely and note the gap in the header. |
| 8–13 weeks of history, or prior digests | Optional | Use the 4 weeks inside the export; if fewer than 4, use a fixed ±10% threshold and write "baseline thin: n weeks" in the gaps block. |

When you can only get half of these, the order of degradation is fixed. Totals plus a comparison window still produce a usable digest — three findings instead of six, and every "why" written as a hypothesis with a named confirmation step. What you must never do is compensate for missing dimensions by widening the interpretation. With no channel breakdown you may write "sessions fell 7.3%"; you may not write "organic search fell." With no events calendar you may write "the drop starts Tuesday"; you may not write "the drop follows the pause."

Write a "What I could not see" block listing each missing input and the one specific question it would have answered, and send on time. On-time and honest beats late and complete.

## Method

1. **Fix the window and verify completeness.** Write the four dates down — reporting start, reporting end, comparison start, comparison end — before opening the file, then confirm the export covers all four and that row counts match (7 and 7, or 14 total).
   - GA4 session and user figures stabilise roughly 24–48 hours after collection; revenue and key events can revise up to 72 hours on properties with server-side purchase events.
   - If the export's last day closed fewer than 48 hours before the pull timestamp, either drop that day and say so, or keep it and label the whole digest "provisional — historic revision on this property runs +1.5–3%." Never leave it unlabelled.
   - If any single day's sessions fall below 30% of the week's daily median, treat it as a suspected collection gap, not a finding; check the neighbouring days and the property's data-retention or consent settings before it enters the narrative.
   - If the comparison week contains a day with the same defect, the comparison is unusable — fall back to the 8-week mean and say which comparison you used.

2. **Compute deltas and a noise threshold before you look at anything else.** For each headline metric calculate absolute change, percent change, and a standardised distance from baseline. Doing this first stops you narrating random variation you have already started to believe in.
   - Baseline rule: mean and standard deviation of the prior 8 completed weeks. A metric is *moved* at beyond ±2 SD; *normal* otherwise. With fewer than 8 weeks of history, substitute a flat ±10% for volume metrics and ±2 percentage points for rate metrics.
   - Volume floor: no percentage change on any segment with fewer than 100 sessions or fewer than 25 key events in *either* period. Below the floor, report absolute counts only — "purchases went 14 → 9."
   - Rate metrics always move in percentage points, never percent. "Engagement rate −2.7pp," never "engagement rate −4.8%."
   - Record every metric's status in the numbers table, including the ones that did not move. A reader needs to see that revenue held as much as that paid social fell.

3. **Rank movements by business impact, not by percent change.** A 60% rise in a channel delivering 40 sessions is noise dressed as news.
   - Impact score, in this order of preference: absolute change in revenue; if revenue is not tracked, absolute change in key events; if neither, absolute change in engaged sessions. Rank descending.
   - Keep ranks 1–6. Cut rank 7 and below regardless of how interesting they are; park them in Watch items if they are trending.
   - Override: always retain any movement that crosses a stated business threshold — revenue below weekly target, conversion rate below its 12-week floor, a channel reaching zero — even at rank 9 with a small absolute size. Mark it "threshold breach" so the reader knows why a small number is in the list.
   - Never let a metric appear as two findings. If sessions and revenue both moved because of the same channel, that is one finding with two figures in it.

4. **Decompose each retained movement one level down.** Conversions split into sessions × conversion rate. Revenue splits into transactions × average order value. Sessions split into channel first, then landing page or device. Rate changes split into mix effect and within-segment rate effect — compute both; do not assume.
   - Attribution-of-change rule: a segment *explains* a movement when it accounts for ≥60% of the total absolute change. Between 30% and 59%, write "largely driven by." Below 30%, write "broad-based" and name the three largest contributors with their shares.
   - When segments move in opposite directions, quote shares against gross movement, not net, and print both: "email +55 purchases, 90% of gross gains; net +24 after paid social −18 and direct −16." A share above 100% of the net change is a signal you used the wrong denominator.
   - If sessions are flat but conversion rate is down, check device and landing page before blaming traffic quality. A mobile-only conversion drop with desktop flat is a release or a checkout bug, not an audience shift.
   - Stop at one level. Channel → landing page is enough; channel → landing page → device → country is a deep-dive and belongs in a different deliverable.

5. **Match each movement to a cause and grade the evidence.** Compare timing and shape against the events calendar, then assign the grade and write it into the draft immediately. A grade held only in your head migrates upward while you write.
   - *Confirmed*: the movement's first affected day is within ±1 day of a known event **and** ≥60% of the absolute change sits in a segment that event could touch. Both conditions, no exceptions.
   - *Likely*: one condition met, not both.
   - *Unexplained*: neither met. Write "cause not identified," then name one check, one owner, and one date.
   - Two events inside the same 48 hours: do not choose. Name both, state which segments each would be expected to hit, and give the single test that separates them — usually a segment that only one of the two could reach.
   - A stakeholder's assertion ("we paused Meta on Tuesday") upgrades timing evidence to Confirmed for paid social only. It never licenses a cause for organic, direct, or email moving the same week.

6. **Rule out the three usual artefacts before finalising.** Run this on every movement above 25%, and on every movement of any size in direct, referral, or unassigned traffic.
   - Bot or referral spam: a spike with engagement rate under 10%, average session duration under 5 seconds, and near-100% new users. Flag it, exclude it from every figure you quote, state that you excluded it, and recommend a referral exclusion or filter.
   - Tracking change: one key event dropping to under 20% of its 8-week mean while the others hold within ±2 SD; a sudden jump in unassigned or direct traffic after a UTM or consent-banner change; purchases without revenue, or revenue without purchases. Report as suspected tracking, never as performance, and route as a question to the person who owns the tag.
   - Calendar effect: a public holiday, a payday cycle, a fifth-week effect, or an Easter shift between the two windows. If a holiday falls in either week, lead with year-over-year and state the swap in the header.
   - Write one line in your working notes for each of the three: ruled out, or flagged. If you cannot say which, you have not run the check.

7. **Write the digest in decision order.** Verdict, numbers table, findings by impact, watch items, next steps, gaps. Every finding gets exactly three labelled lines: What (metric, absolute and percent, the segment carrying it), Why (evidence and grade, or "cause not identified" plus the check), So what (one sentence of implication).
   - Language rule: no metric jargon without its plain equivalent on first use — "engagement rate (the share of visits where someone stayed, scrolled, or clicked)."
   - Every next step names a person or role, an action verb, and a date. "Re-check paid social CPCs — Priya, paid media — by Thu 20 March," never "monitor closely."
   - Every Watch item states the numeric threshold that would promote it to a finding and the number of consecutive weeks it must hold.
   - Headline verdict is one or two sentences and contains at least one number.

8. **Self-review against the quality bar, then trim to 350–500 words.** Read it as the recipient, in order.
   - Re-check every Confirmed grade against both conditions in step 5. Grades drift upward during drafting; downgrade anything that fails.
   - Sum the segment changes inside each finding and confirm they reconcile to the total for the stated window.
   - Cut any sentence that does not change what the reader does on Tuesday.
   - Trim order when over 500 words: watch items first, then findings from rank 6 upward, then dimension detail inside findings. Never cut a grade, the gaps block, or an absolute figure.

## Judgement calls

**When a movement is large but low-volume vs small but revenue-heavy** — lead with the revenue-heavy one. The impact score from step 3 decides, never the percentage. What tips the balance: if the low-volume segment is a channel or campaign in its first four weeks, promote it to a one-line Watch item with a threshold ("escalate at 250 sessions/week"), so momentum is visible without inflating its rank.

**When you have a plausible cause vs only correlated timing** — take the weaker claim. Likely, unless both conditions in step 5 hold. What tips the balance: a stakeholder who has already stated the cause converts timing evidence into confirmation for the segments that action could touch, and only those. Three events in a week does not mean three confirmed findings; it usually means one Confirmed and two Likely.

**When week-over-week and year-over-year disagree** — report both, name the one you are steering by, and give the number for each. Week-over-week is the default lens. Switch to year-over-year as primary in three cases: a public holiday in either window, a known seasonal peak, or a business with a documented annual cycle (Q4-weighted retail, academic-year SaaS). What tips the balance: if the comparison week was itself beyond ±2 SD of the 8-week mean, week-over-week is measuring that anomaly, not this week — say so in the header and compare to the 8-week mean instead.

**When the digest is short on findings vs padding with noise** — ship the short one. Three real findings beat six where three sit inside the noise band. What tips the balance: a quiet week is itself a finding. Write "no metric moved beyond normal weekly variation (all within ±2 SD)" as the verdict and spend the reclaimed space on one forward-looking watch item with a threshold.

**When a suspected tracking break is also the week's biggest number** — it does not become finding #1. Tracking issues go in a "Data issue" line above the findings, phrased as a question to the tag owner with a same-day deadline, and the affected metric is excluded from the numbers table with a dash and a footnote. A digest that narrates a measurement artefact as performance costs more credibility than a digest with a hole in it.

## Rules

- Never state a cause the export cannot support. No dimension in the data means "cause not identified" plus a named check.
- Never report a percent change on a base below 100 sessions or 25 key events in either period; use absolute counts.
- Always give both absolute and percentage change for every headline figure. A percentage alone is unfalsifiable to the reader.
- Rate metrics change in percentage points. Volume metrics change in percent. Never mix the two notations.
- One comparison window per digest, stated in the header. Any exception is labelled inline at the point of use.
- Use only the three grades — Confirmed, Likely, Unexplained. No "possibly," "seems to," "appears," "may have," or "suggests."
- Do not recommend budget reallocation, campaign pauses, or pricing changes. Surface the evidence and the option; the human owns the spend decision.
- Never blend two GA4 properties, or GA4 and an ad platform, into one figure. Report side by side with a one-line note that the definitions differ.
- Flag suspected tracking breakage as a data issue with a same-day owner, never as performance.
- Keep any day that closed fewer than 48 hours before the pull labelled provisional, in the header and in the table.
- Do not carry forward last week's explanation without re-testing it against this week's segments.
- Cap the findings list at six and the digest at 500 words.

## Output format

```
# Weekly GA4 Digest — [Site/Property name]
Week of [Mon DD]–[Mon DD] vs [comparison window]. Data pulled [date, time]. [Provisional-data note if any.]

## The short version
[One or two sentences, at least one number: the single thing the reader should know.]

## The numbers
| Metric | This week | Prior | Change | Status |
|---|---|---|---|---|
| Sessions | | | ±n (±%) | moved / normal |
| Users | | | | |
| Engagement rate | | | ±n pp | |
| Key events (purchases) | | | | |
| Conversion rate | | | ±n pp | |
| Revenue | | | | |

## What moved and why
**1. [Finding headline]** — [Confirmed / Likely / Unexplained]
What: [metric, absolute and % change, the segment carrying it and its share of the change]
Why: [evidence, or "cause not identified" plus the check that would settle it]
So what: [implication in one sentence]

**2. [Finding headline]** — [grade]
[same three lines]

**3. [Finding headline]** — [grade]
[same three lines]

## Watch items
- [Signal] — [current number] — [threshold and duration that would promote it to a finding]

## Next steps
- [Action] — [named owner or role] — [by date]

## What I could not see
- [Missing input] — [the one question it would have answered]
```

- Target 350–500 words. The digest is short; this playbook is the long part.
- Order is fixed: verdict, numbers, findings by impact, watch items, next steps, gaps. Readers stop after the verdict in a quiet week — that is the design, not a failure.
- Maximum six findings, maximum three watch items, maximum four next steps.
- When it runs long, cut in this order: watch items, then findings from rank 6 upward, then dimension detail inside findings. Never cut an evidence grade, the "What I could not see" block, or an absolute figure.

## Worked example

**Input.** A CSV from Northfield Coffee's GA4 property (UK DTC coffee subscription and beans), exported Tuesday 18 March 2025 at 09:15. 14 daily rows covering 10–16 March 2025 and 3–9 March 2025, with session default channel group, device category, and landing page. Ten prior weeks of weekly totals available from previous digests.

| | 10–16 Mar | 3–9 Mar |
|---|---|---|
| Sessions | 18,420 | 19,880 |
| Users | 14,310 | 15,460 |
| Engagement rate | 54.1% | 56.8% |
| Purchases | 412 | 388 |
| Conversion rate | 2.24% | 1.95% |
| Revenue | £24,710 | £22,140 |

Sessions by channel: Organic Search 7,980 vs 7,820; Direct 4,560 vs 5,080; Paid Social 2,110 vs 3,640; Email 1,940 vs 1,180; Paid Search 1,180 vs 1,360; Referral 470 vs 560; Unassigned 180 vs 240.

Purchases by channel: Organic 168 vs 162; Email 96 vs 41; Direct 82 vs 98; Paid Social 34 vs 52; Paid Search 24 vs 27; Referral 6 vs 6; Unassigned 2 vs 2.

Revenue by channel, change: Email +£4,070; Direct −£940; Paid Social −£1,030; all others +£470 net.

Engagement rate by channel: Organic 61.0% vs 65.8%; Direct 52.0% vs 56.4%; Email 71.4% vs 70.9%; Paid Social 38.2% vs 37.6%.

Device: mobile 68% of sessions, conversion rate 1.81% vs 1.72%; desktop 3.18% vs 3.41%.

Eight-week means: sessions 19,340 (SD 690); revenue £22,480 (SD £1,120); engagement rate 56.9% (SD 0.8pp).

Stakeholder note from Sam (CRM): "Sent the spring blend email Wednesday lunchtime." No note about paid social. No known holidays in either window.

**Decisions behind the output.** The 16 March row closed 33 hours before the pull, inside the 48-hour window, so the whole digest is labelled provisional. Sessions at −2 SD below the 8-week mean, revenue at +2 SD, engagement rate at −3.5 SD: all three qualify as moved; purchases at +0.6 SD do not. Impact ranking by absolute revenue change puts email (+£4,070) first, paid social (−£1,030) second, and the engagement-rate collapse third — retained above its rank because engagement rate breached its 12-week floor of 55%. Email clears both Confirmed tests: first lift on Wednesday 12 March, same day as the send, and 90% of gross purchase gains. Paid social gets Unexplained — no event, and no stakeholder assertion to lean on. The engagement drop decomposes to +1.1pp of mix effect (losing low-engagement paid social) and −3.8pp of within-channel rate decline concentrated in Organic and Direct, which is broad-based, so it stays Likely with a named check. Bot check: no segment with engagement under 10% and duration under 5 seconds. Tracking check: all four key events within ±2 SD; purchases and revenue both present on every row. Calendar check: no holiday in either window. Finished digest, 468 words.

**Output.**

# Weekly GA4 Digest — Northfield Coffee
Week of 10–16 March vs 3–9 March. Data pulled 18 March, 09:15. Provisional: Sunday 16 March closed 33 hours before the pull; expect +1.5–3% upward revision on that day.

## The short version
Revenue rose £2,570 (+11.6%) on 1,460 fewer visits — the spring blend email did the work, while paid social traffic fell 42% for reasons this data cannot explain.

## The numbers
| Metric | This week | Prior | Change | Status |
|---|---|---|---|---|
| Sessions | 18,420 | 19,880 | −1,460 (−7.3%) | moved |
| Users | 14,310 | 15,460 | −1,150 (−7.4%) | moved |
| Engagement rate | 54.1% | 56.8% | −2.7pp | moved |
| Purchases | 412 | 388 | +24 (+6.2%) | normal |
| Conversion rate | 2.24% | 1.95% | +0.29pp | moved |
| Revenue | £24,710 | £22,140 | +£2,570 (+11.6%) | moved |

Engagement rate = share of visits where someone stayed 10+ seconds, scrolled, or clicked.

## What moved and why
**1. The spring blend email carried the revenue gain** — Confirmed
What: Email sessions 1,180 → 1,940 (+760, +64%), purchases 41 → 96 (+55), revenue +£4,070. Email is 90% of gross purchase gains; the site-wide net is +24 after paid social −18 and direct −16. The lift starts Wednesday 12 March and runs through Friday.
Why: Sam's send went out Wednesday lunchtime — timing matches to the day and the gain sits entirely in the email channel.
So what: The send worked and revenue tailed for two days after it. Use that tail when forecasting the next send.

**2. Paid social sessions nearly halved** — Unexplained
What: Paid Social 3,640 → 2,110 (−1,530, −42%), which is 105% of the total site session decline. Paid social purchases 52 → 34; revenue −£1,030. The drop starts Monday 10 March and does not recover.
Why: Cause not identified. No campaign change was reported and no tracking anomaly was found. Checking spend, delivery, and UTM tagging in Meta Ads Manager for 9–11 March would separate a budget change from a delivery or tagging issue.
So what: Every other channel except direct held within normal variation; this one channel is the whole traffic story.

**3. Engagement rate fell across organic and direct** — Likely
What: −2.7pp overall, 3.5 SD below the 8-week mean and under the 55% floor. Losing low-engagement paid social should have pushed the average up 1.1pp; instead organic fell 65.8% → 61.0% and direct 56.4% → 52.0%.
Why: No known event matches. Broad-based across two channels points to a site-side change — a release, a consent banner, or a slow template.
So what: Purchases held, so this is not yet a revenue problem, but two channels moving together is worth 20 minutes of checking.

## Watch items
- Mobile conversion rate 1.81% vs 1.72% — inside normal variation; promote to a finding if it holds above 2.00% for two consecutive weeks.

## Next steps
- Pull Meta spend, impressions and UTMs for 9–11 March — Priya, paid media — by Thu 20 March.
- Check for a site release or consent-banner change on 10–11 March — Dan, engineering — by Thu 20 March.
- Book the next blend email into the Wednesday lunchtime slot — Sam, CRM — by Mon 24 March.

## What I could not see
- Ad platform spend and impressions — would separate a budget cut from a delivery or tagging issue in finding 2.
- Site release log — would confirm or clear the site-side explanation in finding 3.

## Quality bar

- [ ] Every headline figure shows both an absolute and a percentage change; every rate metric uses percentage points.
- [ ] No percentage change is reported on a segment under 100 sessions or 25 key events in either period.
- [ ] Each finding carries exactly one grade from the set {Confirmed, Likely, Unexplained} and no other hedging word appears in the digest.
- [ ] Every Confirmed grade has a timing match within ±1 day and a stated segment share of ≥60%.
- [ ] Every Unexplained finding names a check, a person or role, and a date.
- [ ] One comparison window is named in the header and every figure in the digest uses it.
- [ ] Findings are in descending order of absolute revenue change (or key events, or engaged sessions, if revenue is absent); any out-of-order finding is labelled "threshold breach."
- [ ] Bot traffic, tracking breakage, and calendar effects are each explicitly ruled out or flagged for every movement above 25%.
- [ ] Segment changes quoted inside a finding reconcile to the total change for that metric.
- [ ] Attributed shares of any single movement sum to ≤100% of the correctly stated denominator.
- [ ] No recommendation commits budget, pauses a campaign, or changes a price.
- [ ] Any day closed under 48 hours before the pull is labelled provisional in both the header and the table.
- [ ] The digest is 350–500 words, has at most six findings, and ends with the "What I could not see" block.

## Failure modes

**Confident cause, no evidence** — the biggest mover gets paired with whatever event was mentioned, regardless of timing or segment — check that every Confirmed grade satisfies both the ±1 day timing test and the ≥60% concentration test; downgrade to Likely otherwise, and to Unexplained if neither holds.

**Noise reported as news** — a 340-session channel swings 80% and leads the digest — check each finding against the volume floor (100 sessions / 25 key events) and the ±2 SD or ±10% threshold before it enters the list; anything failing either goes to Watch items or is dropped.

**Phantom drop on the final day** — an incomplete Sunday is read as a crash — check the last day's sessions against the week's daily median; if it is below 70% of the median and closed under 48 hours before the pull, exclude the day or label the digest provisional.

**Comparison window drift** — one finding uses week-over-week, another quietly uses year-over-year, and the totals stop reconciling — check that segment changes sum to the total change for the stated window; a residual above 2% means one figure came from a different window.

**Double-counted explanation** — email and a paid campaign are both credited with the same revenue lift, implying 160% of the change — check whether the denominator is net or gross movement; quote gross when segments move in opposite directions, and rewrite as "broad-based" when no segment reaches 30%.

**Mix effect mistaken for a quality drop** — losing a low-engagement channel changes the blended average and gets narrated as visitors disengaging — check by recomputing the blended rate with last week's channel mix; report the mix and rate components separately whenever they point in opposite directions.

**Tracking break sold as performance** — one key event falls 94% and becomes the week's headline finding — check whether the other key events moved together; a single event collapsing while the rest sit within ±2 SD is a tag issue and belongs in a Data issue line with a same-day owner.

**Recycled explanation** — last week's promo is credited again this week because it worked last time — check this week's timing and segment concentration from scratch; a cause that no longer meets both tests loses its Confirmed grade.

## License

MIT
