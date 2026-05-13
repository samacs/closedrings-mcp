# Architecture

## What this is

A small Node CLI (`closedrings-mcp`) that **configures MCP clients to
talk to the Closed Rings hosted MCP server.** After it finishes,
every tool call goes directly from the client to
`https://api.closedrings.sh/mcp` — this package is no longer in the
data path.

```
                              ┌──────────────────────────┐
                              │  closedrings.sh          │
                              │                          │
   ┌──────────────────┐       │  POST /v1/auth/          │
   │ closedrings-mcp  │──────▶│       device/code        │  ← scope=agent
   │   (npx)          │       │  WS  /cable              │
   │                  │◀──────│  POST /v1/...            │
   └──────┬───────────┘       │       (token broadcast)  │
          │                   └──────────────────────────┘
          │ writes server entry
          ▼
   ┌──────────────────────────────────────────┐
   │ claude_desktop_config.json / .mcp.json / │
   │ .cursor/mcp.json / ...                   │
   └──────┬───────────────────────────────────┘
          │ bearer auth + JSON-RPC 2.0
          ▼
   ┌──────────────────────────────────────────┐
   │ api.closedrings.sh/mcp                   │
   │   Mcp::ServerController (Rails app)      │
   │   Tools: current_context, list_projects, │
   │          list_tasks, start_time_block,   │
   │          stop_*, log_*, update_*,        │
   │          delete_*, today_snapshot,       │
   │          recent_time_blocks              │
   │   Resources: closedrings://current,      │
   │              closedrings://projects      │
   └──────────────────────────────────────────┘
```

## What this is not

- **Not** a local stdio MCP server. The hosted remote server already
  exists; running a stdio proxy just adds latency and a process to
  manage.
- **Not** a token cache. Tokens live in each client's config file
  (where the client expects them to live).
- **Not** an SDK. Tool-calling agents talk MCP directly; this CLI is
  for one-time setup.

## Why a separate repo

The main Rails app (`closedrings.sh`) is the **server** — schema,
domain logic, hosted MCP transport. This package is the **client
ergonomics** — install, configure, diagnose. They evolve at different
speeds:

- The server moves with the product (new tools, new resources,
  business logic).
- This plugin moves with the agent ecosystem (new clients to
  support, new install conventions, marketplace requirements).

Coupling them in one repo would force the plugin's release cadence
to match the main app's, which doesn't make sense — most plugin
updates are pure client-side improvements with no server dependency.

## Versioning

Semantic Versioning:

- **Patch (0.1.x)** — bug fixes in the install flow, config writers,
  or platform support. No server change required.
- **Minor (0.x.0)** — new client supported, new commands, additive
  config fields. May require a minimum server version (documented
  in `CHANGELOG.md` + `docs/release-process.md`).
- **Major (x.0.0)** — breaking changes to install UX, command names,
  or required server version. Rare.

## Compatibility matrix

The plugin announces its minimum supported server version in
`package.json` and at the top of `CHANGELOG.md`. The server returns
its version in the MCP `initialize` handshake's `serverInfo.version`,
so `closedrings-mcp doctor` can warn on mismatch.

| Plugin version | Min server version | Notable changes |
|----------------|--------------------|-----------------|
| 0.1.0          | TBD                | First release   |

## Initial command surface

Designing toward this v0.1 shape (subject to revision during
implementation):

```sh
# Mint an agent token via device flow, write configs for every
# detected client. Idempotent — re-running updates the entry.
closedrings-mcp connect [--client <name>]

# Show which clients are configured.
closedrings-mcp status

# Revoke the token on the server and remove the entry from the
# named client's config. Defaults to every client this package
# previously configured.
closedrings-mcp disconnect [--client <name>]

# Diagnose: server reachable? TLS trusted? token valid? client
# picked it up? Each check has a single-line green/yellow/red
# verdict.
closedrings-mcp doctor

# Print the skill manifest to stdout, for clients that consume
# skills via stdin or pipe-install.
closedrings-mcp skill
```

The CLI's behaviour for env vs paths is encapsulated in a small
`config.ts`-style module: API URL (defaults to
`https://api.closedrings.sh`, override with `CLOSEDRINGS_API_URL`),
client config paths, OS detection.

## Client detection

A small registry of supported clients keyed by name:

```
claude-desktop  →  ~/Library/Application Support/Claude/claude_desktop_config.json
                   (or %APPDATA%\Claude\..., ~/.config/Claude/...)
claude-code     →  ~/.claude.json   (or project-scoped .mcp.json)
cursor          →  ~/.cursor/mcp.json    (or project-scoped .cursor/mcp.json)
```

Each entry exposes:

- `detect()` → boolean (does the config file or surrounding app
  exist on this machine?)
- `read()` → current config object, with safe fallbacks for missing
  files
- `write(config)` → atomic write (write tmp → rename), preserving
  any existing `mcpServers.*` entries that aren't ours

## Out of scope (for v0.1)

- Multi-account support (running the flow against a non-default
  workspace). The server doesn't have orgs yet anyway.
- Auto-update of an existing `closedrings` entry when the user's
  agent token rotates. Tokens are long-lived; rotation is manual
  (via `disconnect` + `connect`).
- Local stdio MCP server mode. Maybe in a future release if a
  self-hosted Closed Rings user requests it.
