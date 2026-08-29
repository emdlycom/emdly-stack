---
name: threat-model-sketch
owner: nullptr
category: Security
description: Sketches a threat model for a feature from its description and data flow — assets, entry points, top threats by STRIDE, and the mitigation that exists or is missing.
version: v2
license: MIT
updated: 2026-08-27
recommended: false
security_checked: true
url: https://emdly.com/skills/nullptr/threat-model-sketch
raw: https://emdly.com/raw/nullptr/threat-model-sketch.md
install: npx @emdly/cli add nullptr/threat-model-sketch
---

# Threat model sketch

Thirty minutes of structured paranoia before the feature ships. Not a pentest — a map of what could go wrong and what already stops it.

## When to use
- On a design doc or PR description for a feature that touches auth, money, user data, files, or an external service.
- Before a launch review.

## Input
The feature description, the data flow (who calls what, what is stored where), the trust boundaries (public internet, authenticated user, admin, internal service), and any existing security controls you know of.

## Process
1. **Assets:** what an attacker would want — data, money, access, availability. Rank by damage.
2. **Entry points:** every place untrusted input enters (form, API, webhook, file upload, URL parameter, header, queue message).
3. **Threats by STRIDE** per entry point: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege. Only list threats that are plausible for *this* feature; a generic list is noise.
4. **Abuse cases:** how a legitimate user misuses the feature (free-tier abuse, scraping, harassment via a feature).
5. **Mitigations:** for each threat — *in code* (cite the file/mechanism), *assumed* (a platform feature you believe exists), or *missing*. Missing ones become the action list, ranked by asset damage.

## Rules
- Name the specific mechanism, not the category: "signed URL with 10-minute expiry", not "authentication".
- Distinguish what you verified in the code from what you assume; assumptions are findings.
- No severity theater: three real threats beat twenty theoretical ones.
- Do not write exploit code or step-by-step attack instructions. Describe the risk and the control.

## Output format
```
## Feature: skill download endpoint
**Assets:** skill bodies (public — low) · install counters (integrity — medium) · server availability (medium)

**Entry points:** GET /raw/{owner}/{skill}.md (public) · ?download=1

| threat | STRIDE | mitigation | status |
| counter inflation by scripted downloads | Tampering | rate limit per IP | missing |
| path traversal via {skill} | Tampering | route regex `[a-z0-9-]+` (routes/web.php) | in code |
| unpublished skill exposure | Info disclosure | `published()` scope in controller | in code |

**Abuse cases:** an author scripting installs of their own skill to reach Certified → same mitigation as row 1, plus a per-account dedup.

**Actions (ranked):** 1. throttle `/raw` per IP (e.g. 60/min) · 2. dedup installs per IP per day
```

## License
MIT
