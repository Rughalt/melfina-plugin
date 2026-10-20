---
name: melfina-write-file-binary
description: "Write a binary file (image, PDF, any non-text bytes) to your private file tree. Send the raw bytes in `content_base64` (base64-encoded); the stored mime is always sniffed from the actual bytes, so a wrong `mime` claim cannot stick. UTF-8 text written this way still lands as a normal text file. For payloads over ~1 MB prefer `PUT /api/files/{path}` — raw bytes, no base64 overhead. (HTTP fallback for the `write_file_binary` Melfina MCP tool.)"
---

If the Melfina MCP server is available, use the corresponding MCP tool instead of this skill. Use this skill only as an HTTP fallback when MCP is unavailable.

Do not call both MCP and HTTP for the same operation.

## When to use

Write a binary file (image, PDF, any non-text bytes) to your private file tree. Send the raw bytes in `content_base64` (base64-encoded); the stored mime is always sniffed from the actual bytes, so a wrong `mime` claim cannot stick. UTF-8 text written this way still lands as a normal text file. For payloads over ~1 MB prefer `PUT /api/files/{path}` — raw bytes, no base64 overhead. Use this when your runtime cannot run the Melfina
MCP server (no MCP client, or a lightweight/cloud agent) and you need the
same operation over plain JSON HTTP.

## Endpoint

    POST https://melfina.dragonshorn.cloud/api/tools/write_file_binary
    Content-Type: application/json

`GET https://melfina.dragonshorn.cloud/api/tools` lists the whole tool surface with scopes.
`melfina-api.json` in this package is the machine-readable manifest.

## Auth

Send a Melfina key your runtime injects — credentials never live in this
file:

    Authorization: Bearer $MELFINA_API_KEY

Key forms:

- `melf_…` dedicated agent key — your persona is included in the key.
- `melfs_…` scoped API key — this call needs the `files:write` scope (403
  without it; the `all:read` preset covers every read scope and
  `all:write` every scope).
- `melfg_…` fleet key — also name your agent: `"name": "<your agent
  name>"` in the body, an `X-Melfina-Agent-Key: <your agent name>`
  header, or `?agent_key=<your agent name>` in the query (names only).
  URL-only clients can pin it as the Basic username:
  `https://<your name>:$MELFINA_API_KEY@melfina.dragonshorn.cloud/api/tools/write_file_binary`.

## Request

The body is the tool's arguments object — the same shape MCP `tools/call`
takes in `params.arguments`:

```json
{
    "properties": {
        "path": {
            "description": "Path of the file to write, e.g. \"img/logo.png\".",
            "type": "string"
        },
        "content_base64": {
            "description": "Raw file bytes, base64-encoded (replaces existing). Bytes over ~1 MB belong on PUT /api/files/{path} instead.",
            "type": "string"
        },
        "mime": {
            "description": "Declared content type, e.g. \"image/png\". The stored mime always comes from sniffing the actual bytes.",
            "type": "string"
        },
        "name": {
            "description": "Your agent name \u2014 your persona on this install. Required when the connection authenticates with the shared fleet key; on a dedicated key it must match that key's agent.",
            "type": "string"
        }
    },
    "type": "object",
    "required": [
        "path",
        "content_base64"
    ]
}
```

## Response

`200 OK` returns `{"ok": true, "result": …, "resultType": "…"}`.
`result` is this tool's text payload verbatim (`resultType` is `text`) — file contents stay a string even when they look like JSON. Multi-part results surface as `resultType: "content"`
with `result.content` holding the MCP content parts.

Errors return `{"ok": false, "error": "…"}` with a non-2xx status:

- `401` — no key, an unknown or revoked key, a disabled agent, a fleet
  key with no persona, or a `name` claim that does not match the key's
  agent.
- `403` — a scoped key (`melfs_…`) lacking the `files:write` scope.
- `404` — no such tool.
- `422` — invalid arguments; on schema violations `fields` names the
  offenders.
- `429` — too many requests (the endpoint's throttle — the response is
  the framework's, not the `ok`/`error` envelope).
- `400` — a tool-level error — the failure the MCP tool would return as
  `isError` (for example a not-found file or a rate limit), as the
  `error` string.
