---
name: client-value-score
owner: launifycorp
category: Data &amp; analytics
description: Works out what a client is actually worth from whatever systems the agent can reach — discovering those first, then proposing the search plan. Six measures that stand alone, each with its coverage, and a composite only when you supply the weights. Revenue is not value, size is not value, and concentration is reported as risk.
version: v1
license: MIT
updated: 2026-08-31
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/client-value-score
raw: https://emdly.com/raw/launifycorp/client-value-score.md
install: npx @emdly/cli add launifycorp/client-value-score
---

# Client value score

Works out what a client is actually worth to the business, from whatever systems this agent can actually reach. It starts by finding out what those are, because a scoring method designed against data you do not have produces a confident number built on a third of the picture.

The number most people mean by "client value" is revenue to date, and it is the wrong number three ways. It looks backward. It ignores what the client costs to serve, which is where the difference between a good and a bad account usually lives. And it treats size as value, so the client who is 40% of your revenue scores highest, when what that actually is, is your largest single risk.

So this produces **six measures that mean something on their own**, each with a stated coverage, and a composite only when the person asking supplies the weights. It does not invent weights, and it does not average over a hole in the data.

## When to use

- Deciding where account management time goes next quarter.
- Before a renewal, a price change, or a decision to stop working with someone.
- Working out which clients the business is quietly subsidising.
- After a year of trading, to see whether the mix is what you thought it was.
- Building the measure once, so it can be re-run on a schedule with the same method.

Not for: pipeline and deal scoring on prospects who are not yet clients, forecasting future revenue, or credit assessment.

## Pass 0 — Find out what you can actually reach

**Run this first, every time, and report it before anything else.** Do not assume a fixed set of tools. This skill runs in different hosts — Claude, openclaw, hermes, or something else — and each one exposes a different set, which changes what can be measured and therefore what may be claimed.

Enumerate the tools actually present in this session. Then map them onto what each can supply:

| what you need | typically comes from |
|---|---|
| revenue, invoices, payment dates | accounting or invoicing system, payment processor, a finance sheet |
| direct cost, hours, delivery effort | time tracking, project tool, payroll allocation, a sheet somebody maintains |
| contract terms, renewal dates, notice | CRM, a contracts folder, a sheet |
| support load | helpdesk, shared mailbox, ticket system |
| relationship and meeting load | calendar, CRM activity, email |
| what the client publishes about itself | their website, a CMS if you run it for them |

Then write the map out loud:

```
AVAILABLE
  Google Sheets     — "Fakturace 2024-2026", "Klienti", "Výkazy hodin"
  CMS (WordPress)   — 3 sites we host, per-client
  Email             — shared mailbox, read-only
NOT AVAILABLE
  Accounting system — no connector. Revenue must come from the sheet, which
                      is a copy and may lag the accounting truth.
  Time tracking     — only the hours sheet, filled by hand, coverage unknown
                      until the measures below report it.
```

**A measure whose source is missing is not computed.** It is not estimated, not proxied, and not set to zero. Say which measure died with which missing source, in the report, at the top.

**Propose the search plan before running it**, and get agreement. Reading a client's data across five systems is a real act, and the person asking may know that one of those sheets is abandoned.

## Pass 1 — Resolve the client to one entity

The single largest source of wrong numbers in this work, and the step almost everyone skips.

The same client is `ACME s.r.o.` in the invoicing sheet, `Acme` in the CRM, `acme.cz` in the CMS, and three different email domains in the mailbox. Every unmatched record is revenue or cost that silently disappears from the score.

Build the identity set explicitly, from strongest identifier to weakest, and show your work:

1. A company registration number, VAT number, or an internal client ID that appears in more than one system. Strong.
2. An exact domain match on email or website. Strong.
3. A bank account or a payment reference that recurs. Strong.
4. A normalised name match. Weak, and it merges `ACME s.r.o.` with `ACME Group a.s.` if you let it.

Report the set, the records matched to it per system, and **the records you could not place**. A client with 14 invoices matched and 3 unplaced has a number that is wrong by up to 3 invoices, and the reader needs that.

**Never merge two entities on a name alone.** Where it is ambiguous, present both and ask.

Groups, subsidiaries and holding structures are a decision, not a lookup. Ask whether the score is for the legal entity or the group, and say which you used.

## Pass 2 — Fix the window

You cannot compare a client of three months against one of five years by totalling either.

Pick one window and apply it to everyone: the last complete 12 months is the usual choice. State it. For clients whose relationship is shorter than the window, **annualise nothing** — report the actual period and mark them as short-tenure, because a client three months in with one big project extrapolates to a number that is fiction.

Report every measure against the same window, and say what the window is at the top of every output.

## The six measures

Each one stands alone. Each one carries its coverage. None of them is "the score".

### 1 · Contribution

Revenue minus the direct cost of serving them, over the window. **Not revenue.** A client at 300k and 4% margin is worth less than one at 80k and 40%, and revenue rankings invert that.

If direct cost is not available, say `contribution not computed` and report revenue as revenue. Do not present revenue and call it value.

### 2 · Cost to serve

The measure everyone omits and the one that most often changes the answer. Whatever the tools give: hours logged, support tickets, meetings, email volume, number of scope changes, rounds of revision, out-of-hours contact.

Report it in whatever units you actually have, per unit of revenue: hours per 10k, tickets per month. Never convert hours to money at a rate you invented; if there is a rate, say whose it is.

### 3 · Payment behaviour

Median days from invoice to payment, against the agreed terms. Count of invoices paid late, disputed, or written off. Whether they have ever needed chasing more than once.

This is the most predictive of the six and the easiest to compute, because invoice and payment dates are usually the cleanest data in the business.

### 4 · Direction

Growing, flat, or shrinking, over at least three comparable periods. **Two points is not a trend.** With fewer than three, say so and report the values.

Direction beats level for a decision about where next quarter's attention goes.

### 5 · Stability

Recurring or one-off. Contract or not. Notice period. How many people at the client know you, and whether the relationship survives one of them leaving. A single point of contact is a stability finding, not a convenience.

### 6 · Concentration

This client as a share of total revenue and of total delivery capacity in the window.

**Report concentration as a risk, never as value.** A client at 38% of revenue is not your best client, it is the thing that ends the business if it leaves. Any output that ranks by concentration has the sign backwards.

### And what stays qualitative

Reference value, strategic fit, whether the work is interesting, whether they pay on time because they are organised or because they are frightened. These are real and they are not measurable from these systems. Write them as a short paragraph, name them as judgement, and keep them out of the arithmetic.

## The composite

Only when the person asking supplies the weights.

**This skill does not invent weights.** Weighting is a statement about what the business values, and inventing one produces a ranking that looks objective and encodes an assistant's guess. Ask for the weights, put them in the output, and re-run when they change.

Normalise each measure across the client set before weighting, say which normalisation you used, and print the arithmetic. A composite whose components cannot be read back is a number nobody can argue with, which is the opposite of useful.

**Never produce a composite when any weighted measure is uncomputed.** Averaging over a hole silently sets it to average. Report the component scores and say the composite is unavailable until that source exists.

> Thresholds above are defaults; report the thresholds you used.

## Coverage

Every measure reports the share of the window for which the source actually had data.

- Below **60% coverage**, the measure is reported with its coverage and excluded from any composite. [judgment — below that the records that happen to exist are a self-selected sample, usually the ones somebody was already tracking closely]
- **Coverage is per measure, not per client.** A client can have clean invoicing and no time records at all.
- A month with no records is not a month of zero. Say `no data` and mean it.

## Rules

- **Say the source of every number**, by system and field. A figure whose provenance is not stated cannot be checked, and this output will be quoted in a meeting.
- **Never impute.** No filling gaps with an average, no extrapolating a short tenure, no assuming last year looked like this year.
- **Revenue is not value, and size is not value.** Two separate rules because they fail separately.
- **Rank by nothing without saying by what.** Every ordered list carries the measure it is ordered on.
- **Keep the qualitative out of the arithmetic.** Write it, name it as judgement, do not score it.
- **The score describes the past.** It is a measurement of a window that has closed, not a prediction. See Stop 6.
- **Do not copy personal data out of the systems it lives in.** Names of individual contacts, message contents, anything about a person rather than an account, stay where they are. The output is about the client organisation.

## Output format

```
CLIENT VALUE · ACME s.r.o. · window 2025-09-01 to 2026-08-31 (12 complete months)

SOURCES REACHED
  Sheets     "Fakturace 2024-2026" (invoices, payments), "Výkazy hodin" (hours)
  CMS        WordPress, 1 site, hosted by us
  Email      shared mailbox, read-only
NOT REACHED
  Accounting system — no connector. Revenue is from the invoicing sheet, which
  is a hand-maintained copy. Treat as indicative, not as the ledger.
  Helpdesk — none in use. Support load approximated from mailbox threads and
  labelled as such below.

IDENTITY
  Matched on IČO 27082440, appearing in Sheets and in the contract PDF.
  Sheets "Fakturace": 31 invoices · Sheets "Výkazy": 148 rows · Email: 212 threads
  UNPLACED: 2 invoices to "ACME Group a.s." — a different IČO. Not merged.
  Ask whether the score should be the entity or the group; this is the entity.

MEASURES

1 · Contribution            412 000 CZK revenue · 268 000 direct cost
                            144 000 CZK contribution · 35.0% margin
                            coverage: revenue 100%, cost 78% (hours missing for
                            Nov and Dec 2025) — contribution is a floor, not exact

2 · Cost to serve           148 h logged · 3.6 h per 10 000 CZK revenue
                            approx 18 support threads, from mailbox labels
                            coverage 78%. Support load is approximated, not counted.

3 · Payment behaviour       median 41 days to pay against 14-day terms
                            27 of 31 invoices late · 0 disputed · 0 written off
                            longest 96 days · coverage 100%
                            This is the strongest signal in the file.

4 · Direction               4 quarters in the window, 3 comparable:
                            Q4/25 57k — excluded, contains a one-off migration
                            Q1/26 96k → Q2/26 118k → Q3/26 141k
                            growing, +47% across the three (96 → 141)
                            57 + 96 + 118 + 141 = 412k, ties to measure 1
                            coverage 100%

5 · Stability               no contract on file · no notice period
                            one named contact, no second relationship
                            work is project-based, not recurring
                            coverage: contract NOT FOUND, so this is partly
                            an absence of evidence

6 · Concentration           9.1% of revenue in the window (412k of 4 530k)
                            11.2% of logged delivery hours
                            Reported as risk. Below the level where losing them
                            would be structural, on your own numbers.

QUALITATIVE — judgement, not scored
They are the reference on our site and have sent two enquiries that became
clients. That is worth something the six measures above cannot see, and it does
not go into the arithmetic.

COMPOSITE — not produced
No weights supplied. Say what the business values and in what proportion, and
I will compute it, print the normalisation, and show the arithmetic.
Note that measure 2's coverage is 78%, above the 60% floor, so it would be
included; if you would rather exclude it until the hours are complete, say so.

WHAT WOULD CHANGE THE ANSWER
- The two missing months of hours. Contribution is currently a floor.
- A contract. Measure 5 is mostly reporting that nothing was found.
- Whether ACME Group a.s. belongs in this score.
```

The refusal:

```
NOT SCORED

I can reach the invoicing sheet and nothing else.

That gives revenue and payment dates, which is two of six measures. What it
does not give is what this client costs to serve, and that is the number that
decides whether a client at 412k is a good one or a bad one.

I can report revenue and payment behaviour, clearly labelled as two measures
out of six, and that is genuinely useful on its own. What I will not do is
present it as a value score, because ranking clients by revenue is the specific
mistake this method exists to avoid.

Tell me which it is, and where the hours or cost data lives if it exists
anywhere — a sheet, a project tool, an accountant's export.
```

## Edge cases

- **No tools connected at all.** Say so and stop. Nothing here works from memory.
- **A client shorter than the window.** Report the actual period, mark short-tenure, annualise nothing.
- **A client with no invoices in the window** but an active relationship. That is a finding, not a zero. They may be pre-invoice, on retainer paid elsewhere, or dormant.
- **A sheet that is a copy of the accounting system.** Say it is a copy. It will disagree with the ledger, usually on timing, and finance will notice.
- **Two sheets that disagree.** Report both figures and the gap. Do not pick one silently, and do not average them.
- **Currency mixed across records.** Convert only with a stated rate and date, and show the original alongside. Never convert at today's rate for a transaction from last year without saying so.
- **VAT.** State whether figures are inclusive or exclusive and be consistent. A margin computed on one and a cost on the other is wrong by the VAT rate.
- **Refunds, credit notes and write-offs.** They reduce revenue in the period they belong to, not the period they were issued. Say which convention you applied.
- **Hours logged against the wrong client**, or a catch-all "internal" bucket that clearly contains client work. Report the size of the bucket rather than distributing it.
- **A client who is one person** — a freelancer, a sole trader. See Stop 1. The measures still work; the framing and the permitted use change.
- **Someone asks for a league table of everyone.** Fine, but it carries the measure it is sorted on, the window, and the per-client coverage. A table sorted on a composite whose components have 40% coverage for half the rows is worse than no table.
- **A CMS that only tells you the site exists.** That is not a value signal. Say what you looked for and found nothing, rather than dressing "we host their site" as data.

## Stop and hand back

1. **The client is a natural person and the score will drive how they are treated.** GDPR Article 22(1): *"The data subject shall have the right not to be subject to a decision based solely on automated processing, including profiling, which produces legal effects concerning him or her or similarly significantly affects him or her."* A score that decides pricing, service level, or whether the relationship continues is capable of being exactly that. Produce the measures, hand the decision to a person who takes responsibility for it, and say plainly that the decision must not be made by the number alone. Where the exceptions in 22(2) apply, 22(3) still requires the ability to obtain human intervention, express a point of view, and contest the outcome.
2. **Anything that proxies a protected characteristic.** Nationality, language, the part of the country an address is in, a name's origin, the gender of a contact. These correlate with things that must not drive the answer, and a weighting that includes them launders discrimination through arithmetic. Refuse the input, not just the output.
3. **A score used to end a relationship, cut service, or change a price** without a person reviewing the underlying data. The measures are inputs to that conversation. They are not the conversation.
4. **Data pulled across a purpose boundary.** Invoice records collected to get paid, support messages collected to answer questions, and calendar entries collected to run a diary were not collected to profile the client. Using them here may be fine and may not; it is a question for whoever owns the data protection position, and it should be asked before the pull, not after.
5. **Personal data about individuals inside the client.** Who replies fastest, who is difficult, how often someone emails out of hours. That is monitoring people, and it is not what this is. Keep the output at the level of the organisation.
6. **Any request to present this as a prediction.** Lifetime value, likelihood to churn, expected future revenue. This method measures a window that has closed. A forecast needs a model, validation, and someone accountable for it.
7. **A client who is also an employer, a landlord, a lender, or otherwise holds power over the person being scored.** The framing inverts and so do the obligations.

## Sources

- GDPR **Article 22**, *Automated individual decision-making, including profiling* — 22(1) quoted above; 22(2) sets out the contractual-necessity, legal-authorisation and explicit-consent exceptions; 22(3) requires human intervention, the right to express a point of view and the right to contest; 22(4) restricts reliance on Article 9 special-category data.
- Everything else here is method, not standard. The 60% coverage floor, the three-period minimum for a trend and the choice of six measures are this skill's judgement, marked as such, and a business with better data should set its own.

## License

MIT
