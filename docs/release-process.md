# Release process

This repo doesn't publish to npm. Releases here are
**marketplace submissions** + tagged versions of the skill and
manifests, so a given Closed Rings server can be paired with a
known-good set of artifacts.

## Versioning

Semantic Versioning, interpreted for an artifacts-only repo:

- **Patch (0.1.x)** — copy tweaks in the skill, fixes to a
  manifest, branding refresh. No marketplace re-submission
  required if the change is below their review threshold.
- **Minor (0.x.0)** — new marketplace supported, new skill
  capability, additive manifest fields. May require re-submission
  to one or more marketplaces.
- **Major (x.0.0)** — breaking changes that require all listings
  to be updated in lockstep (e.g. a server URL change, an OAuth
  scope rename). Rare.

The version is also the version users see in the marketplace
listings, so bumps should be deliberate.

## Cutting a release

1. Land all the work on `main` via PRs.
2. Update `CHANGELOG.md` — move `[Unreleased]` to the new version,
   date it, note any marketplace re-submission needed.
3. Tag: `git tag -a vX.Y.Z -m "vX.Y.Z" && git push --tags`.
4. GitHub Release auto-drafts from the tag; paste the relevant
   CHANGELOG section into the body.
5. If any marketplace needs re-submission, file each one and
   track the review state in the GitHub Release notes (link to
   each marketplace's submission page).

## Compatibility with the main app

Each release announces the **minimum supported main-app version**
at the top of `CHANGELOG.md`. That version is what the
marketplace listings claim. If the main app's MCP server URL,
OAuth metadata shape, or supported tool surface changes in a way
that breaks a previously-shipped manifest, the next release here
needs to bump the minimum.

A common case to watch for: the OAuth metadata may change between
spec revisions. If `closedrings.sh` adopts a newer MCP spec
revision, any manifests that hard-code old paths need updates.

## What goes in a release announcement

- One-line summary
- Which marketplaces are affected (with submission links)
- Skill changes (bullet each behavioural rule that moved)
- Compatibility: minimum main-app version + the MCP protocol
  version it targets
- Migration steps for existing users (usually: none — the agent
  picks up the new skill on its next handshake)

## Yanking a release

If a manifest has a critical bug (e.g. a wrong URL in a connector
listing):

1. Delist the affected manifest from the marketplace (each has
   its own process — varies from "edit the listing" to "open a
   support ticket").
2. Tag a patch release with the fix.
3. Re-submit to the marketplace.
4. Mark the broken version under a `### Yanked` heading inside
   that version's section of `CHANGELOG.md`.

Don't delete the tag — leave the audit trail intact.
