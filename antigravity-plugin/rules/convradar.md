---
trigger: model_decision
description: Use when the user asks about their website, Google Analytics 4, conversions, funnel, traffic, a tracking plan or ConvRadar, or when a convradar_convradar tool is missing or answers Unauthorized.
---

ConvRadar answers from the user's connected Google Analytics 4 property through the `convradar_convradar` MCP server.

- Start with `cr_get_account_info`. It names the connected site, the business model and the dates with data. If it reports an import in progress, tell the user the progress and offer the checks it lists in `available_now`.
- When the user does not know where to start, ask ConvRadar: "What's the biggest problem on my site right now, and what should I work on first?"
- If the `convradar_convradar` tools are missing or a call answers Unauthorized, ConvRadar is not signed in on this surface. Stop and tell the user how to sign in. In the agy CLI: open `/mcp`, select `convradar_convradar`, move to Authenticate with the right arrow and press Enter. In the App or IDE: open Customizations, then Installed MCP Servers, and choose Authenticate. A browser tab opens ConvRadar; the user approves access and asks again. The App, the IDE and the CLI each sign in on their own.
- If `convradar_convradar` is not listed in `/mcp` at all, agy started before the user signed in to Antigravity and did not load plugin servers. Tell the user to quit agy, start it again and then Authenticate.
- Never edit `mcp_config.json`, never call mcp.convradar.com with curl, Python or other scripts, and never install or run `mcp-remote` to get around a missing sign-in. Those changes replace the plugin's own sign-in and break it.
