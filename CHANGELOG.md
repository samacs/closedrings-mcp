# Changelog

All notable changes to `closedrings-mcp` will be documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Direction (not yet released)

This repo holds marketplace artifacts — the skill, listing
manifests, and branding assets — for connecting MCP-capable
agents to Closed Rings. **No code; no npm publish.**

The actual auth flow is implemented in the main app
(`closedrings.sh`) as MCP-spec OAuth 2.0 + Dynamic Client
Registration + PKCE. See `docs/architecture.md`.

### Planned for v0.1.0

- `manifests/claude-desktop/connector.json` — for Claude Desktop's
  connector directory submission.
- `manifests/claude-code/plugin.json` + repo-rooted `SKILL.md` so
  `claude plugin install github.com/samacs/closedrings-mcp` works.
- `manifests/cursor/mcp.json` — for the Cursor MCP marketplace.
- Branding assets (logo, screenshots) referenced by the
  manifests.
- Minimum main-app version: the version that lands MCP OAuth
  endpoints (tracked as Phase F).

### Planned for v0.2.0+

- Refresh skill copy based on usage patterns from the first
  cohort of marketplace installs.
- Additional marketplace listings as the ecosystem grows (Zed,
  Continue, etc.).

---

## Compatibility

| Plugin    | Requires main app version | MCP protocol revision |
|-----------|---------------------------|------------------------|
| 0.1.0     | TBD (Phase F + OAuth)     | 2025-06-18             |
