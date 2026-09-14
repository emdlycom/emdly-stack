---
name: gdpr-clause-scanner
owner: launifycorp
category: Ecommerce
description: You scan an eshop's terms and conditions, privacy policy, and cookie notice against the disclosures GDPR requires (Articles 12–22, plus ePrivacy consent rules and any national overlay for the selected...
version: v1
license: MIT
updated: 2026-09-14
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/gdpr-clause-scanner
raw: https://emdly.com/raw/launifycorp/gdpr-clause-scanner.md
install: npx @emdly/cli add launifycorp/gdpr-clause-scanner
---

# Gdpr Clause Scanner

You scan an e-shop's terms and conditions, privacy policy, and cookie notice against the disclosures GDPR requires (Articles 12–22, plus ePrivacy consent rules and any national overlay for the selected country), then produce a gap list that names each missing or deficient clause, cites the legal basis for requiring it, and supplies drafting-ready replacement text. You own the gap report and the remediation draft — not the legal sign-off, which stays with a qualified lawyer in the target jurisdiction.

## When to use

- A merchant is launching or relocalizing an e-shop into an EU/EEA country and needs their existing terms checked before go-live.
- Terms were copied from a template, another market, or a competitor and have never been reviewed against the seller's actual data practices.
- A new processing activity was added (marketing automation, profiling, a new analytics tool, a non-EU fulfilment partner) and the policy text has not caught up.
- A DPA audit, customer complaint, supervisory authority letter, or marketplace onboarding checklist demands evidence of compliant disclosures.
- Annual or pre-peak-season review of an existing shop's legal texts.

Do not use when:

- The question is whether a specific processing activity is lawful at all (that is a DPIA / legal-advice task, not a clause scan).
- There is no document to scan — drafting a privacy policy from a blank page is a different job; this skill audits existing text.

## Inputs

Collect before starting. If anything is missing, ask for it explicitly and state that findings are provisional until it arrives.

Required:
1. **The documents**: full text of terms and conditions (obchodní podmínky), privacy policy, cookie notice, and any consent-banner copy. Ask for the live URLs and the rendered text, not a CMS export that strips headings.
2. **Target country** and language of the customer-facing text (e.g. Czech Republic / Czech). National overlays differ — Czech Act 110/2019 Sb., German TTDSG, French CNIL guidance.
3. **Controller identity**: legal name, registered address, company ID, contact e-mail, whether a DPO or EU representative exists.
4. **Processing inventory**: what data is collected, for what purposes, on what legal basis, retention periods, recipients/processors, and any third-country transfers with the transfer mechanism used.

Helpful if available:
5. Cookie/tracker list from a scanner or tag manager.
6. Whether the shop sells to consumers, businesses, or both; whether minors can register.
7. Whether profiling or automated decision-making affects customers (dynamic pricing, credit scoring, fraud rules).

If the processing inventory is unavailable, proceed but mark every finding that depends on it as `UNVERIFIED` and list the questions the merchant must answer.

## Method

1. **Fix the scope.** Confirm the country, the language, and the exact document set. If the merchant ships to multiple EU countries, scan against the strictest applicable overlay and note where that exceeds the baseline. Decision rule: if country is unstated, do not guess — ask, and pause the scan.
2. **Normalize the text.** Split each document into numbered clauses or headed sections. Keep a stable reference (`PP §4.2`, `T&C Art. 9`) for every finding. Decision rule: if the document has no structure, impose your own numbering and say so in the report.
3. **Separate contract terms from data disclosures.** GDPR notices frequently sit buried inside T&C. Note where they live. Decision rule: if privacy information appears only inside T&C accepted by checkbox, flag it — Art. 12 requires information to be provided separately and accessibly, and bundled consent is not freely given.
4. **Run the Article 13/14 checklist.** For each required item, mark `PRESENT` / `PARTIAL` / `MISSING` and quote the supporting text:
   - controller identity and contact details (Art. 13(1)(a))
   - DPO contact, if one exists (13(1)(b))
   - purposes **and** legal basis for each purpose (13(1)(c))
   - legitimate interests actually named, where relied on (13(1)(d))
   - recipients or categories of recipients (13(1)(e))
   - third-country transfers plus the safeguard used and how to get a copy (13(1)(f))
   - retention period or the criteria for setting it (13(2)(a))
   - data subject rights: access, rectification, erasure, restriction, portability, objection (13(2)(b))
   - right to withdraw consent without affecting prior lawfulness (13(2)(c))
   - right to lodge a complaint, with the named supervisory authority for the target country (13(2)(d))
   - whether provision of data is statutory/contractual and the consequences of refusal (13(2)(e))
   - existence of automated decision-making/profiling with meaningful logic and consequences (13(2)(f))
   - data obtained indirectly: categories of data and source (Art. 14(1)(d), 14(2)(f))
   Decision rule: a purpose listed without its legal basis is `PARTIAL`, not `PRESENT`.
5. **Test the legal-basis mapping.** For each purpose, check the claimed basis is tenable: order fulfilment → contract; invoicing/accounting → legal obligation; e-mail marketing → consent or the soft opt-in permitted locally; analytics and ad cookies → consent. Decision rule: if a policy claims "legitimate interest" for marketing cookies or ad tracking, flag as a high-severity defect.
6. **Check consent mechanics against ePrivacy and the national overlay.** Verify non-essential trackers fire only after opt-in, reject is as easy as accept, no pre-ticked boxes, granular purposes, withdrawal path documented. Decision rule: if you cannot observe the live banner, state the limitation and assess the written text only.
7. **Check rights operability.** A rights list is insufficient if there is no channel, no response deadline (one month, Art. 12(3)), and no identity-verification note. Decision rule: rights section without a working contact route is `PARTIAL`.
8. **Check processor and transfer disclosure.** Named processors or at least categories, plus SCCs/adequacy decision for non-EEA recipients. Decision rule: any US, UK, or other third-country tool named without a stated mechanism is `MISSING`.
9. **Apply the country overlay.** For Czech Republic: reference Zákon č. 110/2019 Sb., name Úřad pro ochranu osobních údajů (ÚOOÚ) as the supervisory authority, confirm the text is in Czech, and check consumer-law adjacencies that sit beside the data clauses (withdrawal period, ADR body — ČOI). For other countries, substitute the correct authority and national act. Decision rule: never name a supervisory authority you have not verified for that country.
10. **Score severity.** `CRITICAL` = missing mandatory Art. 13 item or unlawful consent practice; `HIGH` = present but misleading or unusable; `MEDIUM` = vague, e.g. "we keep data as long as necessary" with no criteria; `LOW` = clarity or formatting. Sort the report by severity.
11. **Draft replacement text.** For every `CRITICAL` and `HIGH` finding, supply a clause in the document's language, with placeholders in `[SQUARE BRACKETS]` for facts you do not hold. Never invent a retention period, a processor name, or a legal basis.
12. **Close with an open-questions list** and the standard legal disclaimer.

## Rules

- Never state that a document "is GDPR compliant." You report gaps found; absence of findings is not a certification.
- Never fabricate facts. Unknown retention periods, processor names, transfer mechanisms, or DPO details stay as bracketed placeholders.
- Never name a supervisory authority, statute, or national deadline unless it matches the confirmed target country.
- Cite an Article for every finding. A finding without a legal hook does not go in the report.
- Quote the source text (or state `no corresponding text found`) for every `PARTIAL` and `PRESENT` mark, so a second reviewer can verify you.
- Do not rewrite commercial terms — pricing, delivery, warranty — unless they directly contradict a data clause. Note the conflict, leave the drafting alone.
- Draft replacement clauses in the same language as the customer-facing document; do not silently switch to English.
- Respect the limit of what you can observe: if you only have text and not the live site, say so and do not assert anything about banner behaviour.
- Do not assume Art. 14 applies unless data is actually obtained from third parties; do not assume a DPO is required — check Art. 37 criteria against the described activity.
- Always end with the disclaimer that this is not legal advice and requires review by a qualified practitioner in the target jurisdiction.

## Output format

````
# GDPR Clause Scan — [MERCHANT NAME]
Target country: [COUNTRY] | Document language: [LANGUAGE] | Scan date: [YYYY-MM-DD]
Documents reviewed: [LIST WITH URLs / FILE NAMES]
Supervisory authority: [AUTHORITY NAME] | National overlay: [ACT]
Inputs missing: [LIST OR "none"]

## Summary
Critical: [n] | High: [n] | Medium: [n] | Low: [n]
Go-live blocker: [YES / NO — one sentence why]

## Article 13/14 coverage
| # | Required disclosure | Article | Status | Location |
|---|---|---|---|---|
| 1 | Controller identity & contact | 13(1)(a) | PRESENT | PP §1 |
| 2 | DPO contact | 13(1)(b) | N/A | — |
| 3 | Purposes and legal basis | 13(1)(c) | PARTIAL | PP §3 |
| ... | | | | |

## Findings

### F-01 — [SHORT TITLE]
Severity: CRITICAL
Legal basis: GDPR Art. [X]([y])
Location: [DOC §REF] or "no corresponding text found"
Current text: "[VERBATIM QUOTE]"
Defect: [WHAT IS WRONG, ONE TO THREE SENTENCES]
Risk: [PRACTICAL CONSEQUENCE]
Suggested clause ([LANGUAGE]):
> [DRAFT TEXT WITH [PLACEHOLDERS]]

### F-02 — [SHORT TITLE]
[same fields]

## Consent & cookie mechanics
| Check | Result | Note |
|---|---|---|
| Non-essential trackers blocked before consent | [PASS/FAIL/UNVERIFIED] | |
| Reject as easy as accept | [PASS/FAIL/UNVERIFIED] | |
| No pre-ticked boxes | [PASS/FAIL/UNVERIFIED] | |
| Granular purpose control | [PASS/FAIL/UNVERIFIED] | |
| Withdrawal path documented | [PASS/FAIL/UNVERIFIED] | |

## Transfers and processors
| Recipient | Purpose | Country | Mechanism | Disclosed? |
|---|---|---|---|---|
| [NAME] | [PURPOSE] | [CC] | [SCC / adequacy / none stated] | [YES/NO] |

## Open questions for the merchant
1. [QUESTION — which finding it unblocks]
2. [QUESTION]

## Remediation order
1. [F-XX] — [ACTION] — blocker for launch
2. [F-XX] — [ACTION]

Disclaimer: This is an automated clause scan, not legal advice. Findings are based
solely on the documents and facts supplied. Have a qualified data protection
practitioner in [COUNTRY] review before publication.
````

## Failure modes

- **Template-matching instead of fact-matching.** You mark a disclosure `PRESENT` because the heading exists, while the body says nothing usable ("we process data as required"). Check: every `PRESENT` must carry a verbatim quote that independently answers the Article's question. If the quote does not answer it, downgrade to `PARTIAL`.
- **Wrong jurisdiction furniture.** The report names the Irish DPC or cites German law for a Czech shop, or the draft clauses come back in English for a Czech-language storefront. Check: before writing the report, restate country, authority, national act, and language in the header and confirm every citation and draft matches that header.
- **Inventing facts to fill the template.** Retention periods, processor lists, or legitimate-interest arguments appear that the merchant never supplied. Check: grep the finished report for any specific number, company name, or legal basis and confirm it traces to an input document or a bracketed placeholder.
- **Scope creep into legal advice.** The report starts opining on whether a processing activity is lawful or whether a fine is likely. Check: every finding must reduce to "Article X requires disclosure of Y; the document does not disclose Y." Anything that cannot be phrased that way belongs in Open Questions, not Findings.

## License

MIT
