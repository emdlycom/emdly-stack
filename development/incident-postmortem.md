---
name: incident-postmortem
owner: sevzero
category: Development
description: Drafts blameless postmortems from a timeline and a channel export — impact, contributing factors, and actions with owners.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/sevzero/incident-postmortem
raw: https://emdly.com/raw/sevzero/incident-postmortem.md
install: npx @emdly/cli add sevzero/incident-postmortem
---

# Incident postmortem

Written the day after, from what actually happened, without a villain. This skill drafts it so the responders only have to correct, not compose. The output is for the people who were awake during the incident and for the team who will hit the next one, and it must be true of every sentence that a responder reading it recognises the night they had. That means the gaps stay gaps, the numbers carry their sources, and nothing is smoothed into a story because a story reads better than an admission.

## When to use
- After any incident with a status page entry or a paged responder.
- For near-misses, when the team decides it is worth learning from.
- When the incident channel has scrolled past the point where anyone can reconstruct the order of events by reading it.
- Before a review meeting, so the meeting spends its time on the actions rather than on the timeline.
- Not for: an incident still in progress. Draft nothing until it is resolved; see Stop and hand back.

## Input
The incident channel export (with timestamps), alerts, deploy log, the status page updates, and the monitoring numbers for impact (error rate, latency, affected requests or customers).

State in one line which of those five you were given and which you were not. The **channel export** carries the timeline and the decisions; the **monitoring numbers** carry the impact. Missing either changes what the document can claim, and the header says so rather than the reader discovering it in section 2.

## Sections

1. **Summary** — three sentences: what users experienced, for how long, what fixed it. No cause in the summary; the cause is section 4, where it can be qualified.
2. **Impact** — in user terms with numbers: requests failed, orders delayed, customers affected, and the time window. Every figure names its source. A percentage is written with the denominator that produced it. Money only if the input contains it — never estimated, never extrapolated from a rate.
3. **Timeline** — reconciled across sources, in UTC, one line per event: detection, escalation, each hypothesis tried, mitigation, resolution. Mark gaps ("no activity 02:10–02:40"). Where two sources disagree on a time, show both and mark it, rather than picking the tidier one.
4. **Contributing factors** — plural, always. Not "root cause": the deploy, the missing alert, the runbook that was stale, the load pattern. Each one is a fact, not a judgment. Three to six of them; one is a root cause with a new name, and ten is a list nobody acts on.
5. **What went well** — detection time, a good call, a runbook that worked. At least one, and it must be specific enough to cite a timestamp. If nothing did, write that, and say what would have.
6. **Actions** — each with an owner (role or name from the channel), a due date placeholder, and how we will know it is done. Split into *prevent recurrence*, *detect faster*, *mitigate faster*. Every one of the three headings appears, even if the entry under it is "none identified".

## Rules
- Blameless means no names attached to mistakes. Names appear only as action owners and in the timeline as roles ("on-call", "deployer").
- Quote the channel for decisions ("02:14 on-call: rolling back 1.14.2") rather than narrating intent. You did not have access to anyone's intent.
- Every number cites its source (alert, dashboard, log line). A number with no source does not go in.
- Mark what you could not establish as **"unknown"** — an honest gap beats a smooth story. Use the literal word. `unknown` is a finding: each one gets a line saying what would have established it, and a recurring `unknown` is itself an action about instrumentation.
- Actions must be specific enough to close: "add an alert on webhook queue depth > 5 000 for 5 min", not "improve monitoring".
- Do not write a counterfactual as a fact. "The canary would have caught this" is a claim; "the canary stage was skipped" is the fact, and the claim belongs in the action's rationale if anywhere.
- Recompute every figure before writing it. Percentages must reconcile with their counts.

## Output format

```
# Postmortem — checkout errors, 2026-08-14

## Summary
Between 01:52 and 03:07 UTC, 38% of checkout attempts returned HTTP 500 for 75 minutes.
1 130 customers were affected and 2 398 checkout attempts failed. Rolling back the 1.14.2
deploy resolved it.

## Impact
- 6 310 checkout attempts in the window; 2 398 returned 500. 2 398 / 6 310 = 38.0%.
  (source: checkout error dashboard, `http_requests_total{route="/checkout",status="500"}`)
- 1 130 distinct customer ids among the failures (source: orders table, failed_at between
  01:52 and 03:07).
- 412 of those customers retried and completed successfully before 03:07 (source: same query,
  matched on customer id). Net customers who did not complete in the window: 718.
- Revenue impact: unknown. The input contains no order values, and this document does not
  estimate them. Establishing it needs the finance export for 14 Aug — action D4.
- Status page: degraded at 02:09, resolved at 03:14 (source: status page log).

## Timeline (UTC)
01:52 deploy 1.14.2 completes (deploy log)
01:52 first 500s on /checkout (error dashboard; first sample at 01:52:40)
01:58 error-rate alert fires at 25% threshold (PagerDuty)
02:03 on-call acknowledges (PagerDuty)
02:06 on-call: "seeing 500s on checkout, looking at the payment client" (channel)
02:09 status page set to degraded (status page log)
02:10–02:40 no activity in the channel. The on-call's own notes are not in the export, so
      what was investigated in this window is unknown.
02:41 on-call: "provider dashboard is green, it's us" (channel)
02:47 second responder joins (channel)
02:58 deployer: "1.14.2 changed the payment timeout, PR #418" (channel)
03:01 on-call: "rolling back 1.14.2" (channel)
03:04 rollback completes (deploy log)
03:07 error rate returns to baseline, 0.4% (error dashboard)
03:14 status page resolved (status page log)

Disputed: PagerDuty records the acknowledgement at 02:03; the channel's first message from
the on-call is 02:06. Both are shown. Which reflects when they were actually at a keyboard
is unknown.

## Contributing factors
1. 1.14.2 changed the payment client timeout from 30 s to 3 s (PR #418); the provider's
   p95 is 4.1 s (provider status page, 30-day view).
2. The canary stage was skipped. The deploy pipeline allows skipping it with a flag and
   requires no second approval (pipeline config, `allow_skip_canary`).
3. The error-rate alert threshold was 25% for 5 min, so it fired at 01:58 against a first
   error at 01:52 — 6 minutes of unalerted failure.
4. Why the timeout was changed in PR #418 is unknown. The PR description does not say and
   the discussion is not in the input. Establishing it needs the author's comment — this is
   context for action P1, not a blocker for it.

## What went well
- Once the payment client was named at 02:58, the rollback decision took 3 minutes and the
  rollback itself 3 more (channel, deploy log).
- The status page was updated at 02:09, 11 minutes after the page, without being asked.

## Actions
| kind | action | owner | due | done when |
| prevent | P1 — canary skip requires a second approver | deploy pipeline owner | TBD | `allow_skip_canary` change merged and a skip attempt is blocked in a test run |
| prevent | P2 — client timeouts must exceed the provider's documented p95; add the check to PR review for payment code | payments lead | TBD | check present in the PR template and one PR blocked by it |
| detect | D3 — checkout error-rate alert at 10% for 2 min | on-call lead | TBD | alert fires within 2 min during a game day |
| detect | D4 — join order value to the failure query so revenue impact is available at incident time | data owner | TBD | the dashboard shows value alongside failed count |
| mitigate | M5 — runbook entry: on checkout 500s after a deploy, check `rollout history` before the provider dashboard | on-call lead | TBD | entry merged and used in the next game day |

Unknowns carried out of this incident: what was investigated 02:10–02:40, why the timeout
was changed, and the revenue impact. D4 closes the third. The first two are recorded and
not chased further.
```

## Edge cases

- **No channel export**, only alerts and the deploy log. The timeline can carry machine events but no decisions, and section 3's "each hypothesis tried" is unavailable. Say so at the top, write the machine timeline, and mark every human step `unknown`. Do not narrate what the responders were probably doing.
- **No monitoring numbers.** Section 2 cannot be written as specified. Write `Impact: unknown` with the reason, describe the impact qualitatively from the status page and channel, and make retrieving the numbers action one. Do not convert "a lot of errors" into a percentage.
- **Timestamps in mixed zones or formats.** Convert everything to UTC and say what you converted from. Where an offset is ambiguous — a bare local time with no zone — mark that line `time uncertain` rather than assuming the team's zone.
- **Sources disagree.** Show both, mark it disputed, and say which source you would trust for what. Never average two timestamps.
- **Export too large** — thousands of messages, or several days of channel. Take the window from first alert to resolution plus 30 minutes either side, say what window you used and how many messages fell outside it, and note that decisions made outside the window are not represented.
- **Near-miss with no user impact.** Sections 1 and 2 change shape: what would have happened, what stopped it, and how close it came. Say plainly that no users were affected, and keep sections 4 to 6 unchanged — the contributing factors are the whole point of writing it up.
- **A single responder.** Blameless anonymisation fails: "the on-call" identifies one person to everyone who saw the rota. Say so in the header, keep the roles, and let the responder read it before it circulates.
- **Names, customer records or personal data inside the export.** Do not carry them into the document, including in quoted channel lines. Replace with roles or ids, note that you redacted, and see Stop and hand back if the data itself was exposed by the incident.
- **Incident is still open**, or the mitigation is holding but the cause is not understood. Do not draft a postmortem. See Stop and hand back.

## Stop and hand back

Stop means: the draft does not circulate, not to the team channel and not to a review meeting, until the named party has read it. Say which trigger fired and to whom it went, at the top of the draft.

- **Data exposure.** Any sign that data was readable by someone not entitled to it — wrong-tenant responses, a cache serving another user's page, an object store left open, records in an error payload or a log shipped off-platform. Stop. Route to security and legal before anyone else reads the draft. Disclosure obligations run on clocks that start at discovery, and a postmortem circulating first can start them badly.
- **Personal data involved**, whether or not it left the building. Route to the privacy owner. Do not write the affected-record count into the draft until they confirm what may be stated; write `pending privacy review` in its place.
- **Payment failure.** Charges that failed, double-charged, were captured without an order, or refunds that did not process. Route to the payments owner and finance before circulating. Do not state a money figure the input does not contain, and do not describe an affected customer's charge state in a document going to a wide channel.
- **Regulated downtime.** A service under an SLA with a reporting obligation, or in a regulated category — payments, health, financial reporting, anything with a supervisory notification clock. Route to legal or compliance first, and ask before the timeline is published, because the timeline is often the reportable artefact.
- **The incident is still open.** Do not draft. Hand back a timeline-so-far marked "live incident, not a postmortem", and say the postmortem starts after resolution. A draft written mid-incident gets quoted as if it were the finding.
- **A contributing factor points at one person's action.** Stop before writing it. Hand to the incident lead to reframe as a system fact — what allowed the action, not who took it. If it cannot be reframed, it does not go in this document, and the conversation belongs with that person's manager rather than in a channel.
- **An action would change production** — an alert threshold, a pipeline gate, a timeout. The postmortem proposes it and names the owner. It does not make the change, and this skill never states an action as already taken.

## License
MIT
