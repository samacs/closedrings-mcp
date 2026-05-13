# manifests/

Per-marketplace listing manifests. Each subdirectory holds the
files one specific marketplace expects, in that marketplace's
format. Empty for now; filled in by the dev session once the
server-side OAuth flow lands.

## Planned layout

```
manifests/
├── claude-desktop/
│   ├── connector.json     # Claude Desktop's connector manifest
│   ├── icon.png           # 256×256 brand mark
│   └── README.md          # submission notes (review process, contacts)
├── claude-code/
│   ├── plugin.json        # Claude Code plugin manifest
│   └── README.md          # `claude plugin install` URL + verification steps
├── cursor/
│   ├── mcp.json           # Cursor MCP marketplace listing
│   └── README.md          # submission notes
└── ... (additional marketplaces as the ecosystem grows)
```

## What each manifest must point at

All listings ultimately reference:

1. The hosted MCP server URL: `https://api.closedrings.sh/mcp`
2. The OAuth metadata location:
   `https://api.closedrings.sh/.well-known/oauth-protected-resource`
3. The branding: logo, short description, long description,
   permitted scopes
4. The skill file in this repo (different marketplaces consume
   it differently — Claude Code reads it from the plugin
   bundle; others may not consume it at all and rely on the
   server's `serverInfo.instructions` instead)

## Sourcing the current format

Each marketplace's format changes as the ecosystem matures. Don't
freeze a manifest schema from memory — read the current
submission docs when adding or updating a listing:

- Claude Desktop connectors: <https://docs.claude.com/en/docs/claude-code/mcp> (or the current connector directory submission docs)
- Claude Code plugins: <https://docs.claude.com/en/docs/claude-code/plugins>
- Cursor: <https://docs.cursor.com/> (search for MCP / marketplace)

URLs above are starting points; the actual submission docs may
have moved by the time you read this.
