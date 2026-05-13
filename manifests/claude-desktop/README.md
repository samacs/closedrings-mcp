# manifests/claude-desktop/

The package we hand to Anthropic for the Closed Rings listing in the
[Claude connector directory](https://claude.ai/directory). The directory
is curated — submission goes through a Google Form Anthropic links from
[claude.com/partners/mcp](https://claude.com/partners/mcp), not a
self-serve registry — so this directory is **the source of truth we
copy-paste from when we fill the form**, not something Claude Desktop
fetches at runtime.

## Files

| File | What it is |
|---|---|
| [`connector.json`](connector.json) | One-file manifest of every field the form asks for: identity, branding, publisher, server, auth, scopes, tools, security. Maintained as a JSON document so it diffs cleanly. |
| [`submission.md`](submission.md) | The cover-letter for the review team. What it does, why it belongs, repro steps, auth detail, security posture, open questions. |
| [`assets/`](assets/) | Branding: SVG master, four pre-rendered icon sizes, wordmark, four screenshots of the connect flow. Submitted with the form. |

## How to submit

1. **Verify production.** All four endpoints below must respond
   correctly from outside our network:

   ```sh
   curl -fsSL https://api.closedrings.sh/.well-known/oauth-protected-resource | jq .
   curl -fsSL https://closedrings.sh/.well-known/oauth-authorization-server  | jq .
   curl -fsS  -i -X POST https://api.closedrings.sh/mcp \
        -H 'Content-Type: application/json' \
        -d '{"jsonrpc":"2.0","id":1,"method":"ping"}' \
        | grep -i www-authenticate
   curl -fsSL -X POST https://closedrings.sh/oauth/register \
        -H 'Content-Type: application/json' \
        -d '{"client_name":"submission-smoke-test","redirect_uris":["http://127.0.0.1:8765/cb"]}'
   ```

   First three should return JSON / a 401-with-WWW-Authenticate. The
   fourth should return a `client_id`; manually delete that test row
   from `oauth_clients` afterwards (it's harmless if you forget).

2. **Smoke-test the actual flow.** From a fresh Claude Desktop install,
   add `https://api.closedrings.sh/mcp` as a custom connector and
   verify the consent screen renders with the right copy. Take fresh
   screenshots if the consent UI has changed since the assets in this
   directory were captured.

3. **Open the form**:
   <https://docs.google.com/forms/d/e/1FAIpQLSeafJF2NDI7oYx1r8o0ycivCSVLNq92Mpc1FPxMKSw1CzDkqA/viewform>

4. **Fill from `connector.json`.** Every field the form asks for has a
   matching key in the manifest — search there first before re-typing.

5. **Attach `submission.md`** as the long-form description / cover
   letter. Upload `assets/icon-1024.png` (or whichever size the form
   accepts) and the four screenshots in order.

6. **Tag the release.** After the form is submitted, tag the next
   patch version of this repo (`git tag v0.1.0 && git push --tags`)
   and update `CHANGELOG.md` with the submission date so we can
   correlate any back-and-forth with the review team to a specific
   manifest version.

## When to update + resubmit

| Change | Resubmit? |
|---|---|
| Server URL changes | **Yes** (auth flow breaks; users can't connect) |
| New OAuth scope | **Yes** (consent UI text changes) |
| Tool surface changes (add / remove a tool) | Yes if the connector description references it |
| Skill copy / playbook tweak | No (the server ships the canonical copy via `serverInfo.instructions`) |
| Branding refresh | Yes (replace assets in this directory + resubmit) |
| Publisher contact info change | Yes (so they can reach you) |
| Production redeploy with no behavior change | No |

Material changes to the manifest should bump `connector.json`'s
`schema_version` (semver) so we can correlate "what's in production"
with "what was last submitted".

## Notes on what's not here

- **No Claude Desktop-specific runtime config.** Custom connectors are
  configured through the app's UI, not via a manifest the user
  installs. The directory listing is purely metadata + branding.
- **No client_id pre-registration.** Dynamic Client Registration
  handles all of that on first connect — Anthropic doesn't pre-register
  Claude Desktop with each connector publisher.
- **No webhooks.** Closed Rings doesn't push to the agent; the agent
  polls or subscribes to the resources we expose.
