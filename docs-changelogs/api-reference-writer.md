---
name: api-reference-writer
owner: shiplog
category: Docs & changelogs
description: Writes an endpoint's reference page from its route, request validation and response shape — every field, every error, one working example.
version: v2
license: MIT
updated: 2026-08-21
recommended: false
security_checked: true
url: https://emdly.com/skills/shiplog/api-reference-writer
raw: https://emdly.com/raw/shiplog/api-reference-writer.md
install: npx @emdly/cli add shiplog/api-reference-writer
---

# API reference writer

Reference docs are only useful when they are complete. This skill writes one endpoint at a time and refuses to leave a field undocumented.

## When to use
- Per endpoint, from the controller/handler, its validation rules and a sample response.
- On a whole route file to produce the index and flag endpoints without docs.

## Input
Route (method, path, auth), validation rules or schema, the response serializer/resource, an actual response sample, and the error handling conventions.

## Page structure
1. **One line:** what the endpoint does, in the caller's terms.
2. **Request:** method, path, auth requirement, path params, query params, body fields — each with type, required/optional, constraints from the validation rules, and a one-line meaning. No field without a meaning.
3. **Response:** status code, every field of the sample with type and meaning. Nested objects get their own table.
4. **Errors:** a table of status → when, derived from the validation rules and the handler (401 no token, 403 not owner, 404 not found, 422 with the field list).
5. **Example:** one runnable `curl` with placeholders in `<angle brackets>`, and the response it returns.
6. **Notes:** rate limits, idempotency, pagination — only when they exist in the code.

## Rules
- Every constraint comes from the validation rules. Do not invent limits.
- If the sample response has a field the resource does not explain, mark it "undocumented — ask the owner" rather than guessing.
- Examples must be copy-pasteable: real header names, real JSON, no ellipses inside the request.
- Match the docs site's existing headings and casing.

## Output format
```
## POST /v1/agents/{agent}/skills
Install a published skill on one of your agents. Idempotent.

**Auth:** bearer token of the agent's owner.

| body field | type | required | rules | meaning |
| skill | string | yes | `owner/name` | the skill to install |

**Response 201**
| field | type | meaning |
| installed | bool | always true on success |
| skills | string[] | every skill now on the agent |

**Errors**
| 401 | missing/invalid token |
| 403 | agent belongs to another user |
| 404 | agent or skill not found, or skill not published |
| 422 | `skill` not in `owner/name` form |

curl -X POST https://emdly.com/v1/agents/claw-01/skills -H "Authorization: Bearer <token>" -H "Content-Type: application/json" -d '{"skill":"opsmith/jira-ticket-scorer"}'
```

## License
MIT
