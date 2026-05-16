# Changelog

All notable changes to `closedrings-mcp` will be documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## 0.2.0 - 2026-04-16

### Changed — Skill copy aligned with task/project `description` field

Paired with closedrings.sh#136 (the platform release that adds a
free-form `description` field to projects and tasks). The MCP tools
themselves live in that repo and were updated alongside the feature;
this bundle just updates the agent-facing playbook so the workflow
stays consistent.

- `skill/SKILL.md` (canonical) and
  `plugins/closedrings/skills/closedrings-track-time/SKILL.md`
  (plugin-marketplace copy) both gain a `description?` parameter on
  the `create_task` row, with a nudge to capture acceptance criteria
  / links / gotchas when the user dictates them alongside the task
  name.
- New behavioral rule **3a**: read descriptions returned by
  `list_projects` and `list_tasks` before disambiguating an ambiguous
  reference. The right context is often already in the payload.
- Behavioral rule 3 ("don't invent projects") extended to reinforce
  that `create_task` can carry a description on the way in.

### Changed — Skill copy aligned with `create_task` + `git_context.sha`

Paired with the main-app feature branch
`feature/mcp-create-task-and-git-context` that adds `create_task`
to the MCP surface and accepts `sha` on the `git_context` shape
of every time-block write tool.

- `skill/SKILL.md` (canonical) and
  `plugins/closedrings/skills/closedrings-track-time/SKILL.md`
  (plugin-marketplace copy) both gain a `create_task` row in the
  write-tools table, a trigger-phrase entry ("File a ticket
  for…"), and `git_context?` documented as an optional parameter
  on `start_time_block` / `log_time_block` / `update_time_block`.
- Behavioral rule 3 narrowed: "don't invent **projects**" — tasks
  are now agent-creatable under existing projects via
  `create_task`; projects remain dashboard-only.
- `git_context` field shape gains `sha` alongside `branch`,
  `repo`, `commits`, `range`. The CLI populates these
  automatically; agents may pass them when their client knows
  the repo state.

## [0.1.2] — 2026-05-13

### Changed — `docs/architecture.md` aligned with shipped OAuth

The architecture doc said OAuth was "**not yet implemented**,
tracked as Phase F" and dedicated a section to telling a future
dev session what to build. OAuth has shipped on the main app
([closedrings.sh#98](https://github.com/samacs/closedrings.sh/pull/98));
this doc and the repo's README now tell the same story.

- "Where the work actually happens" table: OAuth row points at
  the merged PR with real file paths (`config/routes.rb`,
  `app/controllers/oauth/`, the `OauthClient` /
  `OauthAuthorization` models, `ApiToken` reuse with
  `kind: agent`).
- "What the dev session must verify against the spec" →
  "How OAuth landed". Same numbered list rewritten as a tour of
  what shipped, including the `POST /oauth/authorize/deny`
  endpoint that wasn't in the original plan and the consent UI
  under `app/views/oauth/authorizations/`.
- Added RFC 8707 (Resource Indicators) to the spec references.
  The `resource` parameter is explicitly implemented in
  `app/controllers/well_known/oauth_protected_resource_controller.rb`
  for audience binding.
- Cross-references the Turbo-bypass follow-on fix
  ([closedrings.sh#99](https://github.com/samacs/closedrings.sh/pull/99)).

No code, manifest, or skill changes — `connector.json`, both
plugin manifests, and every `SKILL.md` are byte-identical to
v0.1.1. No marketplace re-submission required.

---

## [0.1.1] — 2026-05-13

### Added — Claude Desktop submission assets

Fills the slots from the v0.1.0 `manifests/claude-desktop/assets/`
README so the package is now actually submission-ready:

- `wordmark.svg` — horizontal logo lockup
- `screenshots/01-dashboard.png` — dashboard with active rings
- `screenshots/02-claude-connect.png` — OAuth consent screen
- `screenshots/03-claude-log.png` — agent calling `log_time_block`
- `screenshots/04-audit-log.png` — OAuth audit trail row strip

Also: `.gitignore` now drops `.DS_Store`; `assets/README.md`
swaps the "still missing" checklist for a concise capture-refresh
workflow.

No code or manifest schema changes — `connector.json` and the
plugin manifests are byte-identical to v0.1.0.

---

## [0.1.0] — 2026-05-13

First release. Two marketplace artifacts ship together:

### Added — Claude Code plugin marketplace

This repo IS a Claude Code plugin marketplace. Install with:

```sh
claude plugin marketplace add samacs/closedrings-mcp
claude plugin install closedrings@closedrings
```

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

### Added — Claude Desktop submission package

Ready to submit to the Anthropic Connector Directory once the
production deploy + screenshot captures land. Stored under
`manifests/claude-desktop/` for version-controlled iteration:

- `connector.json` — single-source-of-truth listing data
  (identity, branding, publisher, server, auth, scopes, tools,
  resources, security posture, regional availability).
- `submission.md` — cover letter for the Anthropic review team
  with reproduction steps, auth detail, and security posture.
- `README.md` — submission checklist + when-to-resubmit table.
- `assets/` — SVG master, PNG icons at 1024 / 512 / 256 / 128 px.
  Screenshots (4) and the wordmark lockup are slotted with a
  capture workflow but not yet captured.

### Status of remaining v0.1.x follow-ups

- [ ] Capture the 4 Claude Desktop screenshots + wordmark, then
  submit the form
- [ ] `manifests/cursor/` for the Cursor MCP marketplace
- Minimum main-app version: the build that lands MCP OAuth
  endpoints (Phase F of the MCP integration epic, merged via
  [`closedrings.sh#91`](https://github.com/samacs/closedrings.sh/pull/91)).

### Planned for v0.2.0+

- Refresh skill copy based on usage patterns from the first
  cohort of marketplace installs.
- Additional marketplace listings as the ecosystem grows (Zed,
  Continue, etc.).

---

## Compatibility

| Plugin | Requires main app version             | MCP protocol revision |
|--------|---------------------------------------|------------------------|
| 0.1.2  | Build that includes Phase F (OAuth)   | 2025-06-18             |
| 0.1.1  | Build that includes Phase F (OAuth)   | 2025-06-18             |
| 0.1.0  | Build that includes Phase F (OAuth)   | 2025-06-18             |
