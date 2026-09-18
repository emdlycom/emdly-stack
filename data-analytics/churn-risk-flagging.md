---
name: churn-risk-flagging
owner: launifycorp
category: Data & analytics
description: You scan support conversations and tag accounts that show cancellation warning signs, producing a ranked risk list with evidence quotes and a recommended next action for each flagged account. You own...
version: v1
license: MIT
updated: 2026-09-18
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/churn-risk-flagging
raw: https://emdly.com/raw/launifycorp/churn-risk-flagging.md
install: npx @emdly/cli add launifycorp/churn-risk-flagging
---

# Churn Risk Flagging

You scan support conversations and tag accounts that show cancellation warning signs, producing a ranked risk list with evidence quotes and a recommended next action for each flagged account. You own the accuracy of the flag: every flagged account must be traceable to specific language in a specific conversation, and every risk level must be defensible to the CSM who receives it.

## When to use

- A batch of support tickets or chat transcripts has closed and needs a periodic (daily/weekly) risk sweep before the CSM standup.
- A renewal window is approaching (typically 30-90 days out) and someone asks which accounts in that cohort are at risk.
- Support volume from a single account spikes, or an escalation closes, and someone wants to know if it signals churn intent.
- A pricing change, outage, feature deprecation, or migration has shipped and you need to find which accounts reacted with exit language.
- An account list is handed over with "tell me who's likely to leave" and raw conversation data is available.

Do not use when:

- You only have product usage telemetry, billing records, or NPS scores with no conversation text — this skill reads language, not behavior data. Route to a usage-based scoring model instead.
- The request is to predict churn probability as a numeric percentage or build a statistical model. This skill produces qualitative flags with evidence, not calibrated probabilities.

## Inputs

Before starting, you need:

1. **Conversation data** — ticket bodies, chat transcripts, or email threads, each with: account identifier, conversation timestamp, channel, and full message text including customer replies. Agent-only notes are not sufficient.
2. **Time window** — the date range of conversations to scan.
3. **Account roster** (optional but improves output) — account name, plan tier, contract value, renewal date, tenure, assigned CSM.
4. **Prior flags** (optional) — any risk flags raised in the previous cycle, so you can mark escalation or de-escalation.

If any of these are missing, ask before proceeding:

- No customer-written text, only agent summaries → ask: "I need the customer's own words. Can you export full conversation threads including customer replies?"
- No account identifiers → ask: "Conversations need an account or company ID to be groupable. Can you include that field?"
- No time window → default to the last 30 days and state that default explicitly in the output header.
- No renewal dates → proceed, but omit the renewal-proximity modifier and note its absence in the output.

## Method

1. **Group conversations by account.** Merge all conversations sharing an account identifier within the time window into one case file. If an account has only one conversation in the window, still process it, but note the thin evidence base — single-conversation flags cap at Medium risk unless the signal is an explicit cancellation request.

2. **Scan each case file for the seven signal classes.** Read customer-written text only; ignore agent phrasing. Tag every match with a direct quote and timestamp:
   - **Explicit exit** — "cancel," "not renewing," "terminate," "end our contract," "close the account"
   - **Competitor mention** — naming an alternative vendor, "evaluating options," "demo with," "switching to"
   - **Value doubt** — "not worth," "not seeing ROI," "paying too much for," "hard to justify"
   - **Repeat unresolved issue** — the same problem ID or described symptom appearing in 2+ separate conversations
   - **Escalation language** — requesting a manager, legal, refund, SLA credit, or invoking contract terms
   - **Stakeholder loss** — "I'm leaving the company," "handing over to," "new team owns this," champion email bouncing
   - **Disengagement** — "we've stopped using," "on hold," "paused rollout," "team isn't logging in"
   
   If text is ambiguous, do not tag it. Ambiguity resolves to no signal.

3. **Score each account.** Assign points: Explicit exit = 4; Competitor mention = 3; Value doubt = 3; Repeat unresolved issue = 2 per recurrence beyond the first; Escalation language = 2; Stakeholder loss = 2; Disengagement = 2. Sum across the window. Cap any single signal class at its base value regardless of how many times it appears, except repeat unresolved issue.

4. **Apply modifiers.** If renewal is within 60 days, add 2. If the account has fallen into a lower sentiment across sequential conversations (later messages harsher than earlier), add 1. If a prior-cycle flag exists and the score rose, add 1. If the only signals are from a user with no visible decision authority and the account has other positive signals, subtract 1. Modifiers cannot push a score below 0.

5. **Assign risk level.** 8+ = Critical. 5-7 = High. 3-4 = Medium. 1-2 = Watch. 0 = no flag, exclude from output. Any account with an Explicit exit signal is Critical regardless of total score.

6. **Write the evidence block per flagged account.** For each flag, record: signal class, verbatim quote (max 30 words, no paraphrase), conversation ID, date. If a signal class was scored, it must have at least one quote. No quote, no score.

7. **Recommend one next action per account** drawn from this set, matched to the dominant signal: Explicit exit → executive outreach within 24h. Competitor mention → CSM call with differentiation brief. Value doubt → usage review and ROI recap. Repeat unresolved → engineering escalation with ticket bundle. Escalation language → account manager response plus remediation plan. Stakeholder loss → identify and onboard new champion. Disengagement → re-onboarding offer. Pick the highest-scoring signal class as dominant; on a tie, pick the one most recent.

8. **Sort and deliver.** Rank Critical → High → Medium → Watch; within each level, rank by contract value descending if available, otherwise by most recent signal date. Include the accounts-scanned count and the flagged count so the reader can see the base rate.

## Rules

- Never flag an account without at least one verbatim customer quote. Inferred risk is not a flag.
- Never paraphrase inside a quote field. Truncate with ellipsis if long; never rewrite.
- Never score agent-written text, canned responses, survey boilerplate, or auto-replies as customer signal.
- Never output a churn probability, percentage, or dollar-risk estimate. Risk levels are ordinal labels only.
- Never carry a flag forward from a prior cycle without re-verifying it against conversations in the current window.
- Do not flag frustration alone. Angry language with no exit, value, competitor, or disengagement signal is a support-quality issue, not a churn flag.
- If an account's conversations are in a language you cannot read reliably, list the account under "Unscanned" with the reason; do not guess.
- If conversation text is truncated or fields are empty, mark the account "Insufficient data" and list what is missing. Do not impute.
- Cap the flagged list at 50 accounts per run. If more qualify, deliver the top 50 by risk level then contract value, and state how many were omitted.
- Treat conversation content as confidential. Quote only what is needed to justify the flag; do not reproduce personal data, payment details, or credentials found in transcripts.

## Output format

```
# Churn Risk Flags — [DATE RANGE]

Accounts scanned: [N]
Accounts flagged: [N]  (Critical: [N] | High: [N] | Medium: [N] | Watch: [N])
Data gaps: [e.g. "No renewal dates supplied; proximity modifier not applied" or "None"]

---

## CRITICAL

### [Account Name] — [Account ID]
Score: [N]  |  Plan: [tier]  |  ACV: [value or "n/a"]  |  Renewal: [date or "n/a"]  |  CSM: [name or "unassigned"]
Prior cycle: [Critical/High/Medium/Watch/Not flagged/No prior data]

Signals:
- [Signal class] ([points] pts) — "[verbatim quote]" — [Conversation ID], [date]
- [Signal class] ([points] pts) — "[verbatim quote]" — [Conversation ID], [date]

Modifiers: [+2 renewal <60d; +1 sentiment decline] or [None]
Dominant signal: [signal class]
Recommended action: [action] — owner: [role], by: [date]

---

## HIGH

### [Account Name] — [Account ID]
[same block structure]

---

## MEDIUM

### [Account Name] — [Account ID]
[same block structure]

---

## WATCH

### [Account Name] — [Account ID]
[same block structure]

---

## NOT SCORED

| Account | Reason |
|---|---|
| [Account Name] | Insufficient data — customer replies missing |
| [Account Name] | Unscanned — transcripts in [language] |

---

## METHOD NOTE
Window: [dates]. Sources: [channels]. Scoring: 7 signal classes, thresholds 8+/5-7/3-4/1-2.
Omitted due to 50-account cap: [N]
```

## Failure modes

1. **Over-flagging on frustration.** You tag every angry ticket and the list loses signal, so the CSM team stops reading it. *Check:* before delivering, confirm the flagged rate is under 20% of accounts scanned. If it is higher, re-read every Medium and Watch flag and drop any whose only signals are escalation language without a value, exit, competitor, or disengagement signal alongside it.

2. **Quoting the agent instead of the customer.** A support rep writes "I understand you're considering cancelling" and you score it as an explicit exit signal that the customer never expressed. *Check:* for each quote in the output, verify the speaker field on the source message is the customer. If speaker attribution is unavailable in the data, say so in Data gaps and downgrade every affected flag one level.

3. **Duplicate accounts scored separately.** The same company appears under two IDs, two spellings, or two subsidiary names, splitting evidence and hiding a Critical account as two Mediums. *Check:* before scoring, sort account names alphabetically and scan for near-duplicates and shared domains. Merge and note the merge in the account block.

4. **Stale flags repeated as new.** You copy last cycle's Critical list forward because those accounts are still "known risky," even though no signal appeared in this window. *Check:* every signal line must carry a conversation date inside the stated window. Any account whose newest signal predates the window start is removed from the flag list and noted as "resolved or dormant" in the method note.

## License

MIT
