---
name: burn-rate-analyst
owner: ledgerline
category: Finance
description: Reads a monthly P&L export and reports runway, burn trends and the three line items moving fastest — with the math shown.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/ledgerline/burn-rate-analyst
raw: https://emdly.com/raw/ledgerline/burn-rate-analyst.md
install: npx @emdly/cli add ledgerline/burn-rate-analyst
---

# Burn rate analyst

Founders ask "how long do we have?"; the honest answer has three numbers and shows its work. This skill turns an accounting export into a burn table, three runway scenarios, and the three line items that moved. It is written for a board pack, so every headline number sits next to the inputs that produced it and every convention is labelled as a convention.

## When to use
- Monthly close, on the P&L and the closing cash balance.
- Before a board update, a fundraising conversation, or a hiring decision that adds fixed cost.
- After a month with an unusual payment, to see what it did to run-rate.
- Not for revenue forecasting, not for valuation, not for a 13-week cash flow. Those need a different model.

## Input
1. **Monthly P&L**, 6 to 12 months, with revenue and costs broken out by category. Cash basis if available; accrual if not, and say which.
2. **Closing cash balance** for the latest month, with the date and currency.
3. **Opening cash balance** for the first month in the table. Needed for the reconciliation in step 6.
4. **Known one-offs**, if the team keeps a list: a legal bill, an annual insurance payment, a recruiting fee.
5. **Annual or multi-year prepayments** still being consumed, with the amount and term.
6. **Stress assumptions**, if the board uses its own. Absent that, the defaults in *Definitions* apply and are named as defaults.

## Definitions (state them in the output, every time)

- **Gross burn** = total cash costs in the month.
- **Net burn** = gross burn − revenue collected in the month.
- **Run-rate gross** = gross burn − one-offs paid this month − annual items paid this month + (annual items ÷ term in months). This is the recurring monthly cost, not the cash that left.
- **Run-rate net** = run-rate gross − revenue.
- **Runway (months)** = cash ÷ run-rate net burn, reported under three scenarios:
  - **Flat** — the latest month's run-rate net burn, held constant. `cash ÷ latest run-rate net`.
  - **Trend** — ordinary least squares fit through the last 3 run-rate net burn figures, projected forward one month at a time, consuming cash until it is exhausted. Report the fitted slope and the final partial month. Needs at least 3 months; with fewer, the fit is undefined and the scenario is refused, not approximated.
  - **Stress** — revenue −20%, costs +10%, applied to the latest run-rate month. **These two percentages are a house convention, not a forecast and not derived from this company's history.** They exist so every ledgerline report stresses the same way. If the board uses different ones, use theirs and say so.

Thresholds above are defaults; report the thresholds you used.

## Process

1. **Separate one-offs.** Use the supplied list first. Then screen every category: any month where a line is **more than 3× its own 6-month median** is flagged as a probable one-off. `[judgment — no accounting standard sets this; it is a screen, chosen wide enough that ordinary opex variance does not trip it. It flags for a human; it never reclassifies on its own.]` A flagged line with no confirmation stays **in** run-rate and is marked `unconfirmed one-off`, with the effect of removing it stated.
2. **Spread annual items** over their term. Show the amount removed and the amount added back, per month.
3. **Build the table**: gross burn, revenue, net burn, run-rate net, per month, with the one-off adjustment named in the row.
4. **Runway** under all three scenarios, with the formula and the inputs beside each number.
5. **Movers**: the three cost categories with the largest **absolute** change from the first to the last month of the window, each with direction, the two values, and its share of the latest run-rate gross. Revenue is reported separately; it is not a cost mover.
6. **Reconcile**: `opening cash − sum of net burn over the window = closing cash`. Report the gap. A gap over **1% of closing cash** [house rule] is a stop, not a footnote.
7. **Read**: one paragraph, plain words, what the numbers say. No advice.

## Rules

- Show the arithmetic. Every headline number carries its inputs inline.
- Never mix accrual revenue with cash burn without saying so. Accrual-only input is labelled `cash approximation` on every affected figure.
- No recommendations. Not "cut contractors", not "raise sooner". Facts, trends, and the questions they raise. The board decides.
- Currency and unit on every number. Round to whole thousands in tables; keep one decimal on runway months.
- A negative run-rate net burn (a profitable month) makes runway undefined. Write `runway not applicable — run-rate net burn is negative (−14 k)`. Do not print a large number or an infinity symbol.
- Never fill a missing month by interpolation. A gap in the P&L is reported as a gap.

## Output format

```
## August 2026 — closing cash €1 240 k (cash basis)

| month | gross burn | revenue | net burn | run-rate adjustment | run-rate net |
| Jun | 212 k | 88 k | 124 k | legal bill 6 k removed (on team list); insurance spread +2 k | 120 k |
| Jul | 231 k | 91 k | 140 k | annual insurance 24 k removed, spread +2 k | 118 k |
| Aug | 226 k | 97 k | 129 k | insurance spread +2 k; recruiting fee 11 k left in (unconfirmed) | 131 k |

**Flagged by the 3× screen**
- Jul, insurance €24 k vs 6-month median €2 k = 12.0× → confirmed one-off on the team's list, removed and spread over 12 months.
- Aug, recruiting €11 k vs 6-month median €2 k = 5.5× → no one-offs list supplied for August. Left in run-rate, marked `unconfirmed one-off`. If confirmed, Aug run-rate gross falls 228 k → 217 k, run-rate net 131 k → 120 k, and flat runway rises to 10.3 mo.
- Aug, contractors €37 k vs 6-month median €20 k = 1.85× → under the screen, not flagged.

**Runway**
- Flat: **9.5 mo** — 1 240 ÷ 131.
- Trend: **8.1 mo** — OLS through run-rate net 120, 118, 131 gives slope +5.5 k/mo and a fitted Aug of 128.5 k. Projected months: 134.0, 139.5, 145.0, 150.5, 156.0, 161.5, 167.0, 172.5 → cumulative 1 226.0 k after 8 months; the remaining 14.0 k covers 14.0 ÷ 178.0 = 0.08 of month 9.
- Stress: **7.2 mo** — revenue 97 × 0.80 = 77.6 k; costs 228 × 1.10 = 250.8 k; net 173.2 k; 1 240 ÷ 173.2 = 7.16. (House stress convention: revenue −20%, costs +10%.)

**Movers (run-rate cost, Jun → Aug, absolute)**
| category | Jun | Aug | change | share of Aug run-rate gross (228 k) |
| contractors | 22 k | 37 k | +15 k | 16% |
| recruiting | 0 k | 11 k | +11 k | 5% |
| marketing | 13 k | 4 k | −9 k | 2% |
| cloud | 21 k | 28 k | +7 k | 12% |
| other | 8 k | 2 k | −6 k | 1% |
| payroll | 132 k | 134 k | +2 k | 59% |
| tools | 12 k | 12 k | — | 5% |
| travel | 0 k | 0 k | no recorded activity | 0% |

Revenue (reported separately, not a cost mover): 88 k → 97 k, +9 k (+10.2%).

**Reconciliation:** opening cash 1 Jun €1 633 k − net burn (124 + 140 + 129 = 393 k) = €1 240 k = stated closing balance. Gap €0 k.

**Read:** run-rate net burn fell in July and rose 13 k in August. The rise is contractors and a recruiting fee that may not repeat; marketing fell 9 k over the same period. Revenue grew 9 k across three months while run-rate net burn grew 11 k. Runway is under a year in all three scenarios and under nine months on the trend fit. The August recruiting fee is unconfirmed and moves flat runway by 0.8 months on its own.
```

### The refusal, when the export does not reconcile

```
## August 2026 — NOT REPORTED

Reconciliation fails. Opening cash 1 Jun €1 611 k − net burn (124 + 140 + 129 = 393 k)
= €1 218 k. Stated closing balance is €1 240 k. Gap €22 k, which is 1.8% of closing
cash, over the 1% tolerance.

No runway figure is issued. A €22 k gap moves flat runway between 9.3 and 9.5 months,
and the gap's sign is unexplained: it is consistent with a receipt outside the P&L,
a transfer between accounts, or a missing cost line.

Reconcile the export against the bank statements for Jun–Aug, or state what the €22 k is,
and re-run. Handed to the finance owner.
```

## Edge cases

- **Fewer than 6 months of history.** Report the table and flat runway. Refuse the 3× one-off screen (a median over fewer than 6 points is not the stated screen) and say so. With fewer than 3 months, refuse trend as well: `trend not computed — OLS needs 3 points, 2 supplied`.
- **Fewer than 3 months.** Flat runway only, plainly labelled as one month's figure held constant.
- **No one-offs list.** Run the 3× screen, report every flag as `unconfirmed one-off`, leave them all in run-rate, and state the runway effect of removing them together.
- **Accrual-only P&L.** Every figure is labelled `cash approximation`. Say explicitly that revenue recognised is not revenue collected, and that runway computed this way is optimistic when receivables are growing.
- **Multiple currencies in the export.** Do not convert. Report per currency and stop; the rate and the date are a finance decision.
- **A missing month.** Report the gap, exclude the month from the trend fit, and say the fit used n points, not 3.
- **Categories change name or split mid-window.** Movers compare like to like. If a category cannot be traced across the window, report it as `not comparable — renamed or split` and exclude it from movers.
- **Revenue exceeds costs in the latest month.** Runway is not applicable; see *Rules*. Still report the table, movers and reconciliation.
- **Export larger than ~40 categories.** Aggregate to the top 12 by latest-month value plus an `other` line, and say which categories are inside `other`.
- **Cash balance is a single number with no date.** Ask for the date. A balance from a different day than the P&L close makes the reconciliation meaningless.

## Stop and hand back

Halt and name the owner. Do not issue a runway figure through any of these.

- **The export does not reconcile** to the stated cash balance within 1% of closing cash. Report the gap and both sides of the arithmetic. Reconcile before reporting. Finance owner decides.
- **The cash balance includes restricted, escrowed, or customer-held funds**, or counts an undrawn credit facility as cash. Report the components you were given and ask which are spendable. CFO or finance owner decides.
- **An accrued but unpaid liability** appears in the input — deferred payroll, unremitted VAT or sales tax, an unpaid tax bill. It is not in gross burn but it is a claim on the cash. Name it and stop.
- **Flat runway under 3 months** [house rule; below one quarter the monthly figures stop being the useful unit and a weekly cash plan is]. Report the numbers, do not model scenarios, hand to the founders and the board.
- **Financing in flight** — a term sheet, a bridge, a facility being negotiated. Runway under one balance and runway under another are different documents. A human states which one the report assumes.
- **The one-off flags change runway by more than one month** and none is confirmed. The classification is now the story. Get the list confirmed first.
- **You are asked what to cut, whether to hire, or when to raise.** Out of scope by the third rule. Report the facts and hand back.

## License
MIT
