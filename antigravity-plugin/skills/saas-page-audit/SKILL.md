---
name: saas-page-audit
description: Use when a user gives a SaaS product, pricing, hosting plan, subscription or B2B service page URL and asks for a conversion audit, why it does not convert, or specific fixes (including "аудит страницы тарифов"). Produces one HTML report using the page, its signup or checkout, connected ConvRadar GA4 when available, buyer questions and competitor pages. Not for store cart products, whole-site audits or date-specific metric drops.
---

# ConvRadar: SaaS and service page audit

Goal: one HTML report about one page that a founder can act on the same day. It says where visitors stop, which statements on the page and at the next step contradict each other, what buyers ask that the page does not answer, what to change first (with mockups), and what that is worth.

Read `references/checks.md`, `references/data.md` and `references/writing.md` before the first call. Build the file from `assets/report-template.html`.

## 0. What you need

If the `convradar_convradar` tools are missing or a call answers Unauthorized, stop and tell the user how to sign in: in the agy CLI open `/mcp`, select `convradar_convradar` and choose Authenticate; in the App or IDE open Customizations, then Installed MCP Servers, and choose Authenticate. If `convradar_convradar` is not listed in `/mcp`, agy started before the Antigravity sign-in: quit and start it again. Never edit MCP config files, call mcp.convradar.com with curl or scripts, or install `mcp-remote` to get around the sign-in.

- The page URL. If the user gave none, ask for it and stop.
- ConvRadar connected to the same site. Unless the user forbids quota use, call `cr_get_account_info` once; it consumes a daily MCP call. If `website_url` is another domain, say the page has no GA4 data in ConvRadar, run steps 2 to 6, and write section 01 from the user's numbers only. If the user forbids quota use, make zero ConvRadar calls, including account info, data description and page capture. Use previously supplied data with its dates and provenance; otherwise write a page-only audit without GA4 numbers.
- Optional, never wait for them: heatmap numbers (scroll reach by page depth, clicks on the decision block), next-step counts if the checkout or signup is not in GA4, past tests on this page. Use them when the user gave them. If they cover another period than ConvRadar, section 01 uses the user's numbers and the ConvRadar window goes to the footer.

## 1. Page numbers (ConvRadar, 7 quota calls total; browser reads do not count)

Follow `references/data.md`. Period: 90 days ending `data_complete_to`. Keep the entire ConvRadar audit within seven daily-quota calls, including account info; browser page reads do not spend this quota. You need: visits that started on the page by source and by device, conversion, form and next-step events in those visits, page views of the page and of its next step, and a mobile speed check when the call budget permits.

## 1b. Where the loss is

Before any page check, locate the loss only from counts joined to the same visits or users and the same page variant. Pricing-page landings, site-wide next-step views and site-wide success events are three different populations; their ratios do not measure pass-through or completion. A shared next step can still supply a separate, clearly labelled site-wide diagnostic, but its totals cannot be attributed to this page.
- Joined counts show a larger loss at the next step on 30+ visits: the H1 and section 01 name that step with its cohort rate; sections 02 and 04 run checks 2 to 6 and the form check there. Mockups show the next step now beside proposed.
- Counts cannot be joined: audit the page and its linked next step, choose emphasis from directly observed friction, and state that the larger loss cannot be located quantitatively. Do not say the page passes visitors on or the next step completes normally from aggregate page-view ratios.
- Joined counts show both steps work: say so with their cohort numbers and keep only problems supported by checks 2 to 6.

## 2. Read the page and its next step

Fetch the full HTML of the page with the browser or web fetch. `cr_capture_via_web_fetch` also spends a daily ConvRadar call, so use it only when the call budget allows; prices that scripts change after load need the browser. Read all text, including tables, tabs, accordions, FAQ, footnotes and the comparison table. Then open every main call to action and read where it leads: checkout, signup, trial, booking, contact form. Run checks 1 to 6 in `references/checks.md` as you go, on the page or the next step as step 1b decided. Any claim that something is absent, unpublished, unanswered or working ("not on the page", "not published", "works"), and any count of buttons or blocks, needs a full-text search of the fetched page first. The H1 and the problem rows never state what the footer lists as "to confirm".

## 3. Buyers and competitors

Checks 7 and 8: about 20 public discussions from the last 24 months, the 3 or 4 most raised questions with counts and how the page answers each, then the 3 nearest competitors.

## 4. Problems

At most 9 rows, one problem per row, about the place step 1b chose, in this order: costs or conditions first seen at the next step and claims with two values (price, plan name, support, contract, trial); promises the delivery breaks; units buyers cannot read; buried answers and an unreachable human path. Evidence is at most 25 words in the form `<where>: <quote, price or count>; <mechanism>`. A problem without evidence is not a row. A buried-answer or human-path row from check 1 outranks smaller rows within the 9: merge the smallest two-value rows into fix 1 before dropping it.

## 5. Fixes and mockups

5 to 7 fixes, cheapest first, numbered. Column 3 is the kind of work: "Text", "Layout + text", "Development", "Billing", "Tracking". Rules:
- Fix 1 is one line naming what becomes one value everywhere. Smaller mismatches go to the footer, at most 6, no typos.
- A fee or cost that contradicts a "no fees" claim is removed; printing it on the plans is the fallback.
- Every unit or spec line flagged by check 5 gets its one-sentence explanation in the "Text for each line" table.
- Name existing blocks by their on-page title. Moving an existing block beats writing a new one; say which block stays.
- Before writing section 04, match each top-3 buyer question and the hero's promise to the block whose heading answers it (check 1). A buried one is fixed by moving that block up after the hero plus a contents bar. This never moves the decision block, so a test on the position of the plans does not block it. An unreachable human path is fixed by a link inside the decision block.
- New copy comes from the page's own text, specs and FAQ. Unknown values are written `$X` or `to confirm` and listed once in the footer.
- The last fix is measurement when the decision block or next step is not tracked (`references/checks.md`, Measurement).
- Traffic, ads, SEO, testimonials and certificates the client would have to produce are out of scope.

Mockups (HTML, in the site's own colours, pins numbered as the fixes): the decision block as proposed, the explanation text for each line you made clickable, every block you move with the contents bar, and the next step now beside proposed when it carries a problem.

## 6. Expected result

When joined counts support the arithmetic, show per-quarter next-step and conversion totals now and after the fixes, plus first-year value of the extra paid conversions at the entry price. State which cohort rate moves, from what to what, and why that target. Then say whether an A/B test can finish at this traffic and give a before/after comparison. Without joined counts for the page and its variant, there is no uplift table or conversion forecast: state the value of one extra paid conversion, the closest valid rate and its current value if available, and site-wide order or trial counts only as context. Never multiply a site-wide registration proxy by this page's traffic or treat a free trial as paid revenue.

## 7. Build and deliver

Fill `assets/report-template.html`: title `<Page name> Page Audit`, kicker `Conversion audit · <host/path>`, H1 of at most 10 words naming where visitors stop or the biggest problem's place, no number; dek with the fixes' result. Sections: 01 Numbers (funnel numbers about the audited place only: visits, reach of the decision block, next step, conversions; site-wide numbers go in the line under the cards; a number you do not have is left out, not replaced), 02 Problems, 03 Buyers and competitors, 04 `<N>` fixes with mockups, 05 Expected result, footer with sources, dates and every `$X`. Run the final pass in `references/writing.md`. Save as `<host without www>-<page slug, "home" for />-audit.html` and give the file.

Chat reply: three sentences at most. Where visitors stop, the first fix, the expected result. Then the file.
