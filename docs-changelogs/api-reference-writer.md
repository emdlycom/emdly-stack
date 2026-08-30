---
name: api-reference-writer
owner: shiplog
category: Docs & changelogs
description: Writes an endpoint's reference page from its route, request validation and response shape — every field, every error, one working example.
version: v3
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/shiplog/api-reference-writer
raw: https://emdly.com/raw/shiplog/api-reference-writer.md
install: npx @emdly/cli add shiplog/api-reference-writer
---

# API reference writer

Reference docs are only useful when they are complete. The reader is an engineer with the endpoint open in one window and your page in the other, deciding what to send and what they will get back, so a field without a meaning is a bug in the page. This skill documents one endpoint at a time from the code that implements it, marks anything the code does not explain rather than guessing at it, and refuses to publish an endpoint whose auth model the input does not establish.

## When to use
- Per endpoint, from its handler, its validation rules and a real response sample.
- On a whole route file, to produce the index and list the endpoints that have no docs.
- When a response shape changed and the existing page no longer matches it.
- To audit an existing page: same passes, output is the diff against what is published.
- Not to design an API. This documents what the code does, including where it is wrong.

## Input

Required, per endpoint:
- **Route** — method, path with its params, and the middleware or guard stack applied to it.
- **Handler or controller source** — the actual code, not a summary.
- **Validation rules or schema** — the request contract. Every constraint on the page comes from here.
- **Response serializer / resource / DTO** — what shapes the body.
- **An actual response sample** — real captured JSON, at the endpoint's most common status.
- **Error conventions** — the shared error body shape and the handler-level mapping.

Optional:
- Non-2xx samples, one per documented status.
- The docs site's existing pages, for heading level, casing and table style.
- OpenAPI or route annotations, useful as a cross-check, never as the only source.

Two of these are load-bearing. Without the **validation rules**, the request table is unsourced and the page cannot be written — report `request contract not supplied` and stop. Without the **response sample**, see `## Edge cases`; the page runs in reduced form and says so.

## Page structure

1. **One line.** What the endpoint does, in the caller's terms, plus whether it is idempotent when the code establishes that.
2. **Auth.** The exact credential, the scope or role, and the object the permission is checked against. Read it from the guard stack, not from the path. If the input does not establish it, stop — see `## Stop and hand back`.
3. **Request.** Path params, query params, body fields. Every row: name, type, required or optional, the constraints from the validation rules, and a one-line meaning. No field without a meaning. Default values come from the code, not from the field name.
4. **Response.** The status code, then every field in the sample with type and meaning. A nested object gets its own table under its own heading. A field present in the sample but absent from the serializer is marked, not explained away.
5. **Errors.** A table of status → when it happens, derived from the validation rules and the handler: which 4xx the guard raises, which the validator raises, which the lookup raises. Include the shared error body once.
6. **Example.** One runnable `curl` with placeholders in `<angle brackets>`, followed by the response it actually returns. Both complete.
7. **Notes.** Rate limits, pagination, idempotency keys, ordering guarantees — only where the code establishes them.

## Rules

- Every constraint traces to a validation rule. Do not invent a maximum length, a page size, or an enum.
- **A field in the sample that the serializer does not explain is marked `undocumented — ask the owner`, in the table, in that wording.** Never infer a meaning from a field name. This rule exists because the guess is always plausible and sometimes wrong.
- The same applies to a status the handler can return but the conventions do not name.
- No ellipses inside a request or a response body. Real header names, real JSON, real ids.
- Types are the wire types the caller sees (`string`, `int`, `bool`, `string[]`, `object`, `null`), not the language's types.
- Match the docs site's heading levels and casing when its pages were supplied; otherwise use the shape below and say the style was not matched.
- Document what the code does. Where that contradicts the intent, document the behaviour and add a `Known mismatch` note; do not document the intent.

## Output format

```
## POST /v1/agents/{agent}/skills

Install a published skill on one of your agents. Idempotent: installing a skill that is
already present returns 201 and does not duplicate it.

**Auth:** bearer token belonging to the agent's owner. Checked against `agent.owner_id`
in `EnsureAgentOwner` middleware. No scope system on this route.

**Path params**

| param | type | required | rules | meaning |
| --- | --- | --- | --- | --- |
| agent | string | yes | 1–64 chars, `^[a-z0-9-]+$` | the agent's handle, not its numeric id |

**Body**

| field | type | required | rules | meaning |
| --- | --- | --- | --- | --- |
| skill | string | yes | matches `owner/name` | the published skill to install |
| pin | string | no | `^v[0-9]+$`, default `null` | version to pin; `null` follows latest |

**Response 201**

| field | type | meaning |
| --- | --- | --- |
| installed | bool | always `true` on success |
| skills | string[] | every skill now installed on the agent, `owner/name` form |
| revision | int | undocumented — ask the owner. Present in the sample, absent from `AgentResource`. |

**Errors** — body is the house shape: `{"error":{"code":…,"message":…,"fields":…}}`.

| status | when |
| --- | --- |
| 401 | missing or invalid bearer token |
| 403 | the agent belongs to another user |
| 404 | agent handle not found, skill not found, or skill not published |
| 409 | undocumented — ask the owner. Raised by `SkillInstaller`; not in the error conventions. |
| 422 | `skill` is not in `owner/name` form, or `pin` is not `vN` |

**Example**

curl -X POST https://emdly.com/v1/agents/claw-01/skills \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"skill":"opsmith/jira-ticket-scorer","pin":"v3"}'

{
  "installed": true,
  "skills": ["opsmith/jira-ticket-scorer", "querydeck/sql-query-explainer"],
  "revision": 4
}

**Notes**
- Rate limit: 60 requests/minute per token (`ThrottleRequests:60,1` on the route group).
- No pagination on `skills`; the array is the full list.
- Ordering of `skills` is not guaranteed by the code. Do not depend on it.
```

The refusal is also an output. It looks like this:

```
## GET /internal/agents/{agent}/telemetry — NOT DOCUMENTED

Blocked, two reasons:
1. Auth not established. The route sits behind `web` middleware only. No guard names a
   credential, and no sample carries an Authorization header. Guessing "bearer token"
   here would publish a wrong auth model for an endpoint that returns other users' data.
2. Path is under /internal and the route file marks the group `@internal`.

Nothing was written. Hand to: the route's owner (last toucher of AgentTelemetryController).
Needed to unblock: the guard that protects this route and the credential it accepts, and
a decision on whether /internal endpoints are published at all.
```

## Edge cases

- **No response sample.** The response table cannot be written from a serializer alone: computed fields, conditional includes and casts do not show there. Write the page with the response section headed `Response 2xx — shape from serializer, not verified against a sample`, list only fields the serializer emits unconditionally, and list the conditional ones under `may be present`. Do not invent a sample and do not present a synthesised one as captured.
- **No validation rules.** The request contract is the page's core. Stop and report `request contract not supplied`; do not read constraints off the database schema, which is a different contract.
- **Sample is an empty collection** (`{"data":[],"total":0}`). It establishes the envelope but no item fields. Document the envelope, and mark the item shape `not established — empty sample`.
- **Sample and serializer disagree** — a field in one and not the other. Sample field missing from the serializer: `undocumented — ask the owner`. Serializer field missing from the sample: document it and mark `not present in sample`.
- **Malformed sample** (truncated JSON, a log line, HTML). Treat it as no sample and say which it was.
- **Route file too large** — more than 40 endpoints or beyond what you can read in full. Produce the index first: method, path, handler, and one of `documented` / `no sample` / `no validation rules` / `auth not established`. Then document endpoints in that order, in batches of 10. Never document from route names alone.
- **The handler delegates** to a service, a job or a base controller you were not given. Document to the boundary and mark the rest `behaviour in <name>, source not supplied`.
- **The route has no auth at all** and the code confirms it is deliberately public. Document it as `Auth: none — public route`, and only when a guard stack was actually supplied to read. Absence of evidence is not a public endpoint.
- **Dynamic or wildcard response fields** (a `metadata` bag, user-defined keys). Document the container and its value type, state that keys are caller-defined, and do not enumerate the sample's keys as though they were the schema.
- **Deprecated endpoint.** Document it, and put the deprecation notice and its date in the one-liner — but the date itself is subject to the stop below.

## Stop and hand back

These pages ship to external callers without another review. Stop, write nothing for that endpoint, and name who decides.

- **Auth model not established by the input.** No guard stack, an unreadable middleware alias, or a sample with no credential. Emit the refusal block above. Hand to the route's owner. Never infer auth from the path prefix, from a sibling endpoint, or from the framework's default.
- **Anything internal or unreleased** — an `/internal` path, a route behind a feature flag, a handler on an unmerged branch, or anything the route file marks internal or private. Do not document it. Hand to the API owner to decide whether it is public at all.
- **A permission check you cannot locate.** The route is authenticated but nothing in the supplied code shows the ownership or tenancy check. Say so plainly; a missing tenancy check is a security finding, and it goes to whoever owns security, not into a docs page.
- **A destructive or irreversible endpoint** — deletes data, revokes credentials, cancels a subscription, moves money. Draft the page, publish nothing, and hand it to the endpoint's owner to confirm the blast radius wording and any undo path.
- **A deprecation or sunset date.** Never publish a date read from a code comment or a PR. Hand to the API owner for confirmation, the same way `shiplog/changelog-composer` holds them.
- **Code and existing published page disagree on auth or on a destructive behaviour.** Do not silently correct the page. Report both, and hand to the owner: one of the two is a live incident.

## License
MIT
