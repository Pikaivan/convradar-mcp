# Installing ConvRadar in Cline

ConvRadar is a **hosted, remote** MCP server — there is nothing to clone, build, or run
locally. This repository is a manifest only. Installation is adding one config block;
authentication happens in the browser on first use.

## Prerequisites

- **Node.js 18+** — provides `npx`, which launches the `mcp-remote` bridge. Verify with `node --version`.
- A **Google Analytics 4** property you have access to.
- A **ConvRadar account** — free during the open beta (no card). It's created automatically
  when you sign in with Google during the OAuth step below.

## Step 1 — Add the server to Cline

Add this entry to Cline's MCP settings file (`cline_mcp_settings.json`), under `mcpServers`:

```json
{
  "mcpServers": {
    "convradar": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.convradar.com/mcp"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

No API keys or environment variables are required — `mcp-remote` handles the OAuth handshake.

## Step 2 — Authenticate (one time)

On the first ConvRadar tool call, `mcp-remote` opens a browser window for **OAuth 2.1** login.
Sign in with the Google account that owns your GA4 property and approve **read-only** access
(scopes: `read:metrics`, `write:hypotheses`). The token is cached locally, so later sessions
don't prompt again.

If no browser opens automatically, copy the authorization URL that `mcp-remote` prints in the
Cline MCP server log and open it manually.

## Step 3 — Verify the install

Ask Cline:

> Run a full audit of my ConvRadar account.

A working install returns real GA4 numbers (sessions, funnel steps, conversion rates). You can
also try *"Where's my biggest funnel drop?"*.

## Notes for the installing agent

- This is a **remote** server. Do **not** `git clone`, `npm install`, or build anything from
  this repo — there is no local binary.
- Do **not** add any token or per-user URL to the config. The public endpoint is
  `https://mcp.convradar.com/mcp`; OAuth is automatic via `mcp-remote`.
- Network access to `mcp.convradar.com` and `registry.npmjs.org` (for `npx`) is required.
- 32 GA4 conversion-diagnostic tools are exposed (read-only analytics plus opt-in user-state tools) (full catalogue in [`README.md`](./README.md)).
  The OAuth grant is read-only — ConvRadar can never modify your analytics data.

## Troubleshooting

- **`npx: command not found`** → install Node.js 18+.
- **401 / authentication errors** → re-run the tool to retrigger the browser login, or clear the
  `mcp-remote` auth cache at `~/.mcp-auth/` and try again.
- **Want to try before connecting GA4?** → https://convradar.com/chat (3 free messages, no signup).
