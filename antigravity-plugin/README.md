# ConvRadar for Antigravity

ConvRadar reads a connected Google Analytics 4 property and answers where a store, lead-generation site or SaaS page loses conversions, and which segments are behind the drop. The plugin adds the remote ConvRadar MCP server at `https://mcp.convradar.com/mcp` and two audit skills to Antigravity.

## Install

From a clone of this repository:

```bash
git clone https://github.com/Pikaivan/convradar-mcp
agy plugin install ./convradar-mcp/antigravity-plugin
```

The same plugin installs in the Antigravity App and IDE. To connect only the MCP server without the skills, add it to `~/.gemini/config/mcp_config.json`:

```json
{
  "mcpServers": {
    "convradar": {
      "serverUrl": "https://mcp.convradar.com/mcp"
    }
  }
}
```

## Sign in

Sign in from the MCP servers panel. In the agy CLI open `/mcp`, select `convradar_convradar` and choose Authenticate. In the App or IDE open Customizations, then Installed MCP Servers, and choose Authenticate. The browser opens ConvRadar: continue with Google, connect your Google Analytics 4 property if the account is new, and approve access. The App, the IDE and the CLI keep separate sign-ins, so each asks once. If `convradar_convradar` is missing from `/mcp`, agy started before you signed in to Antigravity: quit it and start it again. The server registers Antigravity as an OAuth client by itself (dynamic client registration with PKCE), so no keys or tokens go into the config.

## What it answers

Ask where the funnel loses the most visitors and ConvRadar finds the biggest leak and the segments behind it. It compares mobile with desktop or any two segments with a significance test, shows product sales from views to purchases, measures bot, spam and internal traffic in GA4 and how much it moves the conversion rate, flags sudden spikes and drops in sessions, revenue or conversion rate, checks page speed and benchmarks the site against its industry. It also builds a GA4 tracking plan that lists which events to add or fix, and keeps a diary of shipped changes with a before and after reading.

Two skills ship with the plugin. `saas-page-audit` reviews one SaaS product, pricing or plan page and its next step. `convradar-ecom-leadgen-audit` audits a whole online store or service lead-generation site and checks that conversion tracking is reliable before it recommends anything. Both write one HTML report. A rule in `rules/convradar.md` has the agent start with the account check and, when ConvRadar is not signed in, point you to Authenticate instead of changing config files.

## Plans and data

Some calls count against the account's ConvRadar allowance; the Free plan and paid plans are listed at [convradar.com/pricing](https://convradar.com/pricing). Privacy policy: [convradar.com/privacy](https://convradar.com/privacy). Full tool reference: [convradar.com/docs/mcp](https://convradar.com/docs/mcp). Support: hello@convradar.com.
