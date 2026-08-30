---
name: user-interview-distiller
owner: fieldnotes
category: Research
description: Distills interview transcripts into claims with evidence — what was said, how often, and what it contradicts in your assumptions.
version: v5
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/fieldnotes/user-interview-distiller
raw: https://emdly.com/raw/fieldnotes/user-interview-distiller.md
install: npx @emdly/cli add fieldnotes/user-interview-distiller
---

# User interview distiller

Interviews produce stories; decisions need claims. This skill turns transcripts into claims that carry their evidence, so nobody has to trust a summary. The output is for a room about to commit engineering months, and it must be true of every claim in it that a reader who disagrees can open the transcript at the quoted line and check. A claim whose evidence is the distiller's memory of reading is the thing this skill exists to prevent, and so is a count that cannot be traced to specific participants.

## When to use
- After a round of interviews, on the transcripts, once coding can be done in one pass across all of them.
- Before a roadmap meeting where "users want X" is about to be said without a number attached.
- To re-read an old round against a new question, by re-coding rather than by searching the previous summary.
- To settle a disagreement between two people who both sat in the interviews and remember them differently.
- Not for: sales calls, support tickets, or survey free-text. The quote-attribution scheme assumes a participant who consented to be quoted and an id that maps to one person.

## Input

Supply all three. Say in one line which you were given and how many participants.

1. **Transcripts, with participant ids.** One id per person, stable across the file, in the form `P1`, `P2` or equivalent. Every quotable line must be attributable to exactly one id. This is not a formatting preference: steps 2 and 3 count participants and every claim carries quotes with ids. Without ids the method does not work; see Edge cases.
2. **The research questions** the round was run to answer.
3. **The team's assumptions going in** — a list, even a rough one. Step 4 tests each one and cannot invent them afterwards.

**On round size.** Guest, Bunce & Johnson (2006, *Field Methods* 18(1):59–82) coded 60 interviews and found 73% of their codes present in the first six transcripts and 92% within twelve, with saturation at twelve. Hennink & Kaiser's systematic review of empirical saturation tests (2022, *Social Science & Medicine* 292:114523) puts saturation across studies at roughly 9–17 interviews for a narrow, homogeneous sample. The catalog's earlier "5–12" figure sat at the low end of that literature. Use **9–17 for a homogeneous group, more where the population is segmented**, and note that these studies concern thematic saturation in a single population: two distinct user segments are two saturation curves, not one. Report the number you actually had and do not claim saturation you did not test for.

> Thresholds above are defaults; report the thresholds you used.

## Process

1. **Code first, count second.** Read each transcript and tag statements with a short code (`export-manual`, `trust-numbers`). Codes come from what people say, not from the assumptions. Do not look at counts until every transcript is coded; a code invented on transcript 7 must be checked back against transcripts 1–6 before it is counted.
2. **Claims.** For each code appearing in **2 or more participants**, write a claim in neutral language and attach the participant count and the two most representative quotes with ids. `[house rule]` — 2 is the smallest number that can distinguish a pattern from a voice. It is a floor for listing something as a claim, not a threshold for believing it; a 2-of-9 claim is reported as 2 of 9 and reads as weak on its face. Every code at 2 or more becomes a claim, including one you would rather report only as a surprise; a code cannot skip the Claims section by being interesting.
3. **Single-voice items** go in their own section. Real, but not a pattern yet. They are not promoted by being interesting. Every code at exactly 1 participant appears here, so that the two sections together account for every code you assigned.
4. **Assumptions cross-check.** For each assumption: supported / contradicted / not addressed, naming the claim that decides it. Every supplied assumption appears, including the ones nothing touched.
5. **Surprises.** Things nobody expected and no research question asked about, quoted. A surprise points at the claim or single-voice item that carries it; it does not restate a count, and it never introduces a code the sections above did not list.
6. **Report the denominator once, at the top**, and use the same denominator everywhere. Where a topic did not reach every participant, carry the sub-denominator on the claim line and say how the topic reached the people it reached: `3/9 (asked of 5)` when five were asked and nobody else raised it, or `3/9 (topic reached 6: unprompted in P4, then asked of P5–P9)` when one participant raised it unprompted and the rest were asked. Never count a participant inside a sub-denominator they sit outside — if P4 is one of the three, P4 is inside the reach set, and the reach set is 6, not 5.

## Rules

- **Every claim quotes.** A claim without a quote is deleted, not softened.
- **Count participants, not mentions.** One person saying it six times is one.
- **Never turn a request into a claim about need.** "P3 asked for CSV export" is evidence for "P3 wants to work with the data elsewhere", and the claim says the latter, with the quote showing the request.
- **Leading questions are flagged.** A question that names the answer ("was the export frustrating?") gets its answer marked `[leading]`, and that answer cannot be one of a claim's two representative quotes. If every quote for a code is `[leading]`, the code does not become a claim; it becomes a line in the single-voice section saying the round asked for it.
- **Preserve the participants' words.** Do not clean up grammar, filler, or false starts. Elide only with `[…]` and never across a change of subject.
- **Neutral language in claims** means: no evaluative adjective, no product name as the subject, no "users love / hate / struggle with". State the behaviour and its object. "They rebuild our numbers in a spreadsheet before trusting them" is a claim; "users don't trust the dashboard" is a conclusion.
- **A claim states behaviour or belief, not a solution.** If the claim names a feature, it is a request and belongs in evidence, not in the claim line.
- **A first-appearance statement is checkable.** If you write that a code first appeared in P8, no participant before P8 may carry that code anywhere in the output. Re-read the Claims and Surprises sections against each other before you ship.

## Output format

### Example 1 — the normal case, with a leading-question flag and an empty assumption

```
## Round: data-trust interviews, 9 participants (single segment: finance analysts)
Denominator: 9 unless a claim says otherwise. Saturation not tested; 9 is the low end of
the 9–17 band in Hennink & Kaiser (2022). Two codes first appeared in P8 and P9, so this
round is probably not saturated — see Surprises.

## Claims (9 participants)
**C1 · 6/9 — They rebuild our numbers in a spreadsheet before trusting them.**
- P2: "I export it, sum it myself, and only then do I put it in the deck."
- P7: "the dashboard says one thing, finance says another, so I check, every time"
→ contradicts assumption A3 ("users trust the dashboard totals").

**C2 · 4/9 — Onboarding is done by a colleague sitting next to them, not by the docs.**
- P1: "[a colleague] showed me. I've never opened the help thing, is there a help thing?"
  (name redacted; the participant named a colleague who consented to nothing.)
- P6: "you learn it from whoever had the job before you, that's just how it went"
→ not addressed by any assumption.

**C3 · 3/9 (topic reached 6: unprompted in P4, then asked of P5–P9) — They keep a private
copy of the report because the shared one changes under them.**
- P4: "I duplicate it into my own folder because someone edits the shared one mid-month"
- P9: "mine's a copy. I know it's stale. I'd rather it be stale than move."
→ supports assumption A6 ("people fork shared reports").
Note: the topic came up unprompted in P4 and was then asked of P5–P9. P1, P2 and P3 were
never asked and never raised it, so the topic reached 6 of the 9, not 5. The three who
hold the code are P4, P6 and P9; P4 is inside the reach set because P4 raised it.

**C4 · 3/9 — They send the export to an external auditor as a routine hand-off.**
- P1: "the export is for the auditor, honestly. that's the only reason I open it."
- P8: "end of quarter I just send the CSV to the audit people, that's what it's for now"
→ not addressed by any assumption. Code `audit-handoff`, first seen in P1. Listed here
because it reached 3 participants; see Surprises for why it is also a surprise.

**C5 · 2/9 — They check a number's freshness before quoting it, and cannot find when it
last updated.**
- P3: "I always look for a timestamp. There isn't one. So I ask in Slack."
- P8: "if I don't know when it ran I won't put it in front of the board"
→ supports assumption A1 ("weekly usage") only weakly; A1 is about frequency, this is not.
Marked as not deciding A1.

Five codes reached 2 or more participants and all five are above. No other code reached 2
participants with a non-leading quote. One that reached 2 on leading quotes only is in
Single voices.

## Single voices
- P5 wants a Slack digest. No other participant mentioned notifications.
  P5: "just put it in Slack, I live there"
- P2 exports to Google Sheets specifically, not Excel. Everyone else who exports said Excel
  or did not say. Too small to judge whether the tool matters.
- Code `stale-tolerance` (1 participant, P9). Staleness is accepted as the price of a copy
  that does not move. First appeared in P9, the last interview.
  P9: "I'd rather it be stale than move."
- Code `board-sign-off` (1 participant, P8). A number that goes to the board is held to a
  different bar than one used internally. First appeared in P8.
  P8: "if I don't know when it ran I won't put it in front of the board"
- Code `mobile-check` (2 participants, P1 and P6) is not listed as a claim: both quotes came
  after the interviewer asked "do you ever check this on your phone?", which names the
  answer. Flagged [leading]. If mobile matters, ask it open next round.

## Assumptions
| id | assumption | verdict | decided by |
| A1 | users open the report weekly | not addressed | no claim establishes frequency; C5 is about freshness, not cadence |
| A3 | users trust the dashboard totals | contradicted | C1 (6/9) |
| A5 | people use this on mobile | not addressed | only evidence is [leading], see Single voices |
| A6 | people fork shared reports | supported | C3 (3/9, topic reached 6) |
| A8 | finance sign-off is the blocker | not addressed | nobody raised sign-off; the round never asked |

## Surprises
- C4, the auditor hand-off. No research question asked about auditors and no assumption
  named one, yet three participants describe the export as an audit artifact rather than a
  reporting one. The product does not mention auditors anywhere.
  P4: "the CSV goes to the auditor. it's not really a dashboard thing, it's an audit thing."
- Two codes (`board-sign-off`, `stale-tolerance`) first appeared in P8 and P9, the last two
  interviews, and both are still single voices. New codes arriving at the end of a round is
  the signal that the round is under-sampled. Recommend 4–6 more analysts before treating
  C3 or C5 as settled.

## Consent and redaction
Redactions: 1 (C2, P1 — a colleague's name replaced with `[a colleague]`). The colleague
is a third party; flagged to the research owner, not coded, not quoted further.
Consent on file for all 9 participants covers an internal report; it does not cover an
external deck. See Stop and hand back before this circulates outside the product team.
```

### Example 2 — the refusal, too few participants

```
## Round: 1 participant — not distilled

Supplied: 1 transcript (P1), 4 research questions, 6 assumptions.

No claims produced. A claim in this skill means a code seen in 2 or more participants;
with one transcript every code has n=1 and the claims section would be the single-voice
section under a more confident heading.

What is returned instead:
- Codes found, with quotes, unaggregated: `export-manual` (3 statements), `audit-handoff`
  (1), `trust-numbers` (2), `no-timestamp` (1).
- P1: "I export it, sum it myself, and only then do I put it in the deck."
- P1: "end of quarter I just send the CSV to the audit people"
- Assumptions: all 6 marked **not addressed**. One transcript cannot support or contradict
  an assumption about a population.
  A1 not addressed · A3 not addressed · A5 not addressed · A6 not addressed ·
  A8 not addressed · A9 not addressed.
- Leading questions found: 1 (Q3, "was the export frustrating?"). P1's answer to it is
  excluded from the codes above.

This is interview notes, not a distillation. Do not take it to a roadmap meeting as
evidence about users. Guest et al. (2006) found 73% of codes in the first six transcripts;
one transcript is not a sample of anything.
```

## Edge cases

- **Fewer than 2 participants.** Do not produce claims. Return the Example 2 shape: codes with quotes, unaggregated, all assumptions marked not addressed, and a plain statement that this is notes rather than evidence. Never write a claim reading `1/1`.
- **2 to 4 participants.** Claims are producible but every one carries its denominator prominently, and the header says `too small for pattern claims — read every count`. Do not report percentages at these sizes; `3/4` is honest, `75%` is not.
- **Transcripts with no participant ids.** The method collapses. Every claim carries quotes with ids, every count is a count of participants, and neither is available. Do not assign ids by guessing at speaker turns, and do not attribute quotes to `P?`. Return `no participant ids — cannot attribute or count`, list the codes with their quotes unattributed and marked as such, and say exactly what is needed: a speaker label per turn, stable across the file. If the transcripts have speaker turns but anonymous labels ("Speaker 1", "Interviewee"), say whether those are stable **within one file only** — they usually are — in which case ids can be assigned per file and counting works across files but not within one.
- **Transcripts where interviewer and participant are not distinguished.** Worse than missing ids, because the interviewer's words will be coded as findings. Do not proceed. Say so and ask for a labelled transcript.
- **Mixed segments in one round** — three analysts and six admins. Do not pool them. Report claims per segment with per-segment denominators, and say which codes crossed segments. A 5/9 that is 5 of the 6 admins and 0 of the 3 analysts is a segment finding wearing a round finding's clothes. A segment of three or fewer is also a re-identification risk; see Stop and hand back before publishing its breakdown.
- **A topic that only some participants were asked about.** Carry the sub-denominator per step 6 and name the reach set explicitly. If a participant raised the topic unprompted, they are inside the reach set: `3/9 (topic reached 6: unprompted in P4, then asked of P5–P9)`. A sub-denominator that excludes someone you counted in the numerator is an arithmetic error, not a rounding choice.
- **Transcripts too long to code in one pass.** Code in order, keep the code list open, and re-check every code invented after the first pass against the earlier transcripts before counting it. Say how many transcripts you re-checked. Never count a code across a set you only coded forward.
- **A transcript is partial, cut off, or unusable** — bad audio, half a session, a no-show. Exclude it, count it in the header (`unusable transcripts: 2`), and keep the denominator at the number actually coded. Do not silently drop it from the count.
- **No assumptions supplied.** Section 4 cannot run. Say `no assumptions supplied — cross-check not run` and do not reverse-engineer assumptions from the claims; that produces a list every claim happens to support.
- **No research questions supplied.** Claims and single voices still work. The Surprises section does not, because a surprise is defined against what was asked. Say so and omit it rather than nominating the most interesting claim.
- **Every quote for a code is leading.** Not a claim. See the rules; it goes to single voices with the flag and a note to ask it open next round.
- **A quote identifies the participant** — names their employer, their manager, a colleague, or a detail only they could have said. Replace the identifying span with `[…]` or a bracketed role, note that you redacted, and count the redactions in the header. A pseudonymous id does not anonymise a quote that describes one person's job. Redacting is an edit you make; deciding whether the redacted quote may then circulate is not yours — see Stop and hand back.

## Stop and hand back

Every quote here came from a person who agreed to an interview, not to publication, and the output travels to a room that will forward it. Stop means: the quote, the claim or the whole distillation does not leave this file until the named human clears it. Redacting is an edit you make; deciding what may circulate is not.

- **A quote the participant did not consent to have circulated.** Consent to be recorded is not consent to be quoted, and consent to be quoted in an internal report is not consent to be quoted in a deck that leaves the company. If the consent record does not cover the audience this distillation is going to, hold the quote, write `quote withheld — consent not established for this audience` in its place, and keep the claim only if a second consented quote supports it. The researcher who ran the round decides, not the reader who wants the quote.
- **Personal or identifying detail in a transcript** — a name, an employer, a manager, a team small enough to name, a health, immigration or legal circumstance, a salary figure. Redact per Edge cases, then stop before the file circulates: tell the research owner which participant, which span, and what you replaced it with. Do not decide by yourself that a redaction is sufficient, and do not leave the unredacted transcript quoted anywhere in your working output.
- **Re-identification risk in a small segment.** A segment of three or fewer, or a claim that carries a role plus a company plus a behaviour, identifies a person even under a `P4`. Do not publish that segment's breakdown. Report the segment size and the claim to the research owner, offer the pooled figure, and say plainly that pooling hides a segment finding so nobody mistakes the pooled number for the whole answer.
- **Anything a participant disclosed about a third party** — a named colleague, a manager, a customer, an employee of a vendor. That person consented to nothing and is not in the room. Do not code it, do not quote it, do not paraphrase it into a claim. Tell the research owner it is in the transcript so they can decide whether the transcript itself needs editing before anyone else reads it.
- **A request to attach a real name, e-mail or employer to a participant id.** Refuse and say why: the ids are the anonymisation, and re-linking them after the fact breaks the terms the interview was granted under. Only the research owner holds that mapping, and whether to use it is theirs.
- **A participant withdraws, or a consent record is missing for one transcript.** Drop that transcript, restate the denominator without it, count it in the header (`withdrawn: 1`, `consent not on file: 1`), and recount every claim and sub-denominator that touched it. Do not keep the quotes and drop the id.
- **The distillation will be used to decide something about the participants themselves** — a customer's contract, an employee's role, a vendor's renewal. This skill produces evidence about behaviour for a product decision. Hand it to whoever owns that other decision with the scope stated, and never let a research quote stand in as a performance record or an account note.

## License
MIT
