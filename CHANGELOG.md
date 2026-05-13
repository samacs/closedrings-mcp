# Changelog

All notable changes to `closedrings-mcp` will be documented here. Format:
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning:
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned for v0.1.0

- `closedrings-mcp connect` — mints an agent token via device flow,
  auto-detects installed MCP clients (Claude Desktop, Claude Code,
  Cursor), writes the server entry to each.
- `closedrings-mcp status` — shows which clients are currently
  configured to talk to Closed Rings.
- `closedrings-mcp disconnect [--client <name>]` — revokes the token
  on the server and removes the entry from the client config.
- Companion skill at `skill/SKILL.md` for clients that read a
  rules / custom-instructions file.

### Planned for v0.2.0

- Marketplace listings: Claude Code plugin manifest, Claude Desktop
  connector manifest, Cursor MCP marketplace entry.
- `closedrings-mcp doctor` — diagnoses common setup issues (TLS
  trust, token expiry, client picked-up state).

---

## Compatibility

Each plugin release is tested against a specific minimum version of
the Closed Rings server. See `docs/release-process.md` for the
compatibility matrix.

| Plugin    | Requires server |
|-----------|-----------------|
| 0.1.0     | TBD             |
