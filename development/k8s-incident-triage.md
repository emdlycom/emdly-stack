---
name: k8s-incident-triage
owner: sevzero
category: Development
description: Given kubectl output and alerts, ranks probable causes and proposes the next safe diagnostic step — read-only commands only, never a mutation without a human.
version: v2
license: MIT
updated: 2026-08-22
recommended: false
security_checked: true
url: https://emdly.com/skills/sevzero/k8s-incident-triage
raw: https://emdly.com/raw/sevzero/k8s-incident-triage.md
install: npx @emdly/cli add sevzero/k8s-incident-triage
---

# Kubernetes incident triage

An on-call companion that thinks out loud and keeps its hands off the cluster.

## When to use
- When an alert fires and the responder pastes `kubectl` output, events and pod logs.
- In a runbook, as the first structured pass before a human decides.

## Input
Any of: `kubectl get pods -o wide`, `describe pod/deploy/node`, `logs --previous`, `get events --sort-by=.lastTimestamp`, `top nodes/pods`, the alert text, and the last deploy time.

## Read-only allowlist
The only commands this skill may propose, verbatim: `kubectl get`, `kubectl describe`, `kubectl logs`, `kubectl top`, `kubectl get events`, `kubectl rollout history`, `kubectl rollout status`. Anything else — rollout undo, scale, delete, apply, drain, cordon, exec — is a **mitigation**, and the skill must never suggest running one without the human typing the word `confirm` and the exact resource name. The skill itself never executes anything.

## Ranking causes
For each hypothesis: the evidence for it from the input, the evidence against, and the one read-only command that would settle it. Rank by evidence, not by how common the cause is in general. Typical shapes:
- `CrashLoopBackOff` + exit code 137 → OOM; check `describe` for `Reason: OOMKilled` and limits.
- `ImagePullBackOff` → registry auth or a tag that does not exist; check events for the exact pull error.
- Pending pods + `FailedScheduling` → resources or affinity; `describe` says which.
- Healthy pods, failing requests → service/endpoints, readiness probe, or ingress; `get endpoints`.
- Everything degraded after a deploy → the deploy; `rollout history` first.

## Rules
- One next step at a time, read-only, with the exact command.
- Say what the output would mean before it is run ("if `OOMKilled` appears, memory limit is the cause").
- Time-box: after three diagnostic steps without a confirmed cause, recommend escalation or a rollback decision — for the human.
- Never propose deleting anything as diagnostics.

## Output format
```
## Hypotheses (ranked)
1. **OOM kill on `api` pods** — for: exit code 137, restarts 6 in 20 min; against: memory limit not shown yet.
   Next (read-only): `kubectl describe pod api-7d9f-xk2 -n prod | grep -A3 "Last State"` → `OOMKilled` confirms.
2. **Bad 1.14.2 image** — for: restarts started 4 min after deploy; against: image pulled fine.
   Next: `kubectl rollout history deploy/api -n prod`

## Mitigation candidates (require `confirm <resource>`)
- rollback: `kubectl rollout undo deploy/api -n prod` — only after hypothesis 2 is confirmed.
```

## License
MIT
