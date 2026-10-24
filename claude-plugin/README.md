# ConvRadar for Claude

ConvRadar bundles two conversion audit skills with a remote ConvRadar MCP connector. Add this plugin in Claude to make both skills available in chat, Cowork, and Claude Code. Connect the ConvRadar connector in the plugin's Connectors tab and sign in to the account associated with your Google Analytics 4 property before requesting an audit that needs private metrics.

- **SaaS Page Audit** reviews one SaaS product, pricing, or subscription page and its next step. It produces an HTML report grounded in the page, buyer questions, competitor pages, and connected GA4 data when available.
- **E-commerce and Lead Generation Audit** reviews a whole online store or service lead-generation site. It checks the reliability of the conversion measurement before making recommendations and keeps the two business models' rules separate.

The plugin connects to `https://mcp.convradar.com/mcp`. The server reads the connected account's GA4 data and exposes diagnostic tools; some calls count against the account's ConvRadar allowance. The skills also read public web pages and sources for page evidence and comparisons. They create HTML reports in the user's workspace. They do not require credentials in the plugin files and do not call ConvRadar write tools. See [ConvRadar's MCP documentation](https://convradar.com/docs/mcp) for account setup and the [public source repository](https://github.com/Pikaivan/convradar-mcp) for the complete skill text.

Example requests: “Audit my SaaS pricing page at https://example.com/pricing” or “Run a full conversion audit of my store.” Claude selects the relevant skill from its description. A full audit may use several data calls; ask for a page-only audit if you do not want ConvRadar calls.
