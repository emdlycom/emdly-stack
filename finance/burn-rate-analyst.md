---
name: burn-rate-analyst
owner: ledgerline
category: Finance
description: Reads a monthly P&L export and reports runway, burn trends and the three line items moving fastest — with the math shown.
version: v2
license: MIT
updated: 2026-08-14
recommended: false
security_checked: true
url: https://emdly.com/skills/ledgerline/burn-rate-analyst
raw: https://emdly.com/raw/ledgerline/burn-rate-analyst.md
install: npx @emdly/cli add ledgerline/burn-rate-analyst
---

# Burn rate analyst

Founders ask "how long do we have?"; the honest answer has three numbers and shows its work. This skill produces exactly that from the accounting export.

## When to use
- Monthly close, on the P&L and the cash balance.
- Before a board update or a fundraising conversation.

## Input
Monthly P&L (revenue, COGS, opex by category) for the last 6–12 months, the current cash balance, and a list of known one-offs (a legal bill, an annual insurance payment) if the team keeps one.

## Definitions (state them in the output)
- **Gross burn** = total cash costs in the month. **Net burn** = gross burn − revenue collected.
- **Run-rate burn** = net burn with one-offs removed and annual items spread over 12 months.
- **Runway (months)** = cash ÷ run-rate net burn. Report three: *flat* (last month), *trend* (3-month linear trend continued), *stress* (revenue −20%, costs +10%).

## Process
1. Separate one-offs. If none are listed, flag any line that is > 3× its 6-month median as a probable one-off and ask.
2. Compute gross, net and run-rate burn per month; show the table.
3. Runway under the three scenarios with the formula and the inputs visible.
4. **Movers:** the three line items with the largest absolute change over 3 months, each with the direction and the share of total burn.
5. One paragraph: what the numbers say, in plain words, without advice.

## Rules
- Show the arithmetic. Every headline number has its inputs next to it.
- Never mix accrual revenue with cash burn without saying so; if only accrual figures exist, say "cash approximation".
- No recommendations ("cut marketing"). Facts, trends, and the questions they raise.
- Currency and units on every number.

## Output format
```
## August 2026 — cash €1 240 000
| month | gross burn | revenue | net burn | run-rate net |
| Jun | 212 k | 88 k | 124 k | 118 k |
| Jul | 231 k | 91 k | 140 k | 121 k (annual insurance 19 k removed) |
| Aug | 226 k | 97 k | 129 k | 129 k |

**Runway:** flat 9.6 mo (1 240 / 129) · trend 8.1 mo (net burn +5.5 k/mo) · stress 6.9 mo (rev 78 k, costs 249 k)

**Movers (3 mo):** contractors +31 k (14% of burn) · cloud +9 k · revenue +9 k

**Read:** burn is rising faster than revenue; the gap is contractors. Runway is under a year in every scenario.
```

## License
MIT
