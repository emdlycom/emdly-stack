---
name: linux-web-server-triage
owner: launifycorp
category: Development
description: Read-only triage for a Linux box serving a site over nginx or Apache, with PHP-FPM, bare metal or in Docker. Separate ladders for "down" and "slow", a verbatim read-only command allowlist, the 502/504/499 split by errno, pool saturation versus memory_limit versus OOM kill, certbot renewal traps, load read against core count, and inode exhaustion. Proposes commands; never executes, never mitigates.
version: v4
license: MIT
updated: 2026-09-03
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/linux-web-server-triage
raw: https://emdly.com/raw/launifycorp/linux-web-server-triage.md
install: npx @emdly/cli add launifycorp/linux-web-server-triage
---

# Linux web server triage

Read-only triage for a single Linux box serving a site over nginx or Apache with PHP-FPM, on bare metal or in Docker. It ranks probable causes from the evidence actually supplied, names the one command that would separate the top two hypotheses, and stops there.

It proposes commands. It never executes them, never mitigates, and never restarts anything. Every mitigation is listed as a gated candidate for a human to run.

The discipline this skill enforces: **an unsupplied input is "not assessed", never "fine".** A ranking built on absent evidence is a guess wearing a number.

## When to use

- "the site is down", "502 since the deploy", "everything got slow this afternoon"
- Someone pastes an nginx error log, an FPM pool log, `dmesg`, or a `df` and wants to know what it means.
- Before restarting a service in production, to establish what the evidence actually supports.

Do not use it for application-level debugging — a stack trace in the app's own log is answered by reading the code.

## Step 1 — Classify the symptom

Two ladders, and they diverge immediately. Ask which if it is not stated:

- **DOWN** — connections refused, 5xx on every request, nothing served.
- **SLOW** — pages render but late, some requests time out, intermittent 5xx under load.

Record it as asserted until an input confirms it. "Site is slow" with no status code and no log is `SLOW (asserted, unverified)`.

## Step 2 — Take inventory of the evidence

List what was supplied and what was not. Score the diagnostic pass: **1 of 4** = one input class, **4 of 4** = all four. The four classes:

1. **Edge** — status code, nginx `error.log` / Apache `error_log` tail, access log around the event
2. **App tier** — PHP-FPM pool log, PHP error log, slowlog, `php-fpm -t`, `nginx -t` / `apachectl -t`
3. **Host** — `free -m`, `uptime`, `nproc`, `df -h`, `df -i`, `dmesg -T | tail`, `vmstat 1 5`
4. **Change** — last deploy time, last package upgrade, `systemctl status`, cert expiry, config edits

**Below 2 of 4, decline to rank.** Say what is missing and which single input would move it furthest. A confident hypothesis list from one log line is the failure mode this skill exists to prevent.

## Step 3 — DOWN ladder

Walk it in order; each rung is cheap and rules out the one below.

1. **Does anything listen?** `ss -ltnp` — nothing on :80/:443 means the web server is not running, and the log to read is the service's, not the app's.
2. **Does the web server think its config is valid?** A failed reload leaves the *old* config running, so a broken file can sit unnoticed until the next restart, which then fails. `nginx -t` / `apachectl -t`.
3. **Read the errno, not the status code.** For nginx upstream failures the number in the parentheses is the diagnosis:

   | Evidence | Means |
   |---|---|
   | `connect() ... failed (111: Connection refused)` | FPM is not running or not on that socket/port |
   | `connect() ... failed (11: Resource temporarily unavailable)` | listen backlog full — saturation, not absence |
   | `connect() ... failed (13: Permission denied)` | socket ownership/mode, or SELinux/AppArmor denial |
   | `upstream prematurely closed connection while reading response header` | the FPM child died mid-request — fatal error or OOM kill |
   | `(104: Connection reset by peer)` | upstream dropped it; pair with the FPM log for the same second |
   | `upstream sent too big header` | response headers exceed `fastcgi_buffer_size` — a redirect loop or a large cookie |
   | `Too many open files` | file-descriptor limit (`worker_rlimit_nofile`, unit `LimitNOFILE`) |

4. **502 vs 504 vs 499** — different failures, routinely confused:
   - **502** — upstream refused, died, or spoke nonsense. Cause is at the app tier.
   - **504** — upstream accepted and never finished in time (`fastcgi_read_timeout`, `proxy_read_timeout`). The app is slow, not absent.
   - **499** — the *client* hung up before the response. Nearly always a symptom of the slowness above it, or a load balancer with a shorter timeout than the origin. Never a cause in itself.
5. **Disk.** `df -h` **and** `df -i` — always both. Inode exhaustion presents as `No space left on device` while `df -h` shows free space, and it is caused by many tiny files: PHP session files, mail spool, unrotated logs, a cache directory nobody prunes.
6. **Certificates.** `certbot certificates` for the remaining days, and `systemctl list-timers` for whether the renewal timer even fires. The traps: an HTTP→HTTPS redirect or a closed port 80 breaks `http-01` validation; a changed webroot leaves the challenge unservable; a DNS move invalidates it silently. Renewal failures are also rate-limited by the CA — read the current limits from the CA's own documentation before advising a retry loop, and never state a limit from memory.

## Step 4 — SLOW ladder

1. **Read load against core count.** `uptime` against `nproc`: 8.0 on 8 cores is saturated, on 32 it is idle. And on Linux load counts uninterruptible (D-state) tasks, so high load with low CPU is **I/O or network wait, not CPU** — `vmstat 1 5` separates them (`wa` column, `si`/`so` for swap).
2. **Separate the three memory failures.** They look alike and have nothing in common:
   - **`memory_limit`** — one request exceeded its budget. `Allowed memory size of N bytes exhausted` in the PHP error log. Nothing else on the box is affected.
   - **OOM killer** — the kernel shot a process to save the machine. `dmesg -T | grep -i -E 'out of memory|killed process'`. Everything on the box is affected, and the victim is often not the culprit.
   - **Swap thrashing** — memory is technically available, so nothing errors; the box just crawls. Visible as sustained `si`/`so` in `vmstat`.
3. **Pool saturation.** `WARNING: [pool www] server reached pm.max_children setting` in the FPM log. One occurrence is a spike; a rising count across the window is sustained saturation. Read it together with `pm` mode (`static` / `dynamic` / `ondemand`) and, crucially, with per-child memory — raising `max_children` past what RAM supports converts a queueing problem into an OOM kill, which is strictly worse.
4. **Find the slow requests themselves.** `request_slowlog_timeout` plus `slowlog` gives PHP stack traces of what actually hangs — usually a query without an index or an external HTTP call with no timeout. This is the highest-value input on the SLOW ladder; if it is off, say that turning it on is the next step.
5. **Timeout arithmetic.** `max_execution_time` (PHP), `request_terminate_timeout` (FPM), `fastcgi_read_timeout` (nginx), plus any load balancer or CDN timeout in front. When an inner timeout exceeds an outer one, the user gets a 504 while the work continues to completion, unseen. State the chain as numbers.

## Step 5 — Apache instead of nginx

The ladders are identical; the evidence has different names.

- Config test `apachectl -t` (or `-t -D DUMP_VHOSTS`); logs at `error_log` / `access_log`, per-vhost.
- Saturation reads `server reached MaxRequestWorkers setting, consider raising it` — the analogue of `max_children`, with the same RAM caveat.
- Establish the MPM first (`apachectl -V | grep MPM`): **prefork** with mod_php means one process per connection and a memory ceiling reached fast; **event/worker** with PHP-FPM behaves like the nginx case.
- With `mod_proxy_fcgi`, the 504 analogue is `ProxyTimeout` / the `timeout=` on the handler, and the 502 analogue appears as `AH01067` / `End of script output before headers`.
- `mod_status` (when enabled, and it usually is not) shows the worker table directly — worth asking for.

## Step 6 — Docker in the picture

If the stack is containerised, three checks come *before* everything above, because they change what the other evidence means:

1. `docker ps -a` — is the container up, or in a restart loop? A loop makes the app logs look like a crash every N seconds when the real cause is the health check or the entrypoint.
2. **Exit code 137** = SIGKILL, almost always the container memory limit. Confirm with `docker inspect --format '{{.State.OOMKilled}}' <c>` — the *host* may have plenty of free RAM while the container is capped.
3. `docker stats` for the live memory ceiling, and remember the host and container see different filesystems: `df` on the host tells you nothing about a full volume inside the container, and vice versa.

Logs live at `docker logs`, not in `/var/log`, unless a volume mounts them out. Note which you were given.

## Command allowlist — read-only, verbatim

Propose only from this list. Anything not on it is a mitigation and is gated.

```
ss -ltnp                                  systemctl status <unit>
nginx -t                                  journalctl -u <unit> --since '1 hour ago'
apachectl -t | -V | -S                    tail -n 200 /var/log/nginx/error.log
php-fpm -t                                tail -n 200 /var/log/php*-fpm.log
free -m                                   uptime
nproc                                     vmstat 1 5
df -h                                     df -i
dmesg -T | tail -n 100                    ps aux --sort=-%mem | head -20
certbot certificates                      systemctl list-timers
docker ps -a                              docker logs --tail 200 <c>
docker stats --no-stream                  docker inspect <c>
```

Never propose, under any framing: `systemctl restart|reload`, `kill`, `docker restart`, editing a config, `certbot renew`, clearing a cache or log, changing a limit.

## Rules

- Never rank causes below diagnostic pass 2 of 4. Decline and name the missing input.
- Never report an unsupplied input as healthy. It is `not assessed`.
- Quote evidence verbatim — the log line, the errno, the number — never paraphrased.
- Every hypothesis carries the evidence *for* it and the single next command that would confirm or kill it. A hypothesis with no discriminating command is not a hypothesis.
- Correlation of one timestamp is not causation. A deploy at 14:02 and 502s at 14:03 is a strong lead, not a conclusion.
- Never state a vendor threshold or rate limit from memory. Cite the current documentation or leave it out.
- Certificate expiry and anything else outside the reported symptom goes under **flagged**, not into the ranking.
- Mitigations are always listed as candidates, always gated, never run.

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
2. **Children dying, not queueing** — against: no `prematurely closed connection` in the sample,
   no OOM line supplied. Next: `dmesg -T | grep -i 'killed process'`.

## Load
uptime `load average: 24.90, 22.14, 15.03` · nproc `4` → 6.2 per core, still rising.

## Not assessed
- Disk space and inodes — no `df -h` or `df -i` supplied. not assessed.
- Application latency — no slowlog supplied.

## Flagged
- TLS certificate: `certbot certificates` reports `VALID: 12 days`. **flagged — renewal overdue, gated.**

## Mitigation candidates — NOT proposed, gated
- raise `pm.max_children`, then `php-fpm -t`, then reload — only after per-child memory is
  measured against free RAM, or this trades queueing for an OOM kill. This skill runs none of these.
```

The declined case:

```
## Inputs given
Free text only: "site is slow, can you look"

Not enough to rank causes. Declining to produce one.
Most valuable single input: PHP-FPM slowlog, or `uptime` with `nproc`.
Symptom class: SLOW (asserted, unverified) · Diagnostic pass: 0 of 4
```

## License

MIT
