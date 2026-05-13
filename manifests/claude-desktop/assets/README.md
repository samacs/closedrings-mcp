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
| `wordmark.svg` | Horizontal logo + wordmark lockup, exported from the existing dashboard wordmark in `closedrings.sh:app/views/marketing/_nav.html.erb`. |
| `screenshots/01-dashboard.png` (2998×1710) | Dashboard `/` view with active rings + recent time blocks. |
| `screenshots/02-claude-connect.png` (2980×1704) | Claude OAuth consent screen at `closedrings.sh/oauth/authorize`. |
| `screenshots/03-claude-log.png` (2232×876) | Claude chat pane: agent calling `log_time_block` and confirming the result. |
| `screenshots/04-audit-log.png` (2812×696) | `/admin/audit` row strip showing the OAuth audit trail. |

> Note: `03` and `04` are wider than the typical 16:10 directory
> card aspect. They render fine in lightboxes; recrop to 16:10 if
> the directory listing previews crop badly.

## Capture workflow (for refreshes)

When the dashboard chrome, consent UI, or audit-log layout changes
in a way that makes these stale:

1. Use a fresh demo account with seed data so captures are
   reproducible — don't use your actual usage history.
2. macOS dark-mode-or-light follows the OS; the dashboard
   auto-themes, so set the OS to whichever matches your other
   listing screenshots.
3. Crop to the relevant pane — directory listings reject
   screenshots with browser chrome, OS menus, or sensitive data
   in the corners.
4. Run each PNG through `pngquant --quality 70-90` to keep file
   size under 500 KB; some directory upload forms cap at 1 MB.
5. Re-export `wordmark.svg` from Figma at the same canvas as the
   prior version so the directory card layout doesn't shift.
