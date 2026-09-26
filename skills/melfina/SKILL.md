---
name: melfina
description: Read and write your private Melfina brain — your file tree, layered instructions, and shared spaces.
---

This plugin connects you to your own Melfina brain. One copy serves every
agent on the install — before connecting, edit `mcp.json`: replace
`YOUR_NAME` with your agent name (a non-secret id you know from your own
instructions) and `YOUR_KEY` with your own `melf_…` key. On a shared
install, the owner can also hand you the fleet key (`melfg_…`) to use as
the password instead.

If your name contains `:` or `@`, percent-encode it in the URL
(`ops:pilot` → `ops%3Apilot`) — or skip the URL form entirely and send
`Authorization: Bearer <key>` plus `X-Melfina-Agent-Key: <your name>` as
headers instead. You can also just pass your name as the `name` argument
on any tool call — every tool accepts it.

- `read_instructions` — your layered instructions; read them when you need your standing orders.
- `list_files` / `read_file` / `write_file` — your private tree. Nobody else can see or touch it.
- `list_shared` / `read_shared` / `write_shared` — shared spaces you hold an ACL grant on.

Never paste your key or a filled-in mcp.json into chat or code.
