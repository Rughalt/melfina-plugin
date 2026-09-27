---
name: melfina-knowledge-upsert
description: "Create or update a canonical knowledge entry — a project, service, decision, convention, runbook, person, or concept the install should know by name. Every write records a versioned snapshot with you as author, so updates never silently overwrite history. Pass expected_version to fail instead of clobbering a change someone else just made. (HTTP fallback for the `knowledge_upsert` Melfina MCP tool.)"
---

If the Melfina MCP server is available, use the corresponding MCP tool instead of this skill. Use this skill only as an HTTP fallback when MCP is unavailable.

Do not call both MCP and HTTP for the same operation.

## When to use

Create or update a canonical knowledge entry — a project, service, decision, convention, runbook, person, or concept the install should know by name. Every write records a versioned snapshot with you as author, so updates never silently overwrite history. Pass expected_version to fail instead of clobbering a change someone else just made. Use this when your runtime cannot run the Melfina
MCP server (no MCP client, or a lightweight/cloud agent) and you need the
same operation over plain JSON HTTP.

## Endpoint

    POST https://melfina.dragonshorn.cloud/api/tools/knowledge_upsert
    Content-Type: application/json

`GET https://melfina.dragonshorn.cloud/api/tools` lists the whole tool surface with scopes.
`melfina-api.json` in this package is the machine-readable manifest.

## Auth

Send a Melfina key your runtime injects — credentials never live in this
file:

    Authorization: Bearer $MELFINA_API_KEY

Key forms:

- `melf_…` dedicated agent key — your persona is included in the key.
- `melfs_…` scoped API key — this call needs the `knowledge:write` scope (403
  without it; the `all:read` preset covers every read scope and
  `all:write` every scope).
- `melfg_…` fleet key — also name your agent: `"name": "<your agent
  name>"` in the body, an `X-Melfina-Agent-Key: <your agent name>`
  header, or `?agent_key=<your agent name>` in the query (names only).
  URL-only clients can pin it as the Basic username:
  `https://<your name>:$MELFINA_API_KEY@melfina.dragonshorn.cloud/api/tools/knowledge_upsert`.

## Request

The body is the tool's arguments object — the same shape MCP `tools/call`
takes in `params.arguments`:

```json
{
    "properties": {
        "entry": {
            "description": "Canonical name of the knowledge entry \u2014 the lookup key, matched case-insensitively. (`name` stays reserved for your persona.)",
            "type": "string"
        },
        "type": {
            "description": "Entry type. Required when creating.",
            "enum": [
                "project",
                "service",
                "decision",
                "convention",
                "runbook",
                "person",
                "concept"
            ],
            "type": "string"
        },
        "aliases": {
            "description": "Other names this entry answers to.",
            "items": {
                "type": "string"
            },
            "type": "array"
        },
        "summary": {
            "description": "One-line \"what is this\" \u2014 shown in search results.",
            "type": "string"
        },
        "state": {
            "description": "Current state and important constraints \u2014 the durable facts.",
            "type": "string"
        },
        "related": {
            "description": "Canonical names of related entries.",
            "items": {
                "type": "string"
            },
            "type": "array"
        },
        "source": {
            "description": "Provenance \u2014 where this knowledge came from (doc, ticket, chat).",
            "type": "string"
        },
        "verified": {
            "description": "Mark the entry as freshly verified \u2014 bumps verified_at.",
            "type": "boolean"
        },
        "expected_version": {
            "description": "Optimistic-lock guard: only write if the entry is still at this version.",
            "type": "integer"
        },
        "name": {
            "description": "Your agent name \u2014 your persona on this install. Required when the connection authenticates with the shared fleet key; on a dedicated key it must match that key's agent.",
            "type": "string"
        }
    },
    "type": "object",
    "required": [
        "entry"
    ]
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
- `403` — a scoped key (`melfs_…`) lacking the `knowledge:write` scope.
- `404` — no such tool.
- `422` — invalid arguments; on schema violations `fields` names the
  offenders.
- `429` — too many requests (the endpoint's throttle — the response is
  the framework's, not the `ok`/`error` envelope).
- `400` — a tool-level error — the failure the MCP tool would return as
  `isError` (for example a not-found file or a rate limit), as the
  `error` string.
