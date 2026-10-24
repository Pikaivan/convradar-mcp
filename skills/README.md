# ConvRadar skills

These are the public source files for workflows that use ConvRadar's hosted MCP server. Each skill lives in its own directory so future workflows can be added without changing existing skill paths.

| Skill | Use it for |
| --- | --- |
| [SaaS Page Audit](saas-page-audit/SKILL.md) | Audit one SaaS pricing, product, hosting plan, subscription, or B2B service page; combine public page evidence with the connected site's GA4 data when available; produce one HTML report. |

The SaaS Page Audit folder contains its instructions, checks, data rules, writing rules, HTML report template, and ChatGPT connection metadata. The files are the same release snapshot served by the ConvRadar MCP endpoint for plugin skill import. Reading them on GitHub does not install a skill in ChatGPT; plugin versions import a snapshot when the developer runs **Scan Tools**.

The contents of this repository, including these skill files, are available under the [MIT License](../LICENSE). ConvRadar's hosted service remains proprietary.
