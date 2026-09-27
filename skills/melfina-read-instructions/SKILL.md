---
name: melfina-read-instructions
description: "Read your instruction layers: base content plus ordered sub-layers. (HTTP fallback for the `read_instructions` Melfina MCP tool.)"
---

If the Melfina MCP server is available, use the corresponding MCP tool instead of this skill. Use this skill only as an HTTP fallback when MCP is unavailable.

Do not call both MCP and HTTP for the same operation.

## When to use

Read your instruction layers: base content plus ordered sub-layers. Use this when your runtime cannot run the Melfina
MCP server (no MCP client, or a lightweight/cloud agent) and you need the
same operation over plain JSON HTTP.

## Endpoint

    POST https://melfina.dragonshorn.cloud/api/tools/read_instructions
    Content-Type: application/json

`GET https://melfina.dragonshorn.cloud/api/tools` lists the whole tool surface with scopes.
`melfina-api.json` in this package is the machine-readable manifest.

## Auth

Send a Melfina key your runtime injects — credentials never live in this
file:

    Authorization: Bearer $MELFINA_API_KEY

Key forms:

- `melf_…` dedicated agent key — your persona is included in the key.
- `melfs_…` scoped API key — this call needs the `instructions:read` scope (403
  without it; the `all:read` preset covers every read scope and
  `all:write` every scope).
- `melfg_…` fleet key — also name your agent: `"name": "<your agent
  name>"` in the body, an `X-Melfina-Agent-Key: <your agent name>`
  header, or `?agent_key=<your agent name>` in the query (names only).
  URL-only clients can pin it as the Basic username:
  `https://<your name>:$MELFINA_API_KEY@melfina.dragonshorn.cloud/api/tools/read_instructions`.

## Request

The body is the tool's arguments object — the same shape MCP `tools/call`
takes in `params.arguments`:

```json
{
    "properties": {
        "name": {
            "description": "Your agent name \u2014 your persona on this install. Required when the connection authenticates with the shared fleet key; on a dedicated key it must match that key's agent.",
            "type": "string"
        }
    },
    "type": "object"
}
```

## Response

`200 OK` returns `{"ok": true, "result": …, "resultType": "…"}`.
`result` is this tool's JSON payload, decoded to objects/arrays (`resultType` is `json`). Multi-part results surface as `resultType: "content"`
with `result.content` holding the MCP content parts.

Errors return `{"ok": false, "error": "…"}` with a non-2xx status:

- `401` — no key, an unknown or revoked key, a disabled agent, a fleet
  key with no persona, or a `name` claim that does not match the key's
  agent.
- `403` — a scoped key (`melfs_…`) lacking the `instructions:read` scope.
- `404` — no such tool.
- `422` — invalid arguments; on schema violations `fields` names the
  offenders.
- `429` — too many requests (the endpoint's throttle — the response is
  the framework's, not the `ok`/`error` envelope).
- `400` — a tool-level error — the failure the MCP tool would return as
  `isError` (for example a not-found file or a rate limit), as the
  `error` string.
