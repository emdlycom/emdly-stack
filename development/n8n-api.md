---
name: n8n-api
owner: launifycorp
category: Development
description: Drive an n8n instance through its public REST API — audit workflows, diagnose failed executions, edit safely. Built around the two things that bite: a workflow PUT is a full replacement, and publishing starts a workflow firing against production. Read first, keep the baseline, never publish on your own authority.
version: v2
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/n8n-api
raw: https://emdly.com/raw/launifycorp/n8n-api.md
install: npx @emdly/cli add launifycorp/n8n-api
---

# n8n API operator

Drive an n8n instance through its public REST API: read workflows and executions, edit workflows safely, manage tags and projects, and diagnose failures without opening the editor.

The API is small and sharp, and it sits on top of a live automation system. Two things follow from that and they shape everything below. **A workflow update is a full replacement**, so a PUT built by hand deletes every node you did not include. And **publishing a workflow makes it start firing** — on its schedule, on its webhook, against whatever production systems it talks to. There is no dry run.

So the working posture is: read first, keep the baseline you read, write the whole object back, and never publish on your own authority.

## When to use

- Auditing what exists on an instance: which workflows are published, what they call, which are failing.
- A bulk edit across many workflows that would be tedious and error-prone in the editor.
- Diagnosing a failed run from execution data.
- Exporting workflows for review, diffing, or version control.
- Wiring n8n into something else, where the other system needs to read n8n's state.

Not for: building a workflow from scratch, which is faster in the editor and then exported; or running a workflow, which this API cannot do — see the capability map.

## Input

Required, and the skill does not start without all three:

- **Base URL.** Self-hosted: `https://<your-instance-url>/api/v1`. Cloud: `https://<name>.app.n8n.cloud/api/v1`.
- **An API key**, created in **Settings → n8n API**. It is sent as the header `X-N8N-API-KEY`.
- **Which instance this is** — production, staging, or a personal one. If nobody says, ask. Every write below lands somewhere real, and "I assumed staging" is not a recovery plan.

**The API is not available on the free trial.** n8n's own note: *"The n8n API isn't available during the free trial. Please upgrade to access this feature."* A 401 on a trial instance is that, not a bad key.

**Treat the key as a root credential.** Keys are created with a label and an expiry. On Enterprise you can also give a key **scopes**, limiting which resources and actions it can reach. Without Enterprise there are no scopes, so **a plain API key has unrestricted access to the account** — every workflow, every credential operation, every user. Never print it, never write it into a file that gets committed, never paste it into a report. See Stop 6.

## Capability map

Read this before planning anything. Half the requests people bring to this API are for things it does not do.

| the API can | the API cannot |
|---|---|
| create, read, update, delete workflows | **execute or trigger a workflow** |
| publish and unpublish workflows | run a workflow "once" to test it |
| read, delete, **retry** and stop executions | start an execution from nothing |
| create, update, delete, test credentials | **read any credential's secret data** |
| fetch a credential type's schema | export credentials with their values |
| list and manage tags, projects, variables, data tables | edit the visual layout meaningfully by hand |
| pull an audit report | see the editor's undo history |

**There is no endpoint that runs a workflow.** The supported way to trigger one from outside is to give it a **Webhook trigger node** and call that node's production URL. That is a different URL, a different auth scheme, and not part of this API. Say so plainly rather than hunting for an execute endpoint that does not exist.

**Credential secrets never come back.** List and get both return credential metadata with the data omitted, and listing is restricted to the instance owner and admins. You can create a credential by sending its data, and you can test one by id using the stored data, but you cannot read the stored data back. Any task shaped like "copy the credentials to the other instance" is not doable through this API, and the answer is not a workaround.

## Connect and verify

Before any other call, prove the connection and record what instance you are on.

Put the key in a curl config file rather than on the command line. A header
passed with `-H` is visible in `ps` output to every user on the box and lands in
shell history; a config file read with `-K` is neither.

One-time setup is the operator's job, not yours: they create `~/.n8n.cfg`
containing a single curl `header` directive carrying the `X-N8N-API-KEY` header
in curl config syntax, and `chmod 600` it. You never see the file's contents.

```bash
N8N="https://n8n.example.com/api/v1"
N8N_CFG="$HOME/.n8n.cfg"

curl -sS -K "$N8N_CFG" "$N8N/workflows?limit=1" -o /dev/null -w '%{http_code}\n'
```

- `200` — connected.
- `401` — bad, expired or missing key. Also the response on a free trial instance.
- `404` on a path you believe exists — check the `/api/v1` prefix is present.

Then state, in the first line of your output, which base URL you are working against and whether the operator called it production. Every later action inherits that.

## Reading

```bash
# workflows, with the fields that matter for an audit
curl -sS -K "$N8N_CFG" \
  "$N8N/workflows?limit=250&active=true" \
| jq '.data[] | {id, name, active, updatedAt, tags: [.tags[].name],
                 nodeCount: (.nodes | length),
                 triggers: [.nodes[] | select(.type | test("[Tt]rigger|webhook")) | .type]}'
```

```bash
# failing executions in a window, without dragging the payloads along
curl -sS -K "$N8N_CFG" \
  "$N8N/executions?status=error&limit=250&startedAfter=2026-08-01T00:00:00Z" \
| jq '.data[] | {id, workflowId, startedAt, stoppedAt}'
```

Execution list filters are `status` (`canceled`, `crashed`, `error`, `new`, `running`, `success`, `unknown`, `waiting`), `workflowId`, `projectId`, `startedAfter` and `startedBefore`.

**`includeData` is off by default and should usually stay off.** Execution payloads carry whatever flowed through the workflow, which routinely means customer records, tokens and message bodies. Fetch it for one execution you are diagnosing, never across a list, and never write it to a file or a report — see Stop 5.

## Pagination

Every list endpoint is cursor-paginated. The default page is **100** and the maximum is **250**. A response with more pages carries a `nextCursor`; pass it back as `cursor` to get the next page, and stop when no `nextCursor` comes back.

```bash
cursor=""; page=0
while :; do
  page=$((page+1))
  resp=$(curl -sS -K "$N8N_CFG" \
    "$N8N/workflows?limit=250${cursor:+&cursor=$cursor}")
  echo "$resp" | jq -c '.data[] | {id, name, active}'
  cursor=$(echo "$resp" | jq -r '.nextCursor // empty')
  [ -z "$cursor" ] && break
  [ "$page" -ge 50 ] && echo "stopping at 50 pages, say why" >&2 && break
done
```

Always report how many pages you walked and whether you hit your own cap. A partial listing presented as a full one is how "we have 40 workflows" becomes wrong.

## Writing a workflow

**`PUT /workflows/{id}` is a full replacement.** The required fields are `name`, `nodes`, `connections` and `settings`. Anything you leave out of the body is gone from the workflow. This is the single most destructive thing in this API and it looks like an ordinary update.

The only safe pattern, every time:

```bash
ID=abc123
# 1. read and keep the baseline — this file is your rollback
curl -sS -K "$N8N_CFG" "$N8N/workflows/$ID" \
  > "baseline-$ID-$(date -u +%Y%m%dT%H%M%SZ).json"

# 2. derive the new body from the baseline, changing only what you mean to change
jq '{name, nodes, connections, settings}
    | .nodes = [.nodes[] | if .name == "HTTP Request"
        then .parameters.url = "https://api.example.com/v2/items" else . end]' \
  "baseline-$ID-"*.json > body.json

# 3. diff before sending — read it, do not skim it
diff <(jq -S '{name,nodes,connections,settings}' "baseline-$ID-"*.json) \
     <(jq -S . body.json)

# 4. send the whole object
curl -sS -X PUT -K "$N8N_CFG" \
  -H 'Content-Type: application/json' --data @body.json "$N8N/workflows/$ID"
```

Rules that go with it:

- **Never hand-write a PUT body.** Derive it from what you read. A body assembled from the documentation will be missing something the instance had.
- **Keep the baseline file** until the change is confirmed working, and name it in your output so someone else can roll back without you.
- **Diff before sending.** If the diff has changes you did not intend, stop; that is the transform being wrong, not the API.
- **A published workflow updates and re-publishes automatically.** n8n's note: the updated version is re-published unless `publishIfActive` is set to `false`. So editing a live workflow puts your edit into production the moment the PUT returns. That is Stop 2.
- **Node IDs and positions matter.** Preserve them. Regenerating IDs breaks connections; dropping positions makes the canvas unreadable for whoever opens it next.

## Publish and unpublish

`POST /workflows/{id}/activate` and `/deactivate` still work, and n8n's reference marks them **deprecated in favour of `POST /workflows/{id}/publish` and `/unpublish`** — the v1 vocabulary was "activate". Prefer the new paths, and if an instance rejects them, fall back and say in your output which pair you used and why.

Publishing is not a configuration change. It is the moment a workflow begins running on its trigger, against production systems, with whatever credentials it holds. **This skill never publishes on its own initiative.** See Stop 1.

## Executions

- Retrieve, delete, **retry**, stop, and stop-many by filter.
- Retry accepts `loadWorkflow` to retry against the latest version of the workflow rather than the version that ran. Say which one you used — retrying an old failure against a rewritten workflow is a different experiment from re-running the original.
- Annotation tags on executions can be read and updated, which is the tidy way to mark triage state without touching the run data.

**Deleting executions destroys the audit trail.** It is often the only record of what a workflow did to a production system on a given day. See Stop 3.

## Credentials

You can create, get, update, delete, test and transfer credentials, and fetch the schema for a credential type. Get and list return metadata only: **credential data (secrets) is not included**, and listing is limited to the instance owner and admin.

Fetching a type's schema before creating a credential is the right move — it tells you the exact field names that type expects, which is not guessable.

Creating or updating a credential means handling a secret in the request body. That is Stop 4, every time, without exception.

> Thresholds above are defaults; report the thresholds you used.

## Rules

- **Read before write, always.** Every mutation is derived from a response you fetched in this session, not from the documentation and not from memory.
- **One workflow per change, one baseline per workflow.** A bulk edit is a loop over that pattern, not a bulk endpoint — there isn't one.
- **Say what you touched.** Every write appears in the output with the id, the name, the fields changed and the baseline filename.
- **Never print the API key**, and never put it on a command line. Read it through the curl config file; `-H` leaks it into `ps` and into shell history.
- **Never write execution data to a file or a report.** Quote the one field you needed and say where it came from.
- **Report counts with their pagination.** "34 workflows across 1 page" not "34 workflows".
- **A failed write is reported, not retried blind.** Read the status and the body first; a 400 on a PUT usually means the body lost a required field, and retrying sends the same broken body again.

## Output format

```
n8n · https://n8n.example.com/api/v1 · PRODUCTION (confirmed by operator)
Connected 200 · key read from ~/.n8n.cfg, never echoed · read-only unless stated

INVENTORY
41 workflows across 1 page (limit 250, no nextCursor)
  published    18
  unpublished  23
  no trigger    2   -> wf_7712 "Old import", wf_9013 "scratch"
Triggers among published: schedule 11 · webhook 5 · form 2

FAILURES  status=error, startedAfter 2026-08-01, 1 page
27 executions across 4 workflows
  wf_4410 "Invoice sync"        19   last 2026-08-29T22:14Z
  wf_5120 "Slack digest"         5
  wf_3301 "CRM upsert"           2
  wf_8890 "Backup"               1
Payloads not fetched (includeData off). Diagnosed wf_4410 from one execution.

DIAGNOSIS  wf_4410
Execution 118342, node "HTTP Request", 401 from api.vendor.example.
18 of 19 failures share that node and that status. The 19th is a timeout.
The credential attached is "Vendor API (prod)", id cr_221. I did not read it —
the API does not return credential data — and I did not run a credential test,
because that would authenticate against the vendor. Suggested next step, for a
human: run the test from the editor, or POST /credentials/cr_221/test knowingly.

CHANGE APPLIED  1 workflow
wf_5120 "Slack digest" · unpublished at time of edit
  baseline: baseline-wf_5120-20260830T131102Z.json
  changed:  nodes[3].parameters.url  ->  https://api.example.com/v2/items
  diff:     1 line, reviewed before sending
  PUT 200. Workflow remains unpublished; publishIfActive not applicable.

NOT DONE
wf_4410 is published. Its fix is a credential rotation, which is Stop 1 and
Stop 4. Baseline read and kept: baseline-wf_4410-20260830T131140Z.json.
Nothing was published, unpublished, deleted or retried.
```

The refusal:

```
NOT CONNECTED

Missing: which instance this is.

The base URL and key are present, but nobody has said whether
https://n8n.example.com is production, staging or personal. Every write in this
API lands on a live automation system, a PUT replaces a workflow entirely, and
publishing starts it firing against whatever it talks to.

Tell me which it is. If it is production, tell me that explicitly and I will
read only, and bring you the diff of any change before it goes anywhere.
```

## Edge cases

- **No base URL or no key.** Ask and stop. Do not guess a hostname.
- **`401` on a working key.** Check expiry first — keys are created with one. On a Cloud trial, the API is not available at all, and that is the same status code.
- **`404` on a documented path.** Check the `/api/v1` prefix, then the n8n version: endpoints move, and `publish`/`unpublish` are newer than `activate`/`deactivate`.
- **The instance is older than the endpoint.** Fall back to the deprecated path and record that you did.
- **An Enterprise-only resource on a community instance** — scoped keys, projects, variables, source control. Report it as unavailable on this licence rather than as an error.
- **A scoped key that cannot reach what the task needs.** Say which scope is missing. Do not route around it with a different key.
- **A workflow whose PUT returns 400.** The body lost a required field: `name`, `nodes`, `connections` or `settings`. Re-derive from the baseline; do not patch the body by hand.
- **A workflow with pinned data, or with `staticData`.** Preserve it through the round trip. Losing it silently changes behaviour in a way the diff of nodes will not show.
- **Two people editing the same workflow.** This API has no optimistic concurrency you can rely on. Re-read immediately before the PUT, and abort if it changed since your baseline.
- **More than 250 results.** Paginate. A single `limit=250` call on an instance with 900 workflows silently returns a third of them.
- **An execution list that is huge.** Filter by `status` and a time window before anything else. Never page the whole history to count something.
- **Execution data containing personal data.** Expected, not exceptional. Do not save it, do not paste it. See Stop 5.
- **A workflow that references a credential you cannot see.** Normal. Report the credential id and name from the node, and stop there.
- **The response is HTML, not JSON.** You reached a proxy, a login page or the editor, not the API. Check the prefix and stop; do not parse it.

## Stop and hand back

1. **Publishing or unpublishing anything.** Publishing starts a workflow running against production systems on its own trigger; unpublishing silently stops something that other things may depend on. Propose it, name the workflow and its trigger, and let the operator do it or say the word.
2. **Any write to a published workflow.** The update re-publishes automatically unless `publishIfActive` is `false`, so the edit is live the instant the PUT returns. Bring the diff first. If the change must happen, ask whether to unpublish, edit, and republish — which is three decisions, not one.
3. **Deleting anything.** Workflows, executions, credentials, projects. There is no undo in this API, and execution history is often the only record of what a workflow did to a customer's data. Deleting executions to "clean up" is a retention policy decision, not maintenance.
4. **Any credential operation that involves a secret** — creating, updating, or testing. Creating means a secret passes through the request body and possibly your shell history. Testing authenticates against the third-party system, which can lock an account or trip an alert. Fetch the schema, tell the operator exactly which fields the type wants, and let them enter it.
5. **Execution data that contains personal or sensitive data.** Do not save it, do not include it in a report, do not pass it to another tool. Quote the single field you needed and say which execution it came from.
6. **An API key pasted into the conversation, or a non-Enterprise key on a shared instance.** A key without scopes has unrestricted account access. If one appears in the chat, say it should be revoked and reissued rather than using it, and read it from the curl config file instead.
7. **A bulk write across more than a handful of workflows.** Run the transform on one, show the diff, get agreement on that one, and only then loop. A transform that is subtly wrong applied to sixty workflows is sixty restores from sixty baselines.
8. **Source control, projects, or user management.** These change how a team works, not how a workflow runs. Read them; do not write them.

## Sources

Facts and quotations above come from n8n's own documentation, `docs.n8n.io/connect/n8n-api/`: the authentication page for the base URL, the `X-N8N-API-KEY` header, key labels, expiry and Enterprise scopes; the pagination page for the 100 default and 250 maximum and the `nextCursor` mechanism; and the workflow, execution and credential endpoint references for the operations, the PUT-replacement semantics, the `publishIfActive` behaviour, the `activate`/`deactivate` deprecation, and the statement that credential data is not returned.

The endpoint reference is generated from n8n's OpenAPI specification, and self-hosted instances carry a built-in API playground. Check both against the version you are pointed at before trusting anything here: this API has already renamed its activation endpoints once.

## License

MIT
