# Release process

## Versioning

Semantic Versioning, with one project-specific clarification on
what counts as a breaking change:

- **Patch (0.1.x → 0.1.y)** — bug fixes, copy tweaks, internal
  refactors. No server requirement change. Safe to publish without
  coordination.
- **Minor (0.x.0 → 0.y.0)** — new client supported, new commands,
  additive config keys. May bump the minimum server version — if
  it does, that's called out at the top of the release notes and
  reflected in the compatibility table in `CHANGELOG.md`.
- **Major (x.0.0 → y.0.0)** — breaking changes to command surface
  (`connect` semantics change in a non-additive way), required Node
  version bump, or required server version bump that's
  incompatible with versions still in the wild. Rare; needs a
  written migration note.

## Cutting a release

1. Land all the work for the release on `main` (PRs against `main`
   reviewed and merged).
2. Update `CHANGELOG.md` — move the `[Unreleased]` section to the
   new version's heading, date it, refresh the compatibility row.
3. Update `package.json#version` to match.
4. Commit: `chore(release): vX.Y.Z`.
5. Tag: `git tag -a vX.Y.Z -m "vX.Y.Z" && git push --tags`.
6. GitHub Action picks up the tag, runs tests + lint, publishes to
   npm if everything's green. (CI workflow lives in
   `.github/workflows/release.yml` — to be added in the first dev
   session.)
7. GitHub Release auto-drafts from the tag; paste the relevant
   CHANGELOG section into the body, attach any precompiled
   artifacts.
8. Bump the docs in the main app at
   [closedrings.sh/docs/mcp/overview](https://closedrings.sh/docs/mcp/overview)
   to mention the new version if anything user-visible changed.

## Compatibility with the main app

The main Rails app at `closedrings.sh` is the server side. The
plugin announces its **minimum supported server version** at the top
of `CHANGELOG.md` and in `package.json#engines.closedrings` (custom
field, read by `closedrings-mcp doctor`).

A release that requires a server-side change (e.g. a new tool, a
new auth shape, a different device-flow scope) goes through a
coordinated rollout:

1. The server change lands and ships to `closedrings.sh` first.
2. The server version on `serverInfo` in the MCP handshake reflects
   the new capability.
3. The plugin release goes out **after** the server is live in
   production. Otherwise users would download a plugin that demands
   features the server doesn't have yet.

Server versions are tracked separately (they roll with the main
app's deploy cadence). The plugin doesn't need to pin a max server
version — it needs to be tolerant of newer servers, which the MCP
protocol's capability negotiation handles.

## What goes in a release announcement

- One-line summary at the top
- New features (one bullet each)
- Fixes
- Compatibility note if the minimum server version bumped
- Migration steps for users on the prior version (usually: none;
  re-run `closedrings-mcp connect`)

Keep it terse. Users skim release notes; the changelog is the
reference for the curious.

## Yanking a release

If a release ships with a critical bug:

1. `npm deprecate closedrings-mcp@X.Y.Z "<reason — point to issue>"`
2. Cut a patch release with the fix.
3. Mark the broken version in `CHANGELOG.md` under a
   `### Yanked` heading inside that version's section.

Don't unpublish — npm policy is to prefer deprecate over unpublish
unless the version has security implications or was published in
error within 72 hours.
