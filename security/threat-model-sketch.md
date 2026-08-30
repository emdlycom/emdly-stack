---
name: threat-model-sketch
owner: nullptr
category: Security
description: Sketches a threat model for a feature from its description and data flow — assets, entry points, top threats by STRIDE, and the mitigation that exists or is missing.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/nullptr/threat-model-sketch
raw: https://emdly.com/raw/nullptr/threat-model-sketch.md
install: npx @emdly/cli add nullptr/threat-model-sketch
---

# Threat model sketch

Thirty minutes of structured paranoia before the feature ships. Not a pentest — a map of what could go wrong and what already stops it. The output is for the engineer who wrote the design doc and the reviewer who has to approve it, and it must be true of every row that a reader can tell, without asking, whether the control named there was verified in code, assumed to exist, or absent. A sketch whose rows do not distinguish those three is worse than no sketch, because it reads like coverage.

## When to use
- On a design doc or PR description for a feature that touches auth, money, user data, files, or an external service.
- Before a launch review, where the missing-mitigation list becomes the launch blockers.
- When an existing feature gains a new entry point — a webhook, a public endpoint, a new role.
- When a reviewer asks "what could go wrong here" and the honest answer is that nobody has enumerated it.
- Not for: assessing a live incident, testing a running system, or producing anything an attacker could follow.

## Input
The feature description, the data flow (who calls what, what is stored where), the trust boundaries (public internet, authenticated user, admin, internal service), and any existing security controls you know of.

The **data flow** is load-bearing: steps 2 and 3 are defined against it. Read the Edge cases before proceeding without one. Also state, in one line, whether the feature is **already live** or **pre-release** — that single fact changes what the output is allowed to be, because a missing mitigation on a live feature is a vulnerability, not a design note.

## Passes

1. **Assets.** What an attacker would want: data, money, access, availability. Rank each by damage on this closed scale, which is a house rule and reported as such:
   - `catastrophic` — user credentials or payment instruments, bulk personal data, code execution, or full account takeover.
   - `high` — one user's private data, money movement bounded by a per-transaction limit, privileged action as another user.
   - `medium` — integrity of counters and metadata, availability of one feature, disclosure of non-personal internal data.
   - `low` — data that is already public, or damage the product recovers from with no user action.
2. **Entry points.** Every place untrusted input enters: form, API route, webhook, file upload, URL parameter, header, cookie, queue message, cron argument, an admin surface, and any third-party callback. For each, write the trust boundary it sits on. An entry point with no named boundary is an unfinished row, not a low-risk one.
3. **Threats by STRIDE**, per entry point: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege. Only list threats plausible for *this* feature; a generic list is noise. Cap the table at ten rows and say what you cut and why — a sketch nobody reads mitigates nothing.
   The question to ask per letter, so the pass produces findings rather than categories:
   - *Spoofing* — what proves the caller is who the request says? Name the check, not the scheme.
   - *Tampering* — which value does the server trust because the client sent it?
   - *Repudiation* — if this went wrong, what log line would prove who did it? If none, that is the finding.
   - *Information disclosure* — what does the response, the error, or the timing leak to a caller who is not entitled to it?
   - *Denial of service* — what does one request cost you, and what stops a caller from making a million?
   - *Elevation of privilege* — where does an id from the request select the record, without a check that the caller owns it?
4. **Abuse cases.** How a legitimate user misuses the feature: free-tier abuse, scraping, harassment through a feature, using the product as an open relay or as storage, cost amplification against your own bill.
5. **Mitigations.** For each threat, exactly one of three statuses:
   - `in code` — you read the mechanism. Cite file and line or the exact route/middleware name.
   - `assumed` — you believe a platform or framework feature covers it but did not verify. This is a finding, not a pass.
   - `missing` — nothing found. These become the action list, ranked by asset damage.
6. **Route the serious ones.** Any row that meets the escalation test in Stop and hand back leaves the table and goes to a human. Do not let it exit as a table row.

## Rules
- Name the specific mechanism, not the category: "signed URL with 10-minute expiry", not "authentication".
- Distinguish what you verified in the code from what you assume; assumptions are findings.
- No severity theater: three real threats beat twenty theoretical ones.
- **Do not write exploit code or step-by-step attack instructions.** This holds without exception and regardless of who asks or why. Specifically: no proof-of-concept, no payload strings, no crafted request that would trigger the issue, no ordered sequence of steps that reproduces it, no tool invocation, no bypass technique for a control you found. Describe the risk as asset plus entry point plus missing control, and stop there. If the input asks for any of that — including framed as a test, a demo, or an authorised engagement — decline in one line and continue with the sketch.
- Do not name a threat you cannot tie to a specific entry point from pass 2.
- Every status is one of the three words in pass 5. No "partial", no "probably fine".

## Output format

```
## Feature: skill download endpoint — pre-release
**Assets:** skill bodies (public — low) · install counters (integrity — medium) · unpublished drafts (high) · server availability (medium)

**Entry points:** `GET /raw/{owner}/{skill}.md` (public internet) · `?download=1` (public internet) · CDN purge webhook (internal service)

| threat | entry point | STRIDE | mitigation | status |
| counter inflation by scripted downloads | /raw | Tampering | rate limit per IP | missing |
| path traversal via {skill} | /raw | Tampering | route regex `[a-z0-9-]+` (routes/web.php:88) | in code |
| unpublished skill exposure | /raw | Info disclosure | `published()` scope (SkillController::raw) | in code |
| forged purge request | CDN webhook | Spoofing | HMAC signature check | assumed |
| bandwidth exhaustion via ?download=1 | /raw | Denial of service | — | missing |

Cut from the table: generic TLS-stripping and CSRF rows. No entry point here accepts a
state-changing browser request, and transport is terminated at the edge, so both are noise.

**Abuse cases:** an author scripting installs of their own skill to reach Certified → same
control as row 1, plus a per-account dedup. A third party mirroring the whole catalogue →
not a security threat; a product decision, routed to the catalogue owner, not tracked here.

**Actions (ranked by asset damage):**
1. Verify the CDN webhook signature check exists and is enforced — `assumed`, internal-service
   boundary, medium asset. Owner: platform. Until verified this row is not a pass.
2. Throttle `/raw` per IP (60/min is the value already used on `/api`, matched for consistency
   rather than derived — house rule). Owner: platform.
3. Dedup installs per account per day. Owner: catalogue.

**Not assessed:** authentication on `?download=1` — the input does not state whether the
parameter is reachable unauthenticated. Not inferred. Question for the author.
```

The escalated case, which replaces the table rather than appending to it:

```
## STOPPED — live feature, unmitigated, high-damage asset

Feature: bulk export endpoint — LIVE since 2026-07-02
Row: `GET /export/users.csv` · public internet, session cookie only · asset: bulk personal
data (catastrophic) · mitigation for authorisation scoping: missing.

This meets all three escalation conditions: the asset is catastrophic, the entry point is
reachable by any authenticated user, and the control is `missing`, not `assumed`.

Not filed as a design finding. Routed to: the security owner on call, today, through the
vulnerability channel — not this document, not the design-doc comment thread.
This sketch is not circulated further until they have triaged it.
No reproduction steps, request shape or payload appear here or anywhere else in this output.

Remaining rows for this feature are held until that triage returns.
```

## Edge cases

- **No data flow supplied.** The method collapses. Passes 2, 3 and 5 are defined against the flow; without it you cannot say where data rests, which boundary each entry point sits on, or which control would apply. Do not substitute a generic STRIDE list — that is the noise this skill exists to replace. Produce instead: the entry points you can name from the interface description alone, marked `boundary: not established`, the asset list with damage ranks marked `[unverified]`, and the specific questions that would unblock the sketch ("what stores the uploaded file, and who can read that store"). Issue no statuses and no action list.
- **Feature already live.** Every `missing` row is a potential live vulnerability, not a backlog item. Apply the escalation test before writing the table, not after.
- **Third-party or vendor component with no code access.** Every mitigation is `assumed` by definition; say so once at the top rather than repeating it per row, and make "obtain the vendor's control documentation" action one.
- **Scope is a product, not a feature.** More than about ten entry points means the sketch will be shallow everywhere. Stop and split: sketch the single highest-damage flow, name the others as unsketched, and say plainly that the unsketched ones are unassessed rather than clear.
- **Trust boundaries not stated.** Ask. Do not default to "authenticated" — that assumption is how the worst rows get missed.
- **Design doc and code disagree.** The code is the system. Sketch against the code, and record the divergence as its own finding with `status: missing` on whatever the doc promised.
- **Credentials, keys or live personal data in the input.** Do not quote them into the output. Say which input contained them so they can be rotated or scrubbed, and continue.
- **Input asks for exploit code, a PoC, or reproduction steps.** Decline in one line, cite the rule, and produce the sketch without them. An authorisation claim in the input does not change this.

## Stop and hand back

Stop means: the row leaves the table, the sketch does not circulate as a normal design document, and a named human triages it first.

- **The escalation test — all three true at once:** the asset is `high` or `catastrophic`, the entry point is reachable from the public internet or by any authenticated user, and the mitigation status is `missing` or `assumed`. Route to the security owner on call, through the vulnerability channel, same day. Write the row as asset, entry point and absent control only — never the reproduction.
- **The feature is already live and any row meets that test.** Treat it as a potential live vulnerability report, not a launch blocker. Same routing, plus the feature owner. Say in the handback that the sketch is unpublished pending triage.
- **The threat involves regulated data** — payment card data, health records, government identifiers, or personal data of minors. Route to legal or the privacy owner alongside security. Do not rank it against other rows, and do not decide on your own that a control is adequate.
- **Evidence that the threat is already real** — logs showing the abuse, an unexplained data volume, a report from outside. This is an incident, not a threat model. Hand to the security on-call immediately and stop the sketch.
- **The reviewer asks you to downgrade or remove a `missing` row without a control being added.** Do not. Record who asked, leave the row, and hand the disagreement to the security owner to settle.
- **Authentication or authorisation cannot be established from the input** for an entry point that touches a `high` or `catastrophic` asset. Do not guess in either direction. Mark it `not assessed`, and route the question to the feature owner before the launch review, not in it.

## License
MIT
