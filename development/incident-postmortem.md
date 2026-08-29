---
name: incident-postmortem
owner: sevzero
category: Development
description: Drafts blameless postmortems from a timeline and a channel export — impact, contributing factors, and actions with owners.
version: v3
license: MIT
updated: 2026-08-08
recommended: true
security_checked: true
url: https://emdly.com/skills/sevzero/incident-postmortem
raw: https://emdly.com/raw/sevzero/incident-postmortem.md
install: npx @emdly/cli add sevzero/incident-postmortem
---

# Incident postmortem

Written the day after, from what actually happened, without a villain. This skill drafts it so the responders only have to correct, not compose.

## When to use
- After any incident with a status page entry or a paged responder.
- For near-misses, when the team decides it is worth learning from.

## Input
The incident channel export (with timestamps), alerts, deploy log, the status page updates, and the monitoring numbers for impact (error rate, latency, affected requests or customers).

## Sections
1. **Summary** — three sentences: what users experienced, for how long, what fixed it.
2. **Impact** — in user terms with numbers: requests failed, orders delayed, customers affected, and the time window. Money only if the input contains it.
3. **Timeline** — reconciled across sources, in UTC, one line per event: detection, escalation, each hypothesis tried, mitigation, resolution. Mark gaps ("no activity 02:10–02:40").
4. **Contributing factors** — plural, always. Not "root cause": the deploy, the missing alert, the runbook that was stale, the load pattern. Each one is a fact, not a judgment.
5. **What went well** — detection time, a good call, a runbook that worked.
6. **Actions** — each with an owner (role or name from the channel), a due date placeholder, and how we will know it is done. Split into *prevent recurrence*, *detect faster*, *mitigate faster*.

## Rules
- Blameless means no names attached to mistakes. Names appear only as action owners and in the timeline as roles ("on-call", "deployer").
- Quote the channel for decisions ("02:14 on-call: rolling back 1.14.2") rather than narrating intent.
- Every number cites its source (alert, dashboard, log line).
- Mark what you could not establish as "unknown" — an honest gap beats a smooth story.
- Actions must be specific enough to close: "add an alert on webhook queue depth > 5 000 for 5 min", not "improve monitoring".

## Output format
```
# Postmortem — checkout errors, 2026-08-14

## Summary
Between 01:52 and 03:07 UTC, 38% of checkout attempts returned 500 (≈ 2 400 attempts, 1 130 customers). Rolling back the 1.14.2 deploy resolved it.

## Impact
… (source: checkout error dashboard, orders table)

## Timeline (UTC)
01:52 deploy 1.14.2 completes (deploy log)
01:58 error-rate alert fires (PagerDuty)
02:03 on-call acknowledges
…

## Contributing factors
1. 1.14.2 changed the payment client timeout from 30 s to 3 s (PR #418); the provider's p95 is 4.1 s.
2. The canary stage was skipped — the deploy pipeline allows it with a flag, no review required.
3. The alert threshold (25% for 5 min) delayed detection by ~6 min.

## Actions
| kind | action | owner | done when |
| prevent | canary skip requires a second approver | deploy pipeline owner | flag change merged |
| detect | error-rate alert at 10% for 2 min on checkout | on-call lead | alert fires in a game day |
```

## License
MIT
