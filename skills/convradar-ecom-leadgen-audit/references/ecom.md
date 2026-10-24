# Store report

Sections in this order. Length: at most 3,500 words; about 2,000 for a good site. An area with no finding gets one line, not a section. Group unquantified fixes in the plan when possible; label an individual deep dive "Not quantified" when the step measurement is unreliable.

These sections apply only after purchase measurement is verified. If the main conversion is unverified, follow the tracking diagnostic in `SKILL.md` and omit the full-report checklist. English headings and labels below are structural examples; translate them into the language of the audited pages.

## Header
H1: the natural translation of "Conversion audit: <Client>". Dek: one sentence, the verified conversion rate on qualified traffic against the vertical, and the step that loses the most. Add a goal line only when the underlying steps and expected extra orders are reliable; compute it with code from extra orders in the plan ÷ the same sessions base. Byline, period, issued date.

## 01 The numbers
- For a high-ticket store (AOV several times the vertical's), the typical bands are directional only; compare each rate with the store's own best device, channel or landing type first.
- Initial data: eight to twelve numbered facts, each a number with a plain verdict when it is an outlier: sessions per day, device share, conversion rate (all and qualified), AOV, items per order if available, sessions per user (sessions ÷ users; without a user count, the returning share stands in), engagement rate with a verdict and the engaged-but-not-converted share (engaged sessions minus purchases, as a share of sessions: the capacity remarketing works on), bounce, returning visitors' share of sessions, of revenue, and their CR, product view → add to cart against 5 to 8% typical.
- Funnel table: sessions → product viewed → added to cart → checkout → purchase, with the step rate; add a search or category step when the data has it.
- If `view_item` fires on pages that are not product pages (it shows up on the homepage or category landings, or exceeds product page views), product views are unreliable: use add-to-cart per 100 sessions per device, and product page views from `table="ga4_fact_page_event_daily"` (`view_item` by `page_path_plus_query_string`, product paths only) when you need the step.
- Funnel by device: sessions (from `cr_get_device_breakdown`), add to cart per 100 sessions, checkout, purchase, checkout success (purchase ÷ checkout) per device, each with its base. Name the step where the device gap is largest.
- Funnel by landing-page type: sessions and purchases per landing page from `cr_query_metrics(metrics=["sessions","purchases"], dimensions=["landing_page"], table="ga4_fact_landing_daily", top_n=100)`, add-to-cart per landing page from the landing-step call (`top_n=100`). Classify every row by path: homepage, category or collection, product, search, content (blog, guides, articles), other. Per type: sessions, product views per 100 sessions (the share that reaches a product), add to cart per 100 sessions, purchases, CR. State the share of sessions the classified rows cover.
- A step rate judged unreliable is still shown once with its benchmark verdict and the caveat; it is not used for impact.
- Homepage landers: the share that reaches a product, and what the first screen offers (categories or a single product). When the homepage → product rate is below the product-landing types, the fix names the first-screen element to change.
- Verdict line: where the store is strong and where the loss is.

## Good site
If CR is at or above the vertical's top quartile and no step is far below its benchmark, the dek says so plainly ("converts at 4.5%, one of the best rates for this price point") and the report becomes: the two or three issues the numbers do show (usually checkout success, speed, remarketing), short page notes, a "Minor issues" list, and a "Tests to launch" list by page type (category, product, cart), three to ten tests each with what to change and the metric. Do not invent problems to fill sections. The good-site verdict is about the steps on the site; a traffic, campaign or tracking problem found in the trust checks is still a main issue in the same report.

## 02 Main issues
Up to five evidenced issues, ranked by supportable cost. Each with its count and place: content or homepage landers that never reach a product, product page add-to-cart below benchmark, cart or checkout success below benchmark (especially on mobile), speed, retention (returning share of traffic and revenue).

## Deep dives, in this order
Write a deep dive only for an area with reliable data or a directly observed page finding. Areas with neither go into one "Not covered" line in the footer with the reason, not into empty sections. Each deep dive: "Data" (counts and rates), then the page read, then "What to change" bullets, then "What to expect" when a trustworthy step rate and comparable target support a range. Otherwise state "Not quantified" and the metric to watch; never derive impact from a broken funnel step. An area with nothing to say gets one line.
1. Page speed: homepage, top category, top product page; verdict with reason. If the tool's summary blames speed for the device gap while Core Web Vitals pass, the verdict follows the vitals and the device split, not the summary.
2. Calls and contact (if call clicks are tracked or a phone number is prominent): call clicks against purchases; phone on empty search, empty category and error pages.
3. Search and navigation: search events per 100 sessions and by device; run two searches on the live site (a top product type, a misspelling) and report what comes back; empty-results page; filters on mobile; header space on mobile. If the store sells parts or sizes that must fit (vehicle, model, size), check whether a fit selector exists and is used.
4. Category pages: share of category landers who reach a product; filter visibility and feedback; consistent images.
5. Content pages (blog, guides): share of traffic, share reaching a product, CR. Read the content lander with the most sessions: count the links to products or categories in its body and say where the first one sits; banners matched to the topic.
6. Product pages: add-to-cart by device; first screen (price, delivery, stock, warranty, reviews near the button, one add-to-cart button); media; description structure (compatibility, specs, documents, FAQ); underperforming products from the product analysis, with a price check against two competitors and the brand's own site for the top one or two. Read every page that states the same fact (delivery, duties, returns, warranty, price) and report contradictions.
7. Cart and checkout: add to cart → checkout → purchase by device; what add-to-cart does on each page type (drawer, pop-up, redirect, nothing; read the theme's cart script or product form when the cart is built by script); cart page (button position, distractions, express pay, delivery and returns info, continue shopping, discount pop-ups over buttons); checkout. If the checkout works, say so: "Not a problem: checkout". If the cart cannot be read, say so, ask for phone screenshots of the cart with one item, and still give the cart checks as tests from the funnel step: after add to cart the visitor sees the order total, the items, delivery time and cost, stock, payment methods and express pay, with nothing (discount pop-up, chat) covering the checkout button.
8. Acquisition and retention: channels and campaigns with CR; email CR against the site (2 to 5 times the site is typical); returning share of traffic and revenue; sessions per user against the sessions a purchase takes when known; remarketing present or not (campaign names, Google Ads and Meta tags in the HTML); audiences sized from the funnel: carts not bought (add to cart minus purchases), product viewers who did not add, checkouts not finished; a stated verdict, never "open".

## Plan
Plan table: order, area, what to change, what to expect (range), the step metric to watch. Then the implementation rule in one paragraph: change one area at a time, measure the step rate of that area (product viewers who add to cart, cart to checkout) on two or three segments (device, new vs returning) before the site CR, then move to the next.

## Tests to launch and minor issues (every store)
Tests grouped by page type (category, product, cart, homepage), three to six per type, each a one-line change and the metric it moves. Then minor issues, one line each (sticky menu, logo size, overlaps). For a good site this section carries most of the report.

## Footer
Sources, period, limits (what GA4 cannot see: repeat purchase, sessions per purchase, ad accounts, email flows), the numbers that would sharpen the estimates.

## Report checklist (tick every line before saving)
- [ ] Header; goal line only when its inputs are reliable
- [ ] 01 The numbers: initial data with verdicts, funnel, funnel by device with base and largest-gap step, funnel by landing type with coverage, homepage landers, verdict line
- [ ] 02 Main issues with count, place, cost (or Good site verdict)
- [ ] Deep dives for the areas with data or a page finding, each with "What to expect" or "Number to watch"; the rest in one "Not covered" footer line
- [ ] Remarketing verdict with audience sizes
- [ ] Plan
- [ ] Tests to launch by page type + minor issues
- [ ] At most 3,500 words (2,000 for a good site)
- [ ] "Not a problem" at least once
- [ ] Footer
