# closedrings-mcp

**Marketplace artifacts for connecting MCP-capable AI agents to
[Closed Rings](https://closedrings.sh).**

There's no software in this repo — the MCP server is hosted on
the main app at `api.closedrings.sh/mcp`. What lives here is the
plumbing for one-click discovery + one-click connect across the
agent ecosystem:

- **The skill** — `skill/SKILL.md`, the `closedrings:track-time`
  playbook agents read to know *when* to call which tool.
- **Marketplace manifests** — listings for Claude Desktop's
  connector directory, Claude Code's plugin install path, the
  Cursor MCP marketplace.
- **Branding assets** — logo + screenshots used in each listing.

When you finish a "connect Closed Rings" flow in any of those
clients, the auth handshake is **the MCP-spec OAuth 2.0 flow** —
the client opens a browser to `closedrings.sh/oauth/authorize`,
you sign in (if not already), see a consent screen, and click
Approve. No token paste, no JSON editing.

---

## Status

🚧 **Pre-release** — server-side OAuth has landed
([PR #98](https://github.com/samacs/closedrings.sh/pull/98));
remaining gates for v0.1.0 are the production deploy and the
Claude Desktop screenshot captures. Until then, follow the manual
setup at
[closedrings.sh/docs/mcp/overview](https://closedrings.sh/docs/mcp/overview).

## Install in Claude Code

This repo IS a Claude Code plugin marketplace — install with one
command from a Claude Code session:

```sh
claude plugin marketplace add samacs/closedrings-mcp
claude plugin install closedrings@closedrings
```

That registers the remote MCP server at `https://api.closedrings.sh/mcp`
and the `closedrings:track-time` skill. Your first tool call kicks
off the OAuth flow — Claude Code reads the `WWW-Authenticate`
header on the 401, walks discovery, registers itself via DCR,
opens the browser for consent, and exchanges the code for a token.

Plugin layout follows Claude Code's conventions:

```
.claude-plugin/marketplace.json           ← marketplace catalog
plugins/closedrings/
├── .claude-plugin/plugin.json            ← plugin manifest (mcpServers + metadata)
└── skills/closedrings-track-time/
    └── SKILL.md                          ← auto-invoked when the user mentions time tracking
```

See [`manifests/claude-code/README.md`](manifests/claude-code/README.md)
for the iteration workflow + verification commands.

## What this is *not*

A common misconception worth flagging up front, because I had it
myself when I started this repo:

- **Not an npm package.** There's no `npx closedrings-mcp connect`
  CLI tool. Auth lives in the MCP protocol now (since the 2025-03-26
  revision) — clients handle the OAuth dance natively.
- **Not a local MCP server.** The remote MCP transport at
  `api.closedrings.sh/mcp` is the only Closed Rings MCP server.
- **Not a token-paste setup helper.** Once the OAuth path is live
  on the main app, paste-token flows go away.

This repo is the *discovery layer* — the bit that puts Closed
Rings into the marketplaces users browse, with the right
manifests, descriptions, and branding for each.

## Architecture in one diagram

```
Claude Desktop / Claude Code / Cursor
        │
        │   1. user adds the connector by URL
        │   2. unauthenticated request gets 401 with
        │      WWW-Authenticate pointing at the
        │      /.well-known/oauth-protected-resource endpoint
        ▼
   closedrings.sh
   ├── /.well-known/oauth-protected-resource    ──┐
   ├── /.well-known/oauth-authorization-server    │
   ├── /oauth/register     (Dynamic Client Reg.)  │  ← server-side
   ├── /oauth/authorize    (consent UI)           │   work in the
   ├── /oauth/token        (PKCE)                 │   main repo
   └── /api/mcp            (existing transport;   ──┘
                           accepts the token
                           OAuth minted)
```

The token OAuth mints is a `kind: agent` `ApiToken` — the same
shape we already have for the manual `/profile/agents` flow. The
MCP transport's auth code path doesn't change; only the
provisioning path does.

For the full server-side spec, see
[`docs/architecture.md`](docs/architecture.md).

## Companion skill

`skill/SKILL.md` ships the `closedrings:track-time` playbook —
trigger phrases, project detection, confirm-writes rule,
retroactive vs live, end-of-day formatting, recipes. Stays in
sync with the server's `Mcp::Playbook::TEXT` (which is what gets
sent via `serverInfo.instructions` on the MCP `initialize`
handshake).

If your client supports persistent rules / custom instructions /
plugin-installed skills, point it at this file's URL:

```
https://raw.githubusercontent.com/samacs/closedrings-mcp/main/skill/SKILL.md
```

## License

MIT — see [LICENSE](LICENSE). The manifests and skill are open
so any marketplace reviewer can audit them.
