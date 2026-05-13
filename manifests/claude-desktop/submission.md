# Closed Rings — Anthropic Connector Directory submission

**Submitter:** Saul Martinez · saul@closedrings.sh · maintainer of [closedrings.sh](https://closedrings.sh)
**Connector URL:** `https://api.closedrings.sh/mcp`
**Source repository (this file):** [github.com/samacs/closedrings-mcp](https://github.com/samacs/closedrings-mcp)
**Last reviewed:** 2026-05-13

---

## What it does, in one paragraph

Closed Rings is a CLI-first time tracker built around the Apple Activity
Rings metaphor. The MCP integration makes Claude a first-class client:
while the user is already in chat, the agent can read what they're
working on, start and stop the live timer, log past work retroactively,
and answer end-of-day questions from real data — without the user
context-switching out of chat. Everything goes through the same domain
layer the dashboard and CLI use, with model-level overlap validation,
an audit trail, and per-token "last seen client" labels.

## Why this belongs in the directory

- **Native fit.** Time tracking that requires the user to switch *to*
  the time tracker is a contradiction. AI agents that already know what
  the user is working on are the right surface. Closed Rings is the
  first MCP-native time tracker shipped with the spec OAuth path.
- **Read-write parity with the dashboard, gated behind consent.** No
  shadow admin surface — the only privileged operation the agent can
  do is the same one the user can already do via the dashboard, audited
  to the same trail, revocable from the same UI.
- **Deliberately conservative tool surface.** No `create_project` or
  `create_task` — agents will hallucinate "Project: refactoring stuff"
  and turn a user's project list into garbage. Project / task
  curation stays in the dashboard. See `connector.json` → `tools[]`.

## Reproduction steps for the review team

The flow is fully self-serve. No coupon, no test tenant.

1. **Sign up** at [closedrings.sh/sign-up](https://closedrings.sh/sign-up)
   (free trial, no credit card).
2. In Claude Desktop → Settings → Connectors → **Add custom connector**.
3. Paste `https://api.closedrings.sh/mcp` as the URL.
4. Claude Desktop registers itself via Dynamic Client Registration,
   opens the browser at `https://closedrings.sh/oauth/authorize?...`,
   you approve, and the browser bounces back to the loopback URL
   Claude Desktop is listening on.
5. Try a tool: ask the agent "What projects do I have?" or "Start
   tracking 30 minutes on the first project."
6. Verify the audit trail at `/profile` (your own row) and the connection
   listing at `/profile/agents` (will show "Claude Desktop (OAuth)").

If anything fails between steps 3–5, the most likely cause is the
OAuth metadata round trip — `curl https://api.closedrings.sh/.well-known/oauth-protected-resource`
should return JSON with `authorization_servers` pointing at
`https://closedrings.sh`. If you'd rather we provision a sandbox
tenant pre-loaded with sample projects and a dummy week of time
blocks, email saul@closedrings.sh and we can have one ready
within a business day.

## Auth in detail

Implements the MCP authorization spec as of revision **2025-06-18**:

| Layer | What we do |
|---|---|
| Discovery | RFC 9728 protected-resource metadata on `api.closedrings.sh`; RFC 8414 authorization-server metadata on `closedrings.sh`. |
| Registration | RFC 7591 Dynamic Client Registration at `POST /oauth/register`. Anonymous; mints `client_id` only. Public clients only (`token_endpoint_auth_method: "none"`). |
| Authorization | OAuth 2.1 §4.1 authorization-code grant with PKCE S256 (no `plain`). RFC 8707 `resource` parameter required and validated against the canonical MCP URI. |
| Token issuance | The OAuth access token IS our `kind: agent` `ApiToken` — same wire format as the existing manual paste flow, same revocation path, same audit trail. Tokens carry the audience they were issued for (`api_tokens.resource`) and are rejected by the MCP transport if presented to the wrong resource server. |
| Token rejection | `WWW-Authenticate: Bearer realm="closedrings", error="invalid_token", resource_metadata="…"` per RFC 9728 §5.1 on every 401. |
| Refresh | Not in v1 — issued tokens are long-lived and revocable from `/profile/agents`. Will add rotation if a marketplace client requires it. |

Three audit kinds fire during a normal connect: `oauth.client.registered`,
`oauth.consent.granted`, `agent_token.created`. A revoke fires
`agent_token.revoked`. Sample audit row JSON is available under
`/admin/audit` for the maintainer's account; can email a redacted
sample to the review team on request.

## Security posture

- **TLS 1.2+** via Cloudflare in front of Heroku (US East).
- **No client secrets stored.** All v1 clients are public; PKCE alone
  authenticates the token exchange.
- **Authorization codes are SHA-256 digested** before persistence
  (10-minute TTL, single-use, consumed-or-fail in a transaction).
  Plaintext code is only present on the in-memory record returned to
  the controller for the redirect.
- **API tokens** carry the same SHA-256-hashed-at-rest treatment as
  the existing CLI tokens.
- **Audience binding (RFC 8707)** rejects any OAuth-minted token
  presented to a resource server it wasn't issued for.
- **Brakeman** clean on the OAuth implementation
  ([PR #98](https://github.com/samacs/closedrings.sh/pull/98)).
- **Incident response:** security@closedrings.sh.
- **Data retention:** time blocks for the account lifetime; audit
  events 365 days; tokens until revoked.

## Branding assets

The `assets/` subdirectory under this manifest contains the connector
icon at the four standard sizes (1024 / 512 / 256 / 128 px), the
SVG master, and four screenshot captures of the consent flow + audit
trail. All assets are MIT-licensed in this repo so the review team
can use them in the directory listing.

If Anthropic prefers a different aspect ratio or image format we
haven't shipped, ping saul@closedrings.sh and we'll re-export
within 24 hours.

## Open questions for the review team

These are pending the review process; happy to align however
Anthropic prefers.

1. **Listing display copy.** The `short_description` and
   `long_description` in `connector.json` reflect what would feel
   honest in the dashboard. Open to editing for tone or length to
   match directory norms.
2. **Scope phrasing.** We expose a single `mcp` scope today — happy
   to split into `read_time` / `write_time` if directory listings
   prefer per-scope consent UIs.
3. **Skill discoverability.** The `closedrings:track-time` skill is
   shipped over `serverInfo.instructions` on every MCP `initialize`
   handshake AND mirrored in `skill/SKILL.md` in this repo. Let us
   know if the directory has a preferred way to surface skills to
   reviewers / users.
