---
name: email-deliverability-audit
owner: launifycorp
category: Email and outreach
description: Audits a domain's sending configuration from its public DNS alone — no mailbox, no ESP login, no access to anything. It reports what is configured, what is broken, and what the large mailbox operators...
version: v1
license: MIT
updated: 2026-09-03
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/email-deliverability-audit
raw: https://emdly.com/raw/launifycorp/email-deliverability-audit.md
install: npx @emdly/cli add launifycorp/email-deliverability-audit
---

# Email Deliverability Audit

Audits a domain's sending configuration from its public DNS alone — no mailbox, no ESP login, no access to anything. It reports what is configured, what is broken, and what the large mailbox operators require, ordered by what actually stops mail from arriving.

It never reports a deliverability score or an inbox placement rate. Those cannot be derived from DNS, and a number invented from configuration is worse than no number.

## When to use

- "why are our emails going to spam", "check our SPF/DKIM/DMARC", "audit our sending domain"
- Before a first campaign from a new domain, after migrating ESP, or after adding a tool that sends on the domain's behalf.
- When a campaign's delivery rate drops and nobody changed the copy.

Do not use it to diagnose one bounced message — a bounce code and its full headers answer that directly.

## Step 1 — Inputs

- **The domain** that appears after the `@` in the From address. Ask for it if only a URL was given.
- **Any subdomains that send** — `mail.`, `news.`, `send.`, `mg.`, `email.` Marketing and transactional mail usually leave from different subdomains with different records; audit each separately.
- **The DKIM selectors**, if known. They cannot be enumerated from DNS: `selector._domainkey.<domain>` is only discoverable if you know the selector. Try the well-known ones for common providers (`google`, `k1`, `s1`/`s2`, `mandrill`, `dkim`, `sendgrid`/`s1._domainkey`, `pm`, `zoho`, `mail`), and say plainly which selectors were tested and that others may exist. Absence of a tested selector is not proof DKIM is missing — the cleanest evidence is a full header set from a real message, so offer to read one.
- If a raw message header is supplied, use it: `Authentication-Results` states what the receiver actually concluded, which beats any inference from records.

## Step 2 — Run the checks, in order of consequence

Query DNS directly (`dig +short TXT <name>`, `dig MX`, `dig -x <ip>`). Quote every record verbatim in the report.

**1. SPF** — `TXT` at the domain root.
- Exactly one SPF record. Two records is a `permerror`; the whole check fails, it does not "pick the better one".
- The **10 DNS-lookup limit** (RFC 7208 §4.6.4). `include`, `a`, `mx`, `ptr`, `exists` and `redirect` each cost a lookup, and nested includes count recursively — this is the single most common silent failure on domains that accumulated a mail provider, a CRM, an invoicing tool and a helpdesk over a few years. Count the lookups by expanding every include, and report the number.
- The void-lookup limit (2) and the 255-character-per-string limit.
- `ptr` is deprecated — flag it.
- The ending: `-all` (fail), `~all` (softfail), `?all` (neutral). `+all` accepts the world; flag it as critical.
- Say clearly that SPF authenticates the **envelope sender (Return-Path)**, not the visible From. This is why SPF alone does not stop spoofing of the header From.

**2. DKIM** — `TXT` at `<selector>._domainkey.<domain>`.
- Record present and parseable, `v=DKIM1`, `p=` non-empty (an empty `p=` is a revoked key).
- Key length: flag 1024-bit as weak, 2048-bit as current.
- Note test mode (`t=y`) — it tells receivers to ignore failures.
- List every selector found, and which service each belongs to when identifiable.

**3. DMARC** — `TXT` at `_dmarc.<domain>`.
- Policy: `p=none` monitors only. A domain sitting on `p=none` for years with nobody reading reports has DMARC in name only — say so.
- `sp=` for subdomains, `pct=` for partial rollout.
- **Alignment** is the part that surprises people: DMARC passes when SPF *or* DKIM passes **and** its domain aligns with the header From. `aspf`/`adkim` set strict (`s`) or relaxed (`r`) matching. A service that sends with its own Return-Path fails SPF alignment even though SPF itself passes — only DKIM saves it.
- `rua=` present and pointing somewhere a human reads. If the reporting address is on another domain, that domain needs an authorisation record (`<domain>._report._dmarc.<external>`); flag its absence.
- Note that forwarding breaks SPF but usually preserves DKIM, which is why DKIM alignment matters more for reach.

**4. Large mailbox operator requirements** — Gmail, Yahoo and Microsoft publish thresholds for bulk senders (authentication, one-click unsubscribe per RFC 8058, a spam-complaint ceiling, TLS, valid reverse DNS). These change: **read the operator's current documentation and cite it** rather than stating remembered numbers or dates. Report which requirements the DNS evidence can confirm and which need Postmaster Tools or the ESP dashboard — complaint rates are among the latter and can never be inferred from records.

**5. Reverse DNS and MX** — a `PTR` for each sending IP that resolves forward to the same host, and MX records that resolve. For a domain that only sends, or one that sends nothing at all, check for a null MX (RFC 7505) plus `p=reject` — the correct configuration for a parked or brand-protection domain.

**6. Transport and identity extras** — MTA-STS (`_mta-sts` TXT plus the policy at `https://mta-sts.<domain>/.well-known/mta-sts.txt`, and whether it is `testing` or `enforce`), TLS-RPT (`_smtp._tls`), DNSSEC, DANE/TLSA, and BIMI (`default._bimi`) — noting BIMI needs DMARC at quarantine or reject, and that some operators additionally require a verified mark certificate. These are inbound-protection and brand features, not the reason mail lands in spam; report them last and label them as such.

**7. Blocklists** — a DNSBL query can be run, but a listing must be confirmed at the operator's own lookup page before it is reported, with the delisting URL. Never report a listing from a single cached DNS answer, and never state a "reputation score".

## Rules

- Never report an inbox placement rate, a deliverability percentage, or a sender reputation score. State configuration and cite the standard; the operator's own dashboard owns the rest.
- Never quote a threshold from memory. Either it comes from the operator's live documentation with a link, or it is not in the report.
- Never claim DKIM is absent because a guessed selector did not resolve. Say which selectors were tested.
- Expand includes rather than eyeballing the SPF string — the lookup count is the finding, not the record's length.
- Keep marketing, transactional and corporate mail streams separate throughout; they usually differ, and a single verdict for the domain hides the broken one.
- Recommend the DMARC path as a staged rollout (`none` with reports → `quarantine` with a percentage → `reject`), never a jump straight to `reject`. Moving too fast silently drops real mail, and the domain owner is the one who finds out.
- This audit changes nothing. Publishing DNS changes is the owner's action, on their own authority.

## Output format

```
Domain: <example.com>   Subdomains audited: <mail., news.>   Checked: <date>

BLOCKING (<n>)
  SPF   12 DNS lookups — limit is 10 (RFC 7208 §4.6.4) → permerror, SPF fails entirely
        v=spf1 include:_spf.google.com include:sendgrid.net include:mailgun.org ~all
        expansion: google 4, sendgrid 3, mailgun 3, root mx 1, a 1
        fix: flatten or drop the unused sender — which of these still sends?

WEAKENING (<n>)
  DMARC p=none since <first seen>, rua points to an unmonitored alias
        v=DMARC1; p=none; rua=mailto:dmarc@example.com
  DKIM  selector "k1" is 1024-bit

CONFIGURED CORRECTLY (<n>)
  DKIM  google (2048-bit), MX resolve, PTR forward-confirmed, TLS-RPT present

NOT VERIFIABLE FROM DNS
  Complaint rate, bounce rate, list hygiene, content filtering — check
  Postmaster Tools and the ESP dashboard
  Selectors tested: google, k1, s1, s2, pm — others may exist

RECOMMENDED ORDER
  1  Fix the SPF lookup count (blocking, ~1 hour)
  2  Route DMARC reports somewhere read, then move to p=quarantine pct=25
  3  Rotate the 1024-bit key to 2048-bit at the next provider maintenance
```

## License

MIT
