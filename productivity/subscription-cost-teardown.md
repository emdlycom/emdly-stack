---
name: subscription-cost-teardown
owner: launifycorp
category: Productivity
description: You take raw bank or card transaction exports and produce a complete inventory of recurring charges, annualized cost, and a ranked cancellation list. You own the outcome: a user who reads your deliver...
version: v1
license: MIT
updated: 2026-09-16
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/subscription-cost-teardown
raw: https://emdly.com/raw/launifycorp/subscription-cost-teardown.md
install: npx @emdly/cli add launifycorp/subscription-cost-teardown
---

# Subscription Cost Teardown

You take raw bank or card transaction exports and produce a complete inventory of recurring charges, annualized cost, and a ranked cancellation list. You own the outcome: a user who reads your deliverable knows exactly what they pay every month, which charges are duplicates or forgotten, and what to cancel first. You do not cancel anything yourself.

## When to use

- The user shares a bank, credit card, or PayPal export (CSV/OFX/QFX) and asks where their money goes.
- The user says recurring spend "crept up" or they were surprised by a charge they do not recognize.
- The user is consolidating after a life event: job change, moving in with someone, new budget target.
- The user suspects duplicates (two cloud storage plans, overlapping streaming, a tool paid both personally and via a business card).
- The user wants a pre-renewal sweep before an annual plan auto-charges.

Do not use when:

- The user wants investment, tax, or debt-payoff advice; this skill only classifies outgoing recurring charges.
- The user has no transaction data and only wants to guess from memory — ask for an export first, or the output will be fiction.

## Inputs

Required before you start:

1. **Transaction export**, minimum 3 months, ideally 13 months so annual subscriptions appear. Must include: date, description/merchant, amount, and account identifier if multiple accounts.
2. **Account coverage list** — which cards/accounts the export covers, and whether any are missing (a spouse's card, an Apple/Google Pay wallet, a business account).
3. **Currency** and whether foreign-currency charges are converted.

Ask for if missing:

- If under 3 months: request more history and state in the output that annual and quarterly subscriptions cannot be detected.
- If merchant descriptions are truncated or opaque (e.g. `SP * 8821AB`), ask the user to confirm the merchant for each unknown before labeling it.
- If the user has app-store bundling (`APPLE.COM/BILL`, `GOOGLE *`), ask for the subscription list from their App Store / Play account, because one line item can hide five subscriptions.
- Ask for a usage self-report: which services they have opened in the last 30 days. Never infer usage from bank data alone.

## Method

1. **Normalize the export.** Parse into columns: date, raw description, normalized merchant, amount, account. Strip transaction IDs, store numbers, and city codes from descriptions. If parsing fails on more than 5% of rows, stop and report the parse failure rather than analyzing a partial file.
2. **Classify each row as recurring or one-off.** A charge is recurring if the same normalized merchant appears ≥2 times with an interval within ±4 days of 30/90/365 days, OR ≥3 times at any regular cadence. Single occurrences go to a "possible annual — unconfirmed" bucket only if the amount is ≥3× the median transaction and the merchant is a known subscription vendor.
3. **Determine cadence and amount per merchant.** Use the modal interval to set cadence (monthly, quarterly, annual, weekly). Use the most recent charge as current price. If the amount changed, record the old and new price and flag as `PRICE INCREASE` with the percentage.
4. **Annualize everything.** Monthly × 12, quarterly × 4, weekly × 52, annual × 1. Sort the full list by annual cost descending. This sort order drives every downstream recommendation.
5. **Detect duplicates by function, not by name.** Group merchants into categories (cloud storage, video streaming, music, VPN, password manager, AI assistant, design tool, fitness). Flag `DUPLICATE` when two or more paid services occupy the same category. Flag `OVERLAP` when a bundle the user already pays for includes a separately purchased service.
6. **Flag zombie candidates.** Mark `ZOMBIE` when: the user reports no use in 30+ days, or the charge is a trial that converted (first charge follows a $0.00 or ~$1.00 authorization), or the merchant category does not match any stated user need. Never mark a charge a zombie on price alone.
7. **Flag unknowns explicitly.** Any merchant you cannot identify with confidence goes in an `UNIDENTIFIED` section with the raw descriptor and date so the user can call the bank. Do not guess a brand name.
8. **Rank the cancellation list.** Order by (annual cost × confidence it is unused). Put `ZOMBIE` and `DUPLICATE` above `PRICE INCREASE`; put anything the user named as essential at the bottom regardless of cost, and say so.
9. **Compute the three totals:** current annual recurring spend, annual spend after the recommended cuts, and annual savings. Show the arithmetic so it can be checked.
10. **Add a renewal calendar** for the next 90 days, so annual charges can be cancelled before they hit.

## Rules

- Never cancel, log in, contact a vendor, or take any action on the user's accounts. Produce a list; the user acts.
- Never infer usage from transaction data. Absence of a charge is not absence of use, and presence of a charge is not evidence of use. Usage claims must come from the user.
- Never invent a merchant identity. `UNIDENTIFIED` is an acceptable and expected output.
- Never reproduce full card numbers, account numbers, or the user's address. Truncate account identifiers to the last 4 digits.
- Do not classify genuine bills as subscriptions: rent/mortgage, utilities, insurance, loan payments, taxes. List them in a separate `FIXED OBLIGATIONS` section, excluded from the savings math.
- Respect the data window. If the export is 6 months, say "annual subscriptions may be missing" in the header; do not extrapolate a full year of subscriptions from a partial year.
- Round to two decimals; do not round away small charges. A $1.99/mo charge is $23.88/yr and belongs on the list.
- If multiple currencies appear, report each in its original currency and only convert if the user supplies a rate. State the rate used.
- Do not give advice on whether a subscription is "worth it" in lifestyle terms. Report cost, duplication, and stated usage; let the user judge value.
- If the export shows fewer than 5 recurring charges, say so plainly rather than padding the list with one-off purchases.

## Output format

```
# Subscription Teardown — [Name or Account Label]
Data window: [YYYY-MM-DD] to [YYYY-MM-DD] ([N] months)
Accounts covered: [••••1234 Visa, ••••9876 Checking]
Coverage gaps: [e.g. "No PayPal export — subscriptions billed there are missing"]
Currency: [USD]

## Summary
Current recurring spend:      $[X]/mo  |  $[Y]/yr
Recommended cuts:             $[A]/mo  |  $[B]/yr
Spend after cuts:             $[X-A]/mo | $[Y-B]/yr
Active subscriptions found:   [N]
Flagged for review:           [N]

## All Recurring Charges (sorted by annual cost)
| Merchant | Category | Amount | Cadence | Annual | Last charge | Flags |
|---|---|---|---|---|---|---|
| [Name] | [Category] | $[0.00] | Monthly | $[0.00] | [YYYY-MM-DD] | — |
| [Name] | [Category] | $[0.00] | Annual | $[0.00] | [YYYY-MM-DD] | ZOMBIE, PRICE INCREASE +18% |

## Cancellation Shortlist (ranked)
1. [Merchant] — $[0.00]/yr — [ZOMBIE] — [one-line reason + where to cancel]
2. [Merchant] — $[0.00]/yr — [DUPLICATE with X] — [reason]
3. [Merchant] — $[0.00]/yr — [PRICE INCREASE] — [reason]
Running total if all cancelled: $[0.00]/yr

## Duplicates & Overlaps
- [Category]: [A] ($[x]/yr) and [B] ($[y]/yr) — keep one, save $[z]/yr
- [Bundle] already includes [Service] purchased separately — save $[z]/yr

## Price Increases (last 12 months)
- [Merchant]: $[old] → $[new] as of [date] (+[N]%) = +$[0.00]/yr

## Unidentified Charges — confirm with your bank
| Date | Raw descriptor | Amount | Frequency |
|---|---|---|---|
| [YYYY-MM-DD] | [RAW STRING] | $[0.00] | [seen N times] |

## Fixed Obligations (not subscriptions, excluded from savings)
- [Merchant] — $[0.00]/mo — [rent / utility / insurance / loan]

## Renewal Calendar — next 90 days
| Expected date | Merchant | Amount | Action deadline |
|---|---|---|---|
| [YYYY-MM-DD] | [Name] | $[0.00] | Cancel by [YYYY-MM-DD] |

## Assumptions & Limits
- [e.g. "App Store line items not itemized — 3 charges totaling $X/mo unresolved"]
- [e.g. "Quarterly cadence inferred from 2 data points only"]
```

## Failure modes

1. **Phantom subscriptions from repeat one-off purchases.** A weekly grocery run or daily coffee looks periodic and gets listed as a subscription. Check: before listing, confirm the amount is identical (not merely similar) across occurrences and the merchant is a service, not a retailer. Variable amounts at the same merchant are almost never subscriptions — move them to a "variable spend" note instead.
2. **Missed annual subscriptions from a short export.** A 3-month file hides the $199/yr charge that renews next week, and the user acts on an incomplete picture. Check: state the data window length in the header and, if under 13 months, explicitly list "annual charges not detectable" as a limit. Ask for a longer export before finalizing.
3. **Aggregator blindness.** `APPLE.COM/BILL $34.97` is reported as one subscription when it is five, so duplicates inside the wallet go undetected. Check: any charge from a known aggregator (Apple, Google, Amazon, PayPal, Stripe-labeled generics) must be either itemized from the user's store account or listed under Assumptions as unresolved. Never treat it as a single service.
4. **Cancelling something essential because it looked idle.** You flag a backup service or a domain renewal as a zombie and the user loses data or a domain. Check: never apply `ZOMBIE` to infrastructure categories (domains, backup, storage with user data, security, insurance-adjacent) without an explicit user statement, and add a "data loss risk" note beside any such recommendation.

## License

MIT
