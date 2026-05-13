# manifests/claude-code/

Claude Code's plugin marketplace is **fully self-serve** — there's
no review process, no submission form. This entire repository IS
the marketplace; users install the plugin with one command. So,
unlike `manifests/claude-desktop/`, this directory holds notes
rather than upload-ready artifacts.

## What ships the plugin

The Claude Code plugin lives at the **top of this repository**, not
under `manifests/`, because that's where Claude Code's marketplace
loader expects it:

```
closedrings-mcp/
├── .claude-plugin/
│   └── marketplace.json                 ← the catalog Claude Code reads
└── plugins/
    └── closedrings/
        ├── .claude-plugin/
        │   └── plugin.json              ← the plugin manifest
        └── skills/
            └── closedrings-track-time/
                └── SKILL.md             ← invoked automatically by Claude
```

`marketplace.json` lists one plugin (`closedrings`) sourced from a
relative path. `plugin.json` declares the remote MCP server
(`https://api.closedrings.sh/mcp`) and points at the bundled skill.
Claude Code starts the MCP server when the plugin is enabled and
auto-discovers `skills/*/SKILL.md`.

## Installing

```sh
# Add this repo as a marketplace
claude plugin marketplace add samacs/closedrings-mcp

# Install the plugin from it
claude plugin install closedrings@closedrings
```

After install, the first MCP tool call triggers the OAuth flow:
Claude Code reads the `WWW-Authenticate` header from a 401, fetches
the discovery documents, registers itself via DCR, opens the
browser for consent, and exchanges the code for a token. The skill
loads automatically — Claude will invoke it whenever the user
mentions time tracking, asks for a standup, or describes past work
they want logged.

## Verifying the install

```sh
# Confirm the marketplace is registered
claude plugin marketplace list

# Confirm the plugin loaded
claude plugin list

# Smoke-test the MCP server is wired
claude mcp list   # should show "closedrings (http) - https://api.closedrings.sh/mcp"
```

In a Claude Code session, ask "What projects do I have on Closed
Rings?" — the agent should pull the skill, call `list_projects`,
and (on first call) open the consent screen in your browser.

## Iteration workflow

Plugin source lives at `plugins/closedrings/`. To iterate locally:

```sh
# Add THIS local checkout as a marketplace (overrides the GitHub one)
claude plugin marketplace add ~/Developer/closedrings-mcp

# Reinstall against the local copy
claude plugin install closedrings@closedrings
```

Edit `plugins/closedrings/skills/closedrings-track-time/SKILL.md`
or `plugins/closedrings/.claude-plugin/plugin.json`, then re-run
`/plugin marketplace update` from a Claude Code session to pull
the change.

When the change is good, bump `version` in **both**
`marketplace.json` (the entry's version) and `plugin.json` (the
manifest's version) so users actually receive the update. Per the
docs, omitting `version` makes every commit count as a new
version, but explicit semver is friendlier for the changelog.

## Cross-references

- Source-of-truth playbook: `app/api/mcp/playbook.rb` in the
  [`samacs/closedrings.sh`](https://github.com/samacs/closedrings.sh) repo
- Standalone skill (for clients without plugin support): `skill/SKILL.md`
  at the root of this repo
- Plugin-shipped skill (this is what Claude Code reads):
  [`plugins/closedrings/skills/closedrings-track-time/SKILL.md`](../../plugins/closedrings/skills/closedrings-track-time/SKILL.md)

When the canonical playbook changes, sync **all three** copies in
the same PR.
