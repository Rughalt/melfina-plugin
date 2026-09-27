---
name: melfina
description: Read and write your private Melfina brain — your file tree, layered instructions, and shared spaces.
---

This plugin connects you to your own Melfina brain. One copy serves every
agent on the install — before connecting, edit `mcp.json`: replace
`YOUR_KEY` in the Authorization header with your own `melf_…` key, or with
the install's fleet key (`melfg_…`) when the owner handed you that instead.

A dedicated `melf_…` key carries your persona already. On a fleet key the
key only opens the endpoint — claim your persona by passing your agent
name (a non-secret id you know from your own instructions) as the `name`
argument on every tool call, or set a fixed `X-Melfina-Agent-Key: <your
name>` header when your client supports headers. URL-only clients can pin
it as the Basic username — `https://NAME:<key>@host/mcp`, percent-encode
`:` or `@` (`ops:pilot` → `ops%3Apilot`).

- `read_instructions` — your layered instructions; read them when you need your standing orders.
- `list_files` / `read_file` / `write_file` — your private tree. Nobody else can see or touch it.
- `list_shared` / `read_shared` / `write_shared` — shared spaces you hold an ACL grant on.
- `knowledge_lookup` / `knowledge_search` / `knowledge_upsert` — the shared internal knowledge base; check it before any external search.

Never paste your key or a filled-in mcp.json into chat or code.
## HTTP fallback skills

Melfina provides internal tools and memory. Before using a Melfina HTTP skill, check whether the Melfina MCP server is available. Prefer MCP whenever possible.

If your runtime cannot run an MCP server, every catalogued tool is also
callable over plain JSON HTTP — see the generated skills under
`skills/melfina-*/` (one per catalogued tool) and the machine-readable
`melfina-api.json`. Each skill repeats the precedence rule and forbids
calling both paths for the same operation.

HTTP calls authenticate with a Bearer key injected by your runtime —
`Authorization: Bearer $MELFINA_API_KEY`. Credentials are never stored
in these files.
