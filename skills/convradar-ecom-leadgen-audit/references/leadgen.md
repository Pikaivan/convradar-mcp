# Lead generation report

This reference covers service lead generation. A SaaS or software product remains out of scope even when it collects demo or contact leads; follow `SKILL.md` and route it to the separate SaaS audit skill.

Sections in this order. Numbers in the section headings are the section numbers. Target length 3,000 to 5,000 words including mockup text; a section with no finding is one line.

These sections apply only after the lead conversion is verified. If the main conversion is unverified, follow the tracking diagnostic in `SKILL.md` and omit the full-report checklist. All English headings and labels below name report elements; translate them into the language of the audited pages.

## Header
H1: the natural translation of "Conversion audit: <Client>". Dek: one sentence, the verified lead rate of the main conversion type (never a sum of types) on the market the business serves against the vertical median and the place where most people are lost. Byline: preparer if the user gave one, a localized ConvRadar attribution, period with dates and days, issued date.

## 01 The numbers
- Five or six stat cards, a number and one line each: sessions with device share; lead rate (qualified traffic) against the vertical median and top quartile; the funnel's weakest step rate; revenue or bookings per offer when the site sells online; the share of sessions on pages that never convert; any seasonality or trend the window shows.
- The funnel, one row per step: sessions → CTA click → form start → lead (thank-you), plus call clicks. One funnel per conversion type (enquiry, booking, call); never summed. A site that takes bookings or payments online gets its own funnel: offer page viewed → booking opened (page views of the booking app's entry page) → contact details given → paid, with the step where contact is captured named.
- One line that says where the loss is: before the form ("97% never start") or inside it ("half who start finish").
- Device table: sessions, CTA rate, lead rate, finish rate (lead ÷ form start), call clicks.
- Sources table: channel, sessions, leads, with the line that explains the biggest mismatch between share of sessions and share of leads.
- Geography in one line: the market served and the out-of-market traffic.
- Landing pages in one paragraph: homepage share and its leads, the best and worst money pages, pages with many sessions and no leads.
- One decision line that closes the section.
- Data limits in one line (cookie consent, calls not tracked, what the counted event really is).

## 02 Main problems
Up to four evidenced problems, in the order of supportable cost. Each: a short heading that states the problem, then one or two sentences with the count, the place and the cost per quarter or the exact missing input.

Typical problems to test for, not to assume:
- CTA clicked but the form never started (click count minus form starts).
- Form started but not sent, against 35 to 50% completion for comparable forms (65%+ is top quarter).
- A page whose finish rate is far below the rest of the site: read what its buttons open.
- Campaigns or channels that take a large share of sessions and return few leads.
- Content pages with many sessions and zero leads: no bridge to the offer. List them with sessions, group them by why people read them (fixing a problem, choosing a type, looking up a spec), and give one next step per group, matched to that intent.
- Returning visitors convert better but nothing brings anyone back.
- Contacts captured but not converted (a booking with an email but no payment).

Contacts captured but not converted (an email left in a booking that was never paid, a quote request with no booking) are valued at the average order or booking value from the page's own prices: "41 unpaid bookings with an email, about £3,280 at the £80 average".

## 03 onward: one section per main problem
Locate the loss three ways before reading the page: by step (between which two events), by device (the rate on each device with its base), and against a sibling flow on the same site (another offer, another page, another device) that already does better. The sibling's rate is the impact target.

Each section:
1. Where the loss sits: between which two measured steps, on which pages, the device split there.
2. What the visitor sees: the page or form read (fields, heading, first screen on a phone, what the button opens, overlays, prices and terms before the step). Captions, numbered.
3. Now and proposed: an HTML mockup of the block as it is and as proposed, numbered pins matching the changes. Only where a page block is the cause; a campaign or tracking problem gets a table instead.
4. What to change: bullets, each a concrete change with the on-page text.
5. Impact: one line of arithmetic from the site's own numbers.

## The form (always, even when it is not a main problem)
- Numbers per page and per device: CTA clicks, form starts, verified leads. Use clicks minus starts as a form-open loss only when both events count the same journey, unit and period, and there is no duplicate firing; otherwise list the two event counts separately without subtracting. Starts to verified leads is a finish rate only when the events represent the same cohort and journey. With no trustworthy form_start event, do not label CTA clicks divided by leads a completion rate; report clicks and verified leads separately and name the missing event.
- The form read, from the HTML of the pop-up or page form: heading (does it repeat the button or name what the visitor asked about), a line on what happens next and when, a line that the visit or quote is free with no obligation, required fields and which comes first (phone first is a finding on a site that never addresses the fear of a sales call), autocomplete attributes on name, email and phone, anything that looks like an error before typing, whether forms on different pages differ.
- Finish rate (form starts → leads, from the landing-step call) for every landing page with 40 or more starts, in one table. A page far below the rest is read: what each of its buttons promises and what the form it opens says; that page gets its own audit.
- Problem → solution table, then the form now and proposed as a mockup with pins.
- If the data says the form is fine (finish rate in the top quarter), say "Not a problem: the form fields" and put the fix where the loss is (before the first keystroke, or before the click). Shortening a form that is already short is a named non-fix.

## Benchmarks
Table: metric, the site, bottom quarter, median, top quarter, verdict, from `cr_compare_to_benchmark` plus the form completion you computed. Then two or three sentences: is the site average, and does one number break the pattern and change where the problem is. Use benchmarks to locate the problem, not to grade: write the sentence "One number breaks the pattern: <metric>. So the loss is not <where it looked> but <where it is>." when one does. Size the gap from the site's rate to the top quarter in leads per quarter at current traffic.

## Page speed
Table: page, mobile score, LCP, CLS. Field data that is identical for every page is for the whole domain; say so and compare pages on the lab score. Verdict with its reason: if all pass Core Web Vitals and the drop is the same on phone and desktop, speed is not the constraint and no money should go there. Name one concrete waste only if the speed tool shows one.

## Acquisition and retention
- Channel table: sessions, share, lead rate. Campaign table (required): every campaign with meaningful sessions, its sessions, lead rate and leads, ranked; then a two-row summary, the group that works and the group that does not, each with its share of sessions, share of leads and lead rate. Name the ones to stop and the ones to keep (a search campaign that converts stays even if its group is weak).
- Where each channel lands: `table="ga4_fact_landing_daily"`, dimensions `session_source_medium` and `landing_page`, for paid search, paid social, organic social and email. A paid or social channel landing on pages that never convert (events, blog, a link-in-bio page) is a finding.
- Channels with traffic and zero leads (a LinkedIn or Meta campaign, a bio-link page): named with sessions.
- Email continuity: email-medium sessions by month; one send then nothing is "one send, not a programme".
- What is missing: remarketing (no campaign with remarketing or display in the names), the share of leads or bookings that come on a return visit and the returning visitors' rate against new, a paid channel the site gets organic traffic from but does not buy (Bing and Yahoo organic sessions size Microsoft Ads), tags absent from the page HTML (Meta pixel, TikTok, Google Ads remarketing).
- Ordered actions, then a possible-outcome table with the assumption in each row and a cautious range.

## What buyers ask
Run at least five searches built from the service and the market served: "<service> <city> reddit", "<service> quote worth it", "<service> how to choose", "<service> scam OR red flag", "<service> cost forum". Read the threads that come back and keep four to six direct quotes with their source. Count the questions raised, split them into what stops someone asking for a quote at all (the landing page's job) and what decides between quotes. Matrix: question × the homepage, the main landing page and the form, each cell answered / mentioned / missing. If the searches return no discussions, use reviews instead: the business's own and two or three competitors' reviews (Google, Trustpilot, Yelp, HomeStars or the local equivalent) and competitors' FAQ pages; quote what reviewers praise and complain about, and build the matrix from those questions. Only when both are empty, say so in one line.
When the same questions are open on several pages, the fix is one block in the shared template that answers them all (a paragraph each), written once. A question that text cannot answer (is this company real) points to proof that can: a short video with a named person, third-party ratings.

## Page audits (always: the homepage and the main page of each offer; a third when a page has an outlier rate)
A page is never cleared on its landing rate alone. Each:
- Three stat cards: sessions with device split, the page's lead rate as a landing page against the site's rate, and of everyone who viewed the page (page views from `dimensions=["page_path"]`) the share who landed on it; one more number that matters (scroll to bottom, time, CTA rate).
- When the page has a form or button event that fires only on that page, split its rate: all events ÷ page views (everyone), landing-step events ÷ landing sessions (arrived cold), and the rest ÷ the rest (arrived after another page). A page that works only for visitors who arrived after another page needs a cold-visitor first screen.
- The peak-intent element on the page (a calculator, a configurator, a price table, a booking widget): what its result offers the visitor next and what capture sits beside it. A result that ends in a phone number or nothing is a finding.
- Every button on the page that leads to the conversion: its promise and what it opens. Several promises (an offer, financing, a quote) opening the same generic form, which drops the promise, is a finding; check it against the page's finish rate.
- The first screen on a phone: the first button or form and its target; proof before or after it.
- What the visitor sees, then problems as subheads, each with its position on the page and the number behind it.
- "Not a problem: …" for any part that works.
- Scroll: the share of landing sessions that reach 90% of the page (`scrolled_users` ÷ sessions from `ga4_fact_landing_daily`), and which blocks sit below where most people stop.
- Blocks that already exist elsewhere on the site and would close this page's gap (a badge row, reviews, a form with a heading) are moved, not rewritten. Give the proposed order of the page as a list: each block marked kept, moved up, new, or collapsed.
- What to change, and an impact line at a rate another page of the same site already reaches.

## Plan and expected impact
Table: # , when (Now, Next, Then), fix (section reference), cost (kind of work: "an afternoon in the ad account", "copy", "small dev", "media budget"), why this one (the number behind it), expected impact (range). Urgent first (a season, a broken step, a leak of personal data), then within each group the cheapest first. "Not on the list" rows for work the data says would waste time (often speed when vitals pass, a redesign when the design is competent, shortening a form that is already short), each with its reason. Then two lines: how to read the ranges (arithmetic on the site's own rates, not industry multipliers), and the rule to ship one fix at a time and read the number its row names before the next. Close with what the quantified fixes deliver together, without double counting fixes that act on the same people, and which items are left unpriced and why.

## Footer
Sources (GA4 via ConvRadar with dates, pages read on date, speed measured on date, web research count), limits, and the numbers that would sharpen the estimates (call outcomes, ad account, heatmap).

## Report checklist (tick every line before saving)
- [ ] Header: localized H1, dek with the verified lead rate, byline, period
- [ ] 01 The numbers: cards, funnel per conversion type, device table, sources, geography, landing pages, decision line, data limits
- [ ] 02 Main problems: only evidenced problems, each count + place + supportable cost or missing input
- [ ] One section per main problem with "what the visitor sees", now/proposed mockup, what to change, impact
- [ ] The form, with per-page numbers and the form read
- [ ] Benchmarks with the gap to the top quarter in leads
- [ ] Page speed with verdict
- [ ] Acquisition and retention with where channels land, zero-lead channels, email continuity, remarketing, outcome table
- [ ] What buyers ask (or one line that no discussions were found)
- [ ] Page audits: homepage and main money page at least
- [ ] Plan with "Not on the list"
- [ ] Footer
