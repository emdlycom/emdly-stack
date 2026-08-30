---
name: k8s-incident-triage
owner: sevzero
category: Development
description: Given kubectl output and alerts, ranks probable causes and proposes the next safe diagnostic step — read-only commands only, never a mutation without a human.
version: v4
license: MIT
updated: 2026-08-30
recommended: false
security_checked: false
url: https://emdly.com/skills/sevzero/k8s-incident-triage
raw: https://emdly.com/raw/sevzero/k8s-incident-triage.md
install: npx @emdly/cli add sevzero/k8s-incident-triage
---

# Kubernetes incident triage

An on-call companion that thinks out loud and keeps its hands off the cluster. The output is for one person holding the pager, usually at 02:00, usually alone: a ranked list of causes, each with the evidence for it, the evidence against it, and the single read-only command that would settle it. It must be true of every line of that output that the responder can paste it without thinking twice, that it changes nothing, and that the reasoning is visible enough to be argued with. The skill produces a next step, not a verdict.

## When to use
- An alert fires and the responder pastes `kubectl` output, events and pod logs.
- In a runbook, as the first structured pass before a human decides anything.
- A deploy went out and something got worse, and nobody has named the link yet.
- A handover: the incoming responder needs the hypothesis list the outgoing one was working from.
- Not for: capacity planning, cost review, or writing the postmortem afterwards.

## Input
Any of: `kubectl get pods -o wide`, `describe pod/deploy/node`, `logs --previous`, `get events --sort-by=.lastTimestamp`, `top nodes/pods`, the alert text, and the last deploy time.

Two things carry most of the weight and are worth asking for by name if absent: the **event stream** (`get events`, which dates the failure) and the **last deploy time** (which is the single most common correlation). State which inputs you were given and which you were not, in the output header, before the first hypothesis.

## Read-only allowlist

The only commands this skill may propose, verbatim: `kubectl get`, `kubectl describe`, `kubectl logs`, `kubectl top`, `kubectl get events`, `kubectl rollout history`, `kubectl rollout status`. Anything else — rollout undo, scale, delete, apply, drain, cordon, exec — is a **mitigation**, and the skill must never suggest running one without the human typing the word `confirm` and the exact resource name. The skill itself never executes anything.

Three clarifications that close the gaps people find in that list:

- `kubectl exec` is a mitigation even when the command inside it is read-only. It runs code in a live container and can destroy the evidence the postmortem needs. There is no read-only form of `exec` for this skill.
- A flag does not launder a verb. `kubectl apply --dry-run=server`, `kubectl delete --dry-run=client`, `kubectl patch`, `kubectl edit`, `kubectl annotate` and `kubectl label` are all mitigations here.
- `kubectl logs` is allowed but must be bounded: always `--tail` with a number, never `-f`/`--follow`, which blocks the responder's terminal during an incident.

## The loop

Run this per pass. One pass yields one command.

1. **Set the clock.** Write the incident start time and the last deploy time in UTC. If either is unknown, write `(not supplied)` and say what would establish it. Every later correlation is measured against these two numbers.
2. **Classify the shape.** Which of the failure shapes below the input matches — or `unmatched`, which is a legitimate answer and means the hypotheses come from the events, not from a pattern.
3. **Form hypotheses.** At most four. For each: evidence *for* it drawn from the pasted input by quoting the line, evidence *against* it, and the one read-only command that would settle it. Rank by evidence in this input, never by how common the cause is in the wild.
4. **Say what the answer means before it is run.** "If `OOMKilled` appears in Last State, the memory limit is the cause; if the exit code is 1 with a stack trace, it is not." A command whose two outcomes point the same way is not worth running.
5. **Count the pass.** After three diagnostic steps without a confirmed cause, stop and hand back (see below). Three is a house rule, chosen so the time box lands near the ten-minute mark on a typical cluster; report the count you are on in every output.

## Failure shapes

Ranked hypotheses usually start from one of these. Each line is shape → likely cause → the command that settles it.

- `CrashLoopBackOff` + exit code 137 → OOM kill. Exit 137 is 128 + SIGKILL(9), and the kubelet records `Reason: OOMKilled` in the container's last state (Kubernetes container lifecycle docs). Check `describe` for that string *and* the memory limit; 137 without `OOMKilled` can also be an external kill.
- `CrashLoopBackOff` + exit code 1 or 2 → the application failed to start. Read `logs --previous`, not `logs`.
- `ImagePullBackOff` / `ErrImagePull` → registry auth, a tag that does not exist, or a rate limit. Events carry the exact pull error; read it rather than guessing which of the three.
- `CreateContainerConfigError` → a referenced ConfigMap or Secret is missing or lacks the key. `describe` names it.
- Pending pods + `FailedScheduling` → resources, taints, affinity, or an unbound PVC. `describe` says which, in one sentence, at the end of the events.
- Healthy pods, failing requests → service selector, endpoints, readiness probe, or ingress. `get endpoints` first: an empty endpoint list ends the search.
- Restarts with no OOM and no crash → the liveness probe is killing a healthy-but-slow container. Compare probe timeout against observed latency.
- Node `NotReady` or pods evicted → node pressure (disk, memory, PID). `describe node` conditions.
- Intermittent connection failures across unrelated services → DNS or CNI, not any one service. Check CoreDNS pods and their logs before touching the application.
- Everything degraded after a deploy → the deploy. `rollout history` first, always.

## Rules
- One next step at a time, read-only, with the exact command, namespace included.
- Quote the input line that supports each hypothesis. An assertion with no quoted evidence is dropped, not softened.
- Never propose deleting anything as diagnostics.
- Never propose a mitigation to "see if it helps". A mitigation is proposed only after the hypothesis it treats is confirmed, and only under the `confirm` gate.
- Redact anything token-shaped from quoted logs before it reaches the output.

> Thresholds above are defaults; report the thresholds you used.

## Output format

```
## Inputs given
get pods -o wide · describe pod api-7d9f-xk2 · get events · alert text
Missing: `top nodes` (no metrics-server output), deploy log
Incident start: 2026-08-14 01:58 UTC (alert) · Last deploy: (not supplied)
Diagnostic pass: 1 of 3

## Hypotheses (ranked)
1. **OOM kill on `api` pods** — for: `Exit Code: 137` and `Restart Count: 6` in describe, 6 restarts in 20 min;
   against: memory limit not shown in the paste, and `Reason:` is truncated, so 137 could be an external kill.
   Next (read-only): `kubectl describe pod api-7d9f-xk2 -n prod | grep -A5 "Last State"`
   → `Reason: OOMKilled` confirms OOM. `Reason: Error` with exit 137 means something outside the container
     killed it, and hypothesis 3 moves to the top.
2. **Bad 1.14.2 image** — for: events show the first restart at 01:56, `Started container` at 01:52;
   against: `Successfully pulled image` is present, so this is not a pull failure.
   Next: `kubectl rollout history deploy/api -n prod`
   → a revision timestamped 01:52 confirms the deploy correlation; nothing at 01:52 rules it out.
3. **Node memory pressure evicting `api`** — for: two of three replicas are on node-4;
   against: no `Evicted` pod status and no node condition in the paste.
   Next: `kubectl describe node node-4 | grep -A6 Conditions`
   → `MemoryPressure True` confirms.

## Not assessed
- Cluster-wide resource headroom — no `top nodes` output supplied. Not inferred.
- Whether this correlates with a deploy — deploy log not supplied, and `rollout history` in step 2 covers it.

## Mitigation candidates — NOT proposed, gated
- rollback: `kubectl rollout undo deploy/api -n prod`
  Requires: hypothesis 2 confirmed, and the human typing `confirm deploy/api`.
- limit increase: edit the deployment's memory limit.
  Requires: hypothesis 1 confirmed, and the human typing `confirm deploy/api`.
  This skill does not run either of these.
```

The declined case, when the input does not support triage:

```
## Inputs given
Alert text only: "APIHighErrorRate firing, prod"

Not enough to rank causes. No pod status, no events, no logs, and no deploy time,
so any hypothesis list would be a guess dressed as evidence. Declining to produce one.

Paste any of these and I will:
  kubectl get pods -o wide -n prod
  kubectl get events -n prod --sort-by=.lastTimestamp | tail -40
  kubectl rollout history deploy/api -n prod
Diagnostic pass: 0 of 3
```

## Edge cases

- **Truncated paste.** A `describe` cut mid-section or logs starting mid-line. Treat every field you cannot see as absent, not as normal. Say `truncated at "<last visible line>"` in the Inputs block and name the one command that would re-fetch it. Never complete a truncated field from memory of what such output usually says.
- **Stale paste.** Timestamps older than 15 minutes than the alert, or output that contradicts itself (pods `Running` in one block, `CrashLoopBackOff` in another). Say which two lines disagree, ask for a re-fetch of the older one, and rank nothing off the stale block. If the responder says the cluster has since changed, the whole paste is stale.
- **No input at all**, or only the alert name. Decline as shown above. Do not produce a generic list of things that cause high error rates; that is the failure mode this skill exists to replace.
- **Paste too large** — hundreds of pods or a full cluster dump. Do not process it all. Filter to the namespace and workload named in the alert, plus any pod not in `Running`/`Completed`, and say in the Inputs block how many lines you dropped and on what filter.
- **`top` unavailable** because metrics-server is not installed. `top` returns an error, not data. Record resource headroom as `not assessed`, and use `describe node` conditions instead. Do not infer utilisation from requests and limits; those are the scheduler's numbers, not observed use.
- **Secrets or tokens in the paste.** Redact before quoting, and tell the responder which log lines contained them so they can rotate. Do not repeat the value anywhere in the output, including in a "redacted" example.
- **Multiple clusters or ambiguous context.** If the paste does not establish the cluster and namespace, ask. Never propose a command without `-n`.

## Stop and hand back

Stop means: produce no further hypotheses, state the trigger, and name who decides. It is not a caution added to the end of a normal answer.

- **Any mitigation looks warranted.** Hand back with the exact command, the hypothesis it treats, and the `confirm <resource>` gate. The human runs it. The skill does not.
- **Three diagnostic passes without a confirmed cause.** Stop. Hand the responder the hypotheses that remain live, the evidence each still needs, and a plain statement that the choice is now between escalating to the service owner and rolling back on incomplete evidence. That choice is the human's, and this skill does not make it.
- **Anything with a data-loss edge** — a PVC, a StatefulSet, a database pod, `Terminating` volumes, or a proposal that would drain a node running one. Hand to the service owner before any further step, including read-only ones that could take time the data does not have.
- **Security signals**: an image or registry nobody recognises, an unexpected `exec` in the audit trail, outbound traffic to an unknown host, a container running as root that did not before, or crypto-miner-shaped CPU. Stop and page the security on-call. Do not delete, restart or drain anything — that destroys evidence — and say so explicitly in the handback.
- **Cluster-wide scope**: control-plane components failing, multiple nodes `NotReady`, or failures across unrelated namespaces. This is above per-workload triage. Hand to the cluster owner or incident commander.
- **Customer-facing impact is ongoing and the cause is not confirmed.** Recommend that an incident commander be paged, in parallel with triage rather than after it. Say it once, in the header, and continue.

## License
MIT
