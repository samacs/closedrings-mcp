# Branding assets — Claude Desktop submission

Files in this directory are referenced by `../connector.json` and
attached to the Anthropic submission form. License: MIT (same as the
rest of this repo).

## What's here

| File | Source / how to regenerate |
|---|---|
| `logo.svg` | Master vector. Copied from `closedrings.sh:app/assets/images/closedrings.svg`. The wordmark + ring composition; works on any background. |
| `icon-1024.png` | 1024×1024 master raster. Copied from `closedrings.sh:app/assets/images/closedrings.png`. |
| `icon-512.png`, `icon-256.png`, `icon-128.png` | Downscales of `icon-1024.png`, generated with macOS `sips`: `for s in 512 256 128; do sips -Z $s icon-1024.png --out icon-$s.png; done`. Re-run on every refresh of the master so they stay in lockstep. |

## What's still missing (and why this submission isn't ready to send yet)

These need to exist before we file the form. Keeping them as named
slots so the gap is obvious; create-and-commit, don't move the slots.

- [ ] **`wordmark.svg`** — horizontal logo + wordmark lockup for the
      directory listing card. ~3:1 aspect ratio. Source from
      `app/views/marketing/_nav.html.erb` (the existing dashboard
      wordmark) by exporting it in Figma at 600×200 with no padding.
- [ ] **`screenshots/01-dashboard.png`** — the dashboard's `/` view
      with at least one project, three time blocks logged, and a
      visible ring near closure. 1600×1000 PNG or 1280×800; matches
      what other directory listings ship. Capture in light mode for
      consistency with directory norms.
- [ ] **`screenshots/02-claude-connect.png`** — Claude Desktop's
      "Add custom connector" → consent screen at
      `https://closedrings.sh/oauth/authorize?...`. Same dimensions.
      Should clearly show the client name, the requested scope, and
      the Approve / Deny buttons.
- [ ] **`screenshots/03-claude-log.png`** — Claude Desktop's chat
      pane mid-conversation, showing the agent calling
      `log_time_block` with a confirmation line and the resulting
      block ID returned. Same dimensions.
- [ ] **`screenshots/04-audit-log.png`** — `/admin/audit` filtered to
      the three OAuth audit kinds (`oauth.client.registered`,
      `oauth.consent.granted`, `agent_token.created`) showing the
      end-to-end trail of one connect. Same dimensions.

## Capture workflow (when you sit down to do the screenshots)

1. Use a fresh demo account with seed data so screenshots are
   reproducible — don't use your actual usage history.
2. macOS dark-mode-or-light follows the OS; the dashboard auto-themes,
   so set the OS to whichever matches your other listing screenshots.
3. Crop to the relevant pane — directory listings reject screenshots
   with browser chrome, OS menus, or sensitive data in the corners.
4. Run each PNG through `pngquant --quality 70-90` to keep file size
   under 500 KB; the form caps uploads.
