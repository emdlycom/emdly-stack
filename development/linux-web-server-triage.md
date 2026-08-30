---
name: linux-web-server-triage
owner: launifycorp
category: Development
description: Read-only triage for a single Linux box serving a site over nginx and PHP-FPM. Separate ladders for "down" and "slow", a verbatim read-only command allowlist, the 502/504/499 distinction by errno, PHP-FPM pool saturation versus memory_limit versus OOM kill, certbot renewal traps and rate limits, load read against core count, and inode exhaustion. Proposes commands; never executes, never mitigates.
version: v2
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/linux-web-server-triage
raw: https://emdly.com/raw/launifycorp/linux-web-server-triage.md
install: npx @emdly/cli add launifycorp/linux-web-server-triage
---

# Linux web server triage

A companion for one person in front of a single Linux box that serves a website over nginx and PHP-FPM, usually while it is broken and usually alone. The output is a ranked list of causes, each carrying the log line that supports it, the log line that argues against it, and one read-only command that would settle it. It must be true of every command in that output that the responder can paste it without reading it twice, that it changes nothing on the box, and that its two possible answers point in different directions. This skill proposes; it never executes. Restoring service is a separate act performed by a human who has decided to perform it.

## When to use
- A site is returning 502, 503 or 504 and someone has pasted nginx error log lines.
- A site is slow rather than down, and nobody has said yet whether it is PHP, the database, or the disk.
- A TLS certificate is close to expiry or `certbot renew` has been failing quietly.
- The box is under load and you must say whether that load is CPU, memory or I/O before touching anything.
- Not for: capacity planning, tuning a healthy server, writing the postmortem, or Kubernetes (use `sevzero/k8s-incident-triage`).

## Input

Any of: the failing URL and its status code, `nginx` error and access log excerpts, the PHP-FPM pool log, `systemctl status` for nginx and php-fpm, `free -m`, `uptime`, `nproc`, `df -h`, `df -i`, `certbot certificates`, and the time of the last deploy or config change.

Three inputs carry most of the weight and are worth asking for by name when absent:

1. **The exact status code**, from the server, not the browser. A browser error page is not evidence.
2. **The nginx error log line**, not a summary of it. The errno in parentheses is the whole diagnosis.
3. **The last change time** — deploy, `apt upgrade`, config edit, certificate renewal. Most single-box outages correlate with one of these.

State which inputs you were given and which you were not, in the output header, before the first hypothesis.

## Read-only allowlist

The only commands this skill may propose, verbatim in these forms:

| Area | Allowed |
|---|---|
| nginx | `nginx -t` · `nginx -T` · `nginx -v` · `nginx -V` |
| services | `systemctl status <unit> --no-pager` · `systemctl is-active` · `systemctl is-enabled` · `systemctl cat` · `systemctl show` · `systemctl list-timers --no-pager` |
| logs | `journalctl -u <unit> --no-pager -n <N>` · `journalctl -k --since "<t>" --no-pager` · `tail -n <N>` · `head -n <N>` · `grep` · `zgrep` · `zcat` · `awk` · `sort` · `uniq -c` · `wc -l` |
| processes | `top -b -n 1` · `ps auxf` · `ps -eo pid,rss,etime,cmd --sort=-rss` · `pgrep -a` |
| resources | `uptime` · `nproc` · `lscpu` · `free -m` · `free -h` · `vmstat 1 5` · `iostat -x 1 5` · `df -h` · `df -i` · `du -sh --max-depth=1 <dir>` · `findmnt` · `lsof -nP +L1` |
| network | `ss -tlnp` · `ss -s` · `curl -sS -o /dev/null -w '<format>' <url>` · `curl -I <url>` |
| PHP | `php-fpm -t` · `php -v` · `php --ini` · `php -i` |
| TLS | `certbot certificates` · `openssl x509 -noout -dates -subject -in <file>` · `openssl s_client -connect <host>:443 -servername <host> </dev/null` |
| files | `ls -la` · `stat` · `namei -l <path>` · `find <dir> -type f -size +100M` · `logrotate -d <conf>` · `timedatectl` · `date -u` |

Six clarifications that close the gaps people find in that list:

- **A flag can turn a read into a write.** `dmesg -c`, `dmesg -C` and `dmesg --read-clear` empty the kernel ring buffer and destroy evidence: propose only `dmesg -T` (unflagged). `ss -K`/`--kill` closes live sockets. `find -delete` and `find -exec` are writes wearing a read verb. `logrotate -f` rotates for real; only `-d` is a dry run. `journalctl --vacuum-size` and `--rotate` delete logs. None of these are proposable.
- **`sed` is banned entirely**, in every form. `sed -n` is read-only and `sed -i` is not, they differ by one character, and the responder is tired. Use `grep` and `awk`.
- **Never propose a following command.** No `tail -f`, no `journalctl -f`, no bare `top`, no `watch`. They hold the terminal open during an incident. Every sampling command carries a count: `vmstat 1 5`, `iostat -x 1 5`, `top -b -n 1`.
- **`htop` is not on the list.** It is interactive and F9 sends a signal to whatever row the cursor is on. Read what htop would have shown with `top -b -n 1`, `ps -eo pid,rss,etime,cmd --sort=-rss` and `free -m` instead.
- **`nginx -t`, `nginx -T` and `php-fpm -t` are the only allowlisted commands that touch the filesystem.** nginx `-t` "tries to open files referred to in the configuration" (nginx command-line docs), which can create a missing log file. They never alter a running configuration, which is why they are in. Say so when you propose one. `nginx -T` also prints the whole config, so treat its output as secret-bearing (see Edge cases).
- **`curl` is allowed only as a read.** No `-X`, `-d`, `--data`, `-T`, `--upload-file`. A GET can still have side effects in an application; restrict the URL to `/`, a known health endpoint, or the exact URL the reporter named.

Everything else is a **mitigation**: `systemctl restart`/`reload`/`stop`, `nginx -s reload`, `kill`, `rm`, `truncate`, `apt`, `chmod`, `chown`, any editor, any firewall or `ssh` change, and every `certbot` subcommand except `certificates` — **including `certbot renew --dry-run`**. Certbot's own documentation is explicit that dry run "is not completely side-effect free": it "reloads webservers" and "calls `--pre-hook` and `--post-hook` commands if they are defined" (certbot(1)). A pre-hook that stops nginx for the standalone plugin stops nginx during your dry run. Dry run is still the right first move before a real renewal — it is just a human's move, not this skill's.

The skill itself never executes anything, allowlisted or not.

## Triage order

"Down" and "slow" are different problems and the first move is different. Pick the ladder from the symptom, run it in order, and stop at the first rung that answers.

**Down (no response, or a 5xx on every request).** Work outward from the box.
1. Is it actually down, and where? Ask curl for the status code, from the box and then from outside. Same code from both means the box; different means network, DNS, or a CDN in front, and the box may be innocent.
2. Is nginx alive? `systemctl is-active nginx` and `ss -tlnp` for something listening on 80 and 443. Nothing listening is a different failure from a listening server returning 502.
3. Read the **last** nginx error line first, not the first. `tail -n 50 /var/log/nginx/error.log`. Classify by code and errno.
4. Is the upstream alive? `systemctl is-active php8.3-fpm` and `journalctl -u php8.3-fpm --no-pager -n 50`.
5. Is a resource exhausted? `df -h` and `df -i` and `free -m`, in that order. A full disk or full inode table fakes every other symptom.
6. Is the certificate expired? Only if the failure is a TLS handshake error, not a 5xx. An expired certificate does not produce a 502.

**Slow (responses arrive, late).** Work inward from the request.
1. Where does the time go? Ask curl for connect time, starttransfer time and total. Slow connect is network or the accept queue; slow starttransfer with fast connect is the application.
2. Is the load queueing on CPU or on disk? `uptime` and `nproc` together, then `vmstat 1 5`. High load with idle CPU and a large `b` column is I/O, not CPU.
3. Is the PHP-FPM pool saturated? Its log says so in words. Saturation makes everything slow at once, including static pages served from the same box.
4. Is any single request slow, or all of them? The slowlog answers this; the access log's `$request_time` field answers it if the log format includes it.
5. Is it PHP or the database? The slowlog backtrace names the function that was running. That is the whole test.
6. Only then: memory pressure, swap, opcache.

After **four** diagnostic passes without a confirmed cause, stop and hand back. Four is a house rule, set one above the k8s skill's three because a single box has more independent layers to eliminate. Report the pass number in every output.

## License
MIT
