# ConvRadar skills

These are the public source files for workflows that use ConvRadar's hosted MCP server. Each skill lives in its own directory.

| Skill | Use it for |
| --- | --- |
| [SaaS Page Audit](saas-page-audit/SKILL.md) | Audit one SaaS pricing, product, hosting plan, subscription, or B2B service page; combine public page evidence with the connected site's GA4 data when available; produce one HTML report. |
| [E-commerce & Lead Gen Audit](convradar-ecom-leadgen-audit/SKILL.md) | Audit a whole online store or service lead-generation site; verify purchases or enquiries, analyze funnel losses, and produce one HTML report or a tracking diagnostic when conversions are unreliable. Excludes SaaS. |

Each folder contains instructions, references, an HTML report template, and ChatGPT connection metadata. The files are the same release snapshots served by the ConvRadar MCP endpoint for plugin skill import. Reading them on GitHub does not install a skill in ChatGPT; plugin versions import a snapshot when the developer runs **Scan Tools**.

The contents of this repository, including these skill files, are available under the [MIT License](../LICENSE). ConvRadar's hosted service remains proprietary.
