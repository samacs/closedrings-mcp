# Architecture

## TL;DR

This repo is **marketplace artifacts only** — manifests, the
skill, and branding. The actual auth + transport lives in the
main Closed Rings Rails app.

## Where the work actually happens

| Concern | Lives in |
|---|---|
| MCP transport (`POST /mcp`, JSON-RPC dispatcher, tool implementations, resources, rate limiting) | Main app (`closedrings.sh`, `Mcp::ServerController`) — shipped through Phase A→E of the MCP integration epic. |
| OAuth 2.1 authorization (DCR, PKCE, consent UI, token issuance) | Main app — shipped in [closedrings.sh#98](https://github.com/samacs/closedrings.sh/pull/98). Routes wired in `config/routes.rb`; controllers under `app/controllers/oauth/`; `OauthClient` / `OauthAuthorization` models persist DCR registrations and one-time codes; tokens reuse `ApiToken` with `kind: agent`. |
| Audit log, agent-token `kind`, weekly digest, admin telemetry | Main app — already shipped through Phase A and E. |
| Marketplace manifests for Claude Desktop's connector directory, Claude Code's plugin install, Cursor's MCP marketplace | **This repo.** |
| The `closedrings:track-time` skill | Mirrored in both places: source of truth is the main app's `Mcp::Playbook::TEXT` (sent over the MCP `initialize` handshake); persistent copy is `skill/SKILL.md` in this repo (read by marketplace bundles + by users who paste into their client's rules). |

## The auth flow we're targeting

MCP authorization went into the spec in the 2025-03-26 revision and
was tightened in 2025-06-18 (the protocol version we currently
advertise from `Mcp::ServerController::PROTOCOL_VERSION`). The
shape, distilled:

```
┌────────────────────┐                              ┌──────────────────┐
│  Claude Desktop    │  1. POST /mcp (no token)     │  closedrings.sh  │
│  / Code / Cursor   │ ───────────────────────────▶ │                  │
│                    │ ◀── 401 WWW-Authenticate: ── │  /api/mcp        │
│                    │     Bearer resource_metadata=│                  │
│                    │     "…/.well-known/oauth-…"  │                  │
│                    │                              │                  │
│                    │  2. GET /.well-known/oauth-  │                  │
│                    │     protected-resource       │                  │
│                    │ ───────────────────────────▶ │                  │
│                    │ ◀── { authorization_servers: │                  │
│                    │       ["https://…"] }        │                  │
│                    │                              │                  │
│                    │  3. GET /.well-known/oauth-  │                  │
│                    │     authorization-server     │                  │
│                    │ ───────────────────────────▶ │                  │
│                    │ ◀── full OAuth metadata      │                  │
│                    │                              │                  │
│                    │  4. POST /oauth/register     │                  │
│                    │     (Dynamic Client Reg.)    │                  │
│                    │ ───────────────────────────▶ │                  │
│                    │ ◀── { client_id, client_     │                  │
│                    │       secret, … }            │                  │
│                    │                              │                  │
│                    │  5. browser → /oauth/        │                  │
│                    │     authorize?…&code_        │                  │
│                    │     challenge=…              │                  │
│                    │ ────────────browser────────▶ │                  │
│                    │                              │  user signs in,  │
│                    │                              │  sees consent,   │
│                    │                              │  clicks Approve  │
│                    │ ◀── redirect with code ──── ─┤                  │
│                    │                              │                  │
│                    │  6. POST /oauth/token        │                  │
│                    │     (code + verifier)        │                  │
│                    │ ───────────────────────────▶ │                  │
│                    │ ◀── { access_token, … }      │                  │
│                    │                              │                  │
│                    │  7. POST /mcp                │                  │
│                    │     Authorization: Bearer …  │                  │
│                    │ ───────────────────────────▶ │                  │
│                    │ ◀── tools work normally ──── │                  │
└────────────────────┘                              └──────────────────┘
```

The token at step 6 is a **`kind: agent` `ApiToken`** — the same
shape we already store for the manual `/profile/agents` flow.
The MCP transport's auth code (`Mcp::Auth`) doesn't change; only
the provisioning path does.

### Why this beats a setup CLI

- **One-click from inside the agent.** User stays in Claude
  Desktop / Cursor / wherever; never sees a terminal, never
  opens a config file.
- **The auth server is the same server they already trust** —
  `closedrings.sh`. Same SSO, same session, same audit log.
- **Token rotation, revocation, expiry** are server-side
  concerns. The MCP spec lets the server issue refresh tokens or
  short-lived access tokens; we can tighten the security posture
  over time without coordinating client releases.
- **It's what every grown-up MCP integration uses** today —
  Sentry, PostHog, Linear, Atlassian, GitHub, Notion. Convergence
  on this pattern is the reason it's worth doing right.

### How OAuth landed

The spec moved between MCP revisions while this work was in
flight; the implementation tracks the **2025-06-18** profile
that `Mcp::ServerController::PROTOCOL_VERSION` advertises.
Reference docs the implementation conforms to:

- MCP authorization spec: <https://modelcontextprotocol.io/specification>
- OAuth 2.1 draft: <https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/>
- RFC 7591 (Dynamic Client Registration)
- RFC 7636 (PKCE)
- RFC 8707 (Resource Indicators — for audience binding)

What shipped in [closedrings.sh#98](https://github.com/samacs/closedrings.sh/pull/98):

1. **Routes** (`config/routes.rb`):
   - `GET  /.well-known/oauth-protected-resource` (api host)
   - `GET  /.well-known/oauth-authorization-server` (apex)
   - `POST /oauth/register` — Dynamic Client Registration
   - `GET  /oauth/authorize` — consent UI
   - `POST /oauth/authorize` — approve form submission
   - `POST /oauth/authorize/deny` — deny submission
   - `POST /oauth/token` — PKCE exchange
2. **Models**: `OauthClient` (DCR-registered clients) and
   `OauthAuthorization` (one-time codes + PKCE verifiers). Access
   tokens reuse `ApiToken` with `kind: agent` rather than a new
   `OauthToken` model, so the existing `/profile/agents` UI and
   audit-log machinery work for both paste-flow and OAuth-flow
   tokens.
3. **`Mcp::ServerController`** returns
   `WWW-Authenticate: Bearer resource_metadata="…"` on
   unauthenticated requests, pointing at the well-known endpoint.
4. **Consent UI** lives under `app/views/oauth/authorizations/`
   in the dashboard layout. Each client gets per-user approval
   memory so "don't ask again for this client" works on the next
   authorize attempt.

Follow-on fix: [closedrings.sh#99](https://github.com/samacs/closedrings.sh/pull/99)
bypasses Turbo on the consent form so the cross-origin redirect
back to the MCP client works in Hotwire-enabled layouts.

## What this repo holds

```
closedrings-mcp/
├── README.md              one-page pitch + status
├── LICENSE                MIT
├── CHANGELOG.md           Keep-a-Changelog
├── skill/
│   └── SKILL.md           the closedrings:track-time playbook
├── manifests/             (currently empty — see below)
│   └── README.md          per-marketplace format notes
└── docs/
    ├── architecture.md    this file
    └── release-process.md SemVer + marketplace submission cadence
```

### What goes under `manifests/` (per marketplace, in rough order of priority)

- `claude-desktop/connector.json` — for Claude Desktop's
  connector directory. Includes the MCP server URL, display name,
  description, scopes, brand mark.
- `claude-code/plugin.json` — for `claude plugin install
  github.com/samacs/closedrings-mcp`. Plus `SKILL.md` at the
  repo root or under a known path (Claude Code looks in known
  conventional locations).
- `cursor/mcp.json` — for the Cursor MCP marketplace.

Exact format per marketplace will be filled in by the dev
session — these change as the marketplaces evolve and there's
no value in guessing the field names without reading the current
submission docs.

## Out of scope

- A self-hosted Closed Rings story. The OAuth flow assumes the
  authorization server is `closedrings.sh`. If a self-hosted user
  ever materializes, they configure their own endpoint URL in the
  client — the protocol doesn't care.
- A stdio MCP server bundle. The remote transport works; no
  reason to add a stdio proxy.
- Per-user / per-org branding in marketplace listings. One brand,
  one connector, one listing per marketplace.
