# Changelog

All notable changes to `closedrings-mcp` will be documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added — 2026-05-13

- **Claude Code plugin marketplace.** This repo now IS a Claude
  Code plugin marketplace; install with
  `claude plugin marketplace add samacs/closedrings-mcp` then
  `claude plugin install closedrings@closedrings`.
  - `.claude-plugin/marketplace.json` — top-level catalog.
  - `plugins/closedrings/.claude-plugin/plugin.json` — manifest
    declaring the remote HTTP MCP server
    (`https://api.closedrings.sh/mcp`).
  - `plugins/closedrings/skills/closedrings-track-time/SKILL.md` —
    auto-invoked playbook (frontmatter shaped for Claude Code's
    skill loader, distinct from the docs-page-shaped
    `skill/SKILL.md` at the repo root).
  - `manifests/claude-code/README.md` — install + iteration
    instructions, verification commands.

### Status of remaining v0.1.0 work

- [ ] Claude Desktop submission package (in flight on
  [`feature/claude-desktop-listing`](https://github.com/samacs/closedrings-mcp/pull/1))
- [x] Claude Code plugin marketplace (this entry)
- [ ] `manifests/cursor/` for the Cursor MCP marketplace
- Minimum main-app version: the version that lands the Phase F
  OAuth endpoints — currently `feature/mcp-oauth` in the main repo
  ([PR #98](https://github.com/samacs/closedrings.sh/pull/98)).

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
