# Page numbers from ConvRadar

Seven daily-quota calls at most for the whole audit. Assume every ConvRadar MCP call consumes one, including `cr_get_account_info`, `cr_describe_data`, `cr_capture_via_web_fetch` and `cr_check_page_speed`; verify the remaining count in the account-info response. If the user forbids quota use, make zero ConvRadar calls. Prefer browser or ordinary web reads for the page and next step. Pass `date_from` and `date_to` to every dated call: the 90 days ending `data_complete_to` (or the day before `data_available_to`), never starting before `data_available_from`.

## Path formats

- `landing_page` is stored without the trailing slash: `/pricing`, not `/pricing/`. If a call returns no rows, retry once with the slash.
- `page_path_plus_query_string` keeps the slash: `/pricing/`. Query strings stopped being stored on 2026-10-09, so never count anything by query string (plan ids or variants in the URL). If the live page has variants in its query string, path-level GA4 totals mix those variants. Name the variant you read, but do not attribute the mixed totals or any downstream conversion to it.
- "No rows" is not zero. Say "not in the data" and move on.

## Calls

1. Visits that started on the page, by source:
   `cr_query_metrics(metrics=["sessions","engaged_sessions","scrolled_users","avg_session_duration"], dimensions=["session_source_medium"], table="ga4_fact_landing_daily", filters={"landing_page": "<path>"}, date_from, date_to, top_n=25)`
2. Same, `dimensions=["device_category"]`.
3. Events in visits that started on the page:
   `cr_query_metrics(metrics=["event_count"], dimensions=["event_name"], table="ga4_fact_landing_step_daily", filters={"landing_page": "<path>"}, date_from, date_to, top_n=50)`.
   Keep the account's `primary_conversion` events, form events (`form_start`, field focus, `form_submit`) and next-step events (`begin_checkout`, `sign_up`, `generate_lead`, trial or booking events). These names also tell you what is tracked for the Measurement fix.
4. Page views and 90% scroll across visits to the page, not only landings:
   `cr_query_metrics(metrics=["screen_page_views","scrolled_users"], dimensions=["page_path"], date_from, date_to, top_n=100)`.
   The filter is ignored on this route: find the page's row in the list, the next step's row (checkout, signup or contact path) and its success page's row when they are on the same host. These are page views, not unique people or an ordered funnel. Do not call the success-to-step view ratio a completion or conversion rate.
5. Speed when the call budget permits: `cr_check_page_speed(url="<page url>", page_type="landing")`.
6. Optional, for comparison only if a call remains: `cr_get_landing_pages(date_from, date_to, sort_by="sessions", top_n=25)`. Compare engagement only with pages of the same type (other product, plan or service pages), never with articles. `cr_describe_data` also takes one call, so substitute it for an optional call when event meaning needs checking.

## Reading the numbers

- Visits = sessions from call 1. Show unqualified rows apart in one line under the stat cards: a source or device with 15%+ of visits that, against the rest of the page's visits, has under half the engagement rate (engaged ÷ sessions), under half the share reaching 90% scroll, or under a third of the average duration. Example: "1,240 visits; 310 of them from one referral source with 9% engagement. 930 without them."
- `scrolled_users` = people who reached 90% of the page height. It does not tell who reached a block in the middle. Reach of the decision block needs a heatmap; without one, leave that stat card out.
- Event counts are occurrences, not people. When the events are filtered to visits that started on the page, describe them as "event occurrences per 100 page landings"; repeated events are not a visitor conversion rate.
- An event that fires more than 5 times as often as form submissions or signups (click trackers, auto-fired "lead" events) gets no stat card and no rate. Mention it only if it is the account's mapped conversion, as "counts every <what it counts>, not leads".
- Do not give a custom event a business meaning (paid, order, lead, trial) unless the page or URL where it fires confirms it. Otherwise quote it by its name.
- Before writing that conversions go uncounted, check every candidate conversion event in `cr_describe_data` and the account's `primary_conversion`; if another event still records them, the finding is "counted only by <event>", not "not counted".
- A next step shared by every page (one /signup or /checkout for the whole site) supplies site-wide diagnostics only. Without session- or user-level linkage to the audited page, its visits, form events and success events cannot be assigned to this page or plan.
- Do not subtract landing sessions from page views to infer arrivals: a session and a page view are different units, and repeat views remain. Do not divide success-page views or success events by next-step views and call the result completion. A conversion rate needs a denominator and numerator from the same cohort of unique visits or users, with event deduplication where needed. If those joins are unavailable, report the observed counts separately and say that completion is not measured.
- If the next step has no rows at all (another domain, not tracked), say so in one line and ask for the counts in the chat reply. Do not estimate them.
- Conversions below 10 in 90 days: quote the count, no rate, no comparison between segments.
- If `primary_conversion` has confidence under 0.6, or its event does not fit the business (a chat message on a SaaS with signups or payments), say so in the line under the stat cards and use the event that fits.
- A pricing page with a free plan has two next steps: the free signup (its events or success page) and the paid checkout. Give both. If payment is not tracked in GA4, ask for the billing system's paid count in the chat reply.
- If admin, internal or staging paths appear in the page list, say that next-step views include team visits.
- `cr_describe_data` lists every event the site fires and consumes one daily-quota call. Order, subscription or trial events that exist site-wide but not in visits from this page are context, not a ceiling on this page's conversions or a page-level rate.

## Footer line

"GA4 via ConvRadar, <date_from> to <date_to>. <excluded rows, if any>. Page and next step read <date>; positions are share of page text (or heatmap height). Buyer research: <N> public discussions, <years>, counts are discussions, not market share. Competitor pages read <date>. $X: <list>."
