---
name: linux-web-server-triage
owner: launifycorp
category: Development
description: Read-only triage for a single Linux box serving a site over nginx and PHP-FPM. Separate ladders for "down" and "slow", a verbatim read-only command allowlist, the 502/504/499 distinction by errno, PHP-FPM pool saturation versus memory_limit versus OOM kill, certbot renewal traps and rate limits, load read against core count, and inode exhaustion. Proposes commands; never executes, never mitigates.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/linux-web-server-triage
raw: https://emdly.com/raw/launifycorp/linux-web-server-triage.md
install: npx @emdly/cli add launifycorp/linux-web-server-triage
---

# Linux web server triage

Probe: testing whether the fenced example blocks are what the publisher rejects.

## Output format

```
## Inputs given
Status code · nginx error.log tail 50 · php8.3-fpm pool log · free -m · uptime · nproc · certbot certificates
Missing: df -h, df -i (disk not assessed), access log, last deploy time
Symptom class: DOWN · Diagnostic pass: 2 of 4

## Hypotheses (ranked)
1. **FPM pool saturated; nginx cannot get a connection** — for: error.log has
   `connect() to unix:/run/php/php8.3-fpm.sock failed (11: Resource temporarily unavailable)`
   and the pool log has `WARNING: [pool www] server reached pm.max_children setting (5), consider raising it`.
   Next: `grep -c 'max_children' /var/log/php8.3-fpm.log`
   → a rising count across the window confirms sustained saturation.

## Load
uptime `load average: 24.90, 22.14, 15.03` · nproc `4` → 6.2 per core, still rising.

## Not assessed
- Disk space and inodes — no `df -h` or `df -i` supplied. not assessed.

## Flagged
- TLS certificate: `certbot certificates` reports `VALID: 12 days`. **flagged — renewal overdue, gated.**

## Mitigation candidates — NOT proposed, gated
- raise `pm.max_children`, then `php-fpm -t`, then reload. This skill runs none of these.
```

The declined case:

```
## Inputs given
Free text only: "site is slow, can you look"

Not enough to rank causes. Declining to produce one.
Symptom class: SLOW (asserted, unverified) · Diagnostic pass: 0 of 4
```

## License
MIT
