# Data from ConvRadar

All calls read-only. `cr_get_account_info` and `cr_describe_data` are free. Window: the 90 days ending `data_complete_to`; pass `date_from`, `date_to` every time.

## Rules for reading any number

- The site's sessions, conversions and revenue for the window come from `cr_get_overview_metrics`; that total is used everywhere a site total appears. Breakdown tables (device, channel, country, landing page) come from other sources and can differ by a few percent; say so once and compute shares inside each table from that table's own total.

- "No rows" is not zero. Write "not in the data" and move on.
- Event counts are events, not people. A rate built from events is "per 100 sessions", not "% of people", unless the tool says users.
- Under 10 conversions in a segment: quote the count, no rate, no comparison. Under about 30 conversions in the window: no money estimate per month; give counts.
- `landing_page` has no trailing slash (`/pricing`); `page_path_plus_query_string` keeps it (`/pricing/`). If a filter returns nothing, retry once with the other form.
- The warehouse event tables keep only funnel events (page_view, view_item, add_to_cart, begin_checkout, purchase for stores; session_start, form_start, form_submit, generate_lead, qualify_lead, sign_up, thank_you_page for leads). Any other event (button clicks, calls, video, search) comes from live GA4: ask `cr_query_metrics` for `event_count` by `event_name` plus one other dimension without naming a table; the answer says `source: ga4_live`.
- `sessions` and an event count cannot come from one call when grouped by campaign or page; make two calls and join them yourself.
- An unverified conversion is not a zero conversion. If the primary event, another submit event and the confirmation page disagree, first check whether they count the same action and unit, repeated views, and the same date window. If no source can be validated, use the tracking diagnostic in `SKILL.md`; do not choose the largest count, average them, or scale device, source or landing-page event counts to a confirmation-page total.
- Event counts by page (not landing page) exist only for store events (`table="ga4_fact_page_event_daily"`, dimension `page_path_plus_query_string`). For lead events by page use the landing page (`ga4_fact_landing_step_daily`) and say "in visits that started on the page".

Core calls = the trust checks below plus the list for this kind of site (about 25 with three speed checks). Several checks can share one call: one event-vocabulary call serves the trust check, the funnel and the call-click count.

## Trust checks (always, before analysis)

1. `cr_get_account_info`: `primary_conversion` (event, count, alternatives), `business_model`, `website_url`, `data_complete_to`.
2. Event vocabulary: `cr_query_metrics(metrics=["event_count"], dimensions=["event_name"], date_from, date_to, top_n=60)`. If it returns no rows, use `cr_describe_data()` for the event names and `table="ga4_fact_funnel_step_daily"` for the funnel events' counts. Look for: the counted conversion and its volume, thank-you or confirmation page events, call clicks (`*phone*`, `*call*`, `click_phone`), form events (`form_start`, `form_submit`), CTA click events, search (`view_search_results`), purchase. A large mismatch with the thank-you signal triggers the reconciliation in step 4a; it does not by itself establish which count is correct.
3. Device and qualified traffic: `cr_get_device_breakdown(date_from, date_to)`. A device with 15%+ of sessions and a conversion rate under a tenth of the others, or engagement under half, is unqualified. Check its source with `cr_get_traffic_breakdown(sort_by="sessions", top_n=25)`. Report "N sessions, M of them <segment> converting at x%; without them the site converts at y%" and use y from then on.
4. Geography: `cr_get_geo_breakdown(date_from, date_to, top_n=15, sort_by="sessions")`. If the business serves one country or region, give the rate on that market. Lead events by country: `cr_query_metrics(metrics=["event_count"], dimensions=["country","event_name"], table="ga4_fact_geo_event_daily", filters={"event_name": <lead event>}, top_n=20)`.
4a. Where the counted event fires: compare its count with page views of the thank-you or confirmation page (`cr_query_metrics(metrics=["screen_page_views"], dimensions=["page_path"], top_n=100)`, find the row) for the same dates, and compare month by month if totals differ. Page views and submit events count different actions and can each repeat. Agreement supports the mapping but does not prove unique leads. If they disagree, inspect the form success behavior, event definition, duplicate fires and redirects. Use a replacement as the lead count only when that action is demonstrably one per completed lead; otherwise mark the conversion unverified and switch to the tracking diagnostic. Never multiply event splits by a page-view ratio.
4b. Sessions per user: `cr_query_metrics(metrics=["sessions","total_users"])` for the window.
5. Trend and tracking start: `cr_get_overview_metrics(date_from, date_to)` gives the change against the prior period of equal length. For the conversion event's first day: `cr_query_metrics(metrics=["event_count"], dimensions=["date"], filters={"event_name": <conversion event>}, top_n=90, sort_by="date")`. A break over 30% is traced with the sources call and the campaign call below, current against prior window.
5a. Conversions with no session: purchases or leads with a blank source, device or landing page (rows like "", "(not set)", "(other)"). Above about 10% of the total, say so (often offline, phone or server-side orders), give step rates on the attributed ones and the total once, and never mix the two in one rate.
6. Junk segments: in the sources and campaign tables, any row with a large share of sessions and engagement under a third of the site's (or a blank campaign with near-zero engagement) is shown apart with the site rate with and without it.

## Calls: lead generation

1. Overview: `cr_get_overview_metrics(date_from, date_to)`.
2. Funnel events by device: `cr_query_metrics(metrics=["event_count"], dimensions=["device_category","event_name"], filters={"event_name":[<CTA click>, "form_start", <lead event>, <call click>]}, date_from, date_to)`. Funnel = sessions → CTA click → form start → lead. The step with the biggest drop is the first candidate.
3. Sources: `cr_get_traffic_breakdown(sort_by="sessions", top_n=25)` for sessions; lead events by channel from `cr_query_metrics(metrics=["event_count"], dimensions=["session_default_channel_group","event_name"], filters={"event_name":[<lead event>]})`.
4. Campaigns: sessions `cr_query_metrics(metrics=["sessions","engaged_sessions"], dimensions=["session_campaign_name"], table="ga4_fact_source_campaign_daily", top_n=30)`; leads `cr_query_metrics(metrics=["event_count"], dimensions=["session_campaign_name","event_name"], filters={"event_name":[<CTA click>, <lead event>]}, top_n=60)`. Join: lead rate per campaign. Group into campaigns that work and campaigns that do not, with their share of sessions and of leads. Campaign names with remarketing, retargeting, RLSA or display show whether remarketing exists.
5. Landing pages: `cr_get_landing_pages(sort_by="sessions", top_n=25)`; lead events by landing page `cr_query_metrics(metrics=["event_count"], dimensions=["landing_page","event_name"], table="ga4_fact_landing_step_daily", filters={"event_name":[<lead event>, "form_start"]}, top_n=60)`. Find: pages with many sessions and zero leads (content pages), the money pages and their rate, pages whose rate is far below the site's.
6. New vs returning: `cr_query_metrics(metrics=["sessions"], dimensions=["new_vs_returning"])` and `cr_query_metrics(metrics=["event_count"], dimensions=["new_vs_returning","event_name"], filters={"event_name":<lead event>})`. Report the share of leads that come on a return visit and the two rates.
6a. Where channels land: `cr_query_metrics(metrics=["sessions","engaged_sessions"], dimensions=["session_source_medium","landing_page"], table="ga4_fact_landing_daily", top_n=60)`.
6b. Page views of a money page (everyone who saw it, not only landings): `cr_query_metrics(metrics=["screen_page_views"], dimensions=["page_path"], top_n=100)`; find the row.
7. Benchmark: `cr_compare_to_benchmark(date_from, date_to)` (vertical auto-detected; override if wrong).
8. Speed: `cr_check_page_speed(url)` for three pages.
9. Optional, for a page the numbers point at: `cr_query_metrics(metrics=["sessions","engaged_sessions","scrolled_users","avg_session_duration"], dimensions=["device_category"], table="ga4_fact_landing_daily", filters={"landing_page":"<path>"})`.

## Calls: ecommerce

1. Overview: `cr_get_overview_metrics(date_from, date_to)` (sessions, CR, revenue, AOV).
2. Funnel: `cr_get_funnel(date_from, date_to)`.
3. Funnel by device: `cr_query_metrics(metrics=["event_count"], dimensions=["device_category","event_name"], table="ga4_fact_funnel_step_daily")` plus sessions per device from step 3 of the trust checks. Checkout success per device = purchase ÷ begin_checkout.
4. Funnel by landing-page type: `cr_query_metrics(metrics=["event_count"], dimensions=["landing_page","event_name"], table="ga4_fact_landing_step_daily", filters={"event_name":["view_item","add_to_cart","purchase"]}, top_n=100)` and sessions from `cr_get_landing_pages(sort_by="sessions", top_n=25)`. Group landing pages by type from their path and title: homepage, category or collection, product, search, blog or article, other. Per type: sessions, share that views a product, add-to-cart, purchases, CR.
5. Search and other events from live GA4: `cr_query_metrics(metrics=["event_count"], dimensions=["event_name","device_category"], filters={"event_name":["view_search_results","search", <call click>, <vehicle or size selector>]})`.
6. Products: `cr_get_product_analysis(top_n=15, status="underperforming")` (no date arguments; it runs on its own last window, state those dates and skip it if they are older than a month) and `cr_get_product_performance(date_from, date_to, sort_by="views", top_n=15)`. Product view → add to cart per product.
7. Sources and campaigns: `cr_get_traffic_breakdown(sort_by="sessions", top_n=25)`; campaigns from `table="ga4_fact_source_campaign_daily"` with purchases and revenue. Email (medium email) conversion against the site; remarketing campaigns present or not.
8. New vs returning: `cr_query_metrics(metrics=["sessions","purchases","purchase_revenue"], dimensions=["new_vs_returning"])`: share of traffic, share of revenue, CR each.
9. Benchmark: `cr_compare_to_benchmark(date_from, date_to)`.
10. Speed: `cr_check_page_speed(url)` for homepage, top category, top product page.

## What the data cannot give

Sessions per purchase, repeat purchase rates, gender, phone call outcomes, ad account settings, heatmaps and email flow results. When a conclusion needs one of these, name it in the footer as the number that would settle it.
