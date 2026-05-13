# closedrings-mcp

**The one-step setup tool for connecting any MCP-capable AI agent to
[Closed Rings](https://closedrings.sh) — the CLI-first time tracker.**

```sh
npx closedrings-mcp connect
```

Detects your installed MCP clients (Claude Desktop, Claude Code,
Cursor), walks you through a device-authorization flow against
`closedrings.sh`, and writes the right config file for each client.
Same psychological shape as `rings login` — no JSON editing, no
token paste, no "which file goes where."

---

## Status

🚧 **Pre-release.** Scaffold only. v0.1 ships when:

- The CLI tool can mint an agent token via device flow against the
  main app (depends on `?scope=agent` support on `/v1/auth/device/code`
  in the main repo — coordinated change).
- Auto-detection + config writers exist for at least Claude Desktop,
  Claude Code, and Cursor.

Until v0.1 is published, follow the manual setup at
[closedrings.sh/docs/mcp/overview](https://closedrings.sh/docs/mcp/overview).

## What this is (and isn't)

**This is a config tool.** It mints a `kind: agent` token against
`closedrings.sh` and writes the right MCP-server entry into each
client's config file. After that, every tool call goes directly
from the client to `https://api.closedrings.sh/mcp` — `closedrings-mcp`
is no longer in the loop.

**This is *not* a local MCP server.** Closed Rings hosts the MCP
transport at `api.closedrings.sh/mcp` (Streamable HTTP, JSON-RPC 2.0).
There's no stdio process to keep alive, no proxy to maintain.

```
┌───────────────┐  device flow  ┌────────────────────────┐
│ closedrings   │ ───────────▶  │ closedrings.sh         │
│ -mcp (npx)    │ ◀───── token  │ /v1/auth/device/code   │
└──────┬────────┘               └────────────────────────┘
       │ writes config
       ▼
┌───────────────────────────────────┐
│ ~/Library/.../claude_desktop_     │ ─── bearer auth ───▶  api.closedrings.sh/mcp
│ config.json  /  .mcp.json  /  …   │
└───────────────────────────────────┘
```

## Companion skill

Ships the `closedrings:track-time` playbook at
[`skill/SKILL.md`](skill/SKILL.md). Capable clients install it
alongside the MCP server entry so the agent knows when to call
which tool. The same content sits on the server's `initialize`
handshake — this file is the persistent, client-side copy.

## Development

```sh
git clone https://github.com/samacs/closedrings-mcp
cd closedrings-mcp
# Tooling TBD — see docs/architecture.md for the v0.1 plan
```

For local dev against your own running Rails server, point at
`https://api.lvh.me:3000/mcp` with a `cr_test_*` token. See the
[main app's setup docs](https://closedrings.sh/docs/mcp/claude-desktop)
for the URL/header shape.

## License

MIT — see [LICENSE](LICENSE). Open so anyone connecting an agent
can audit what the install code does with their token.
