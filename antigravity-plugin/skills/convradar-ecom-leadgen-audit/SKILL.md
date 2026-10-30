---
name: convradar-ecom-leadgen-audit
description: Use when a user asks for a full conversion or CRO audit of an ecommerce store or a lead-generation site that sells services and collects enquiries, quotes, bookings or calls. Uses ConvRadar GA4 data, live pages and public research to deliver an actionable HTML report. Typical requests include "audit my store", "what should we fix to get more leads", "аудит магазина" and "полный аудит лидогенерации". Do not use for SaaS or software products, even when their primary conversion is a demo or contact form; a separate SaaS audit skill handles those. Do not use for a one-page review, a single main problem or a drop on named dates.
---

# ConvRadar: E-commerce and lead generation conversion audit

Scope: ecommerce stores and service businesses whose site generates leads. Do not use this skill for SaaS or other software products, even if a demo, trial, contact or quote form is their main measured conversion. A separate SaaS audit skill covers that business model. Goal: one HTML report a business owner can act on the same week. It says what the site converts, where visitors are lost and why, what that costs, what to fix first, and what is fine and should be left alone.

Before the first call, read `references/data.md` and `references/resume.md`. After the free `cr_get_account_info` identifies the business model, read `references/leadgen.md` or `references/ecom.md`, plus `references/pages.md` and `references/writing.md`. Build the file from `assets/report-template.html`.

## 0. Which audit

Call `cr_get_account_info` (free). Use `business_model`:
- First check whether the business sells software or a SaaS subscription. If so, stop this skill even when `business_model` says `lead_generation` or the site has a demo form. Use the separate SaaS audit skill when available; otherwise explain that this audit does not cover SaaS.
- `lead_generation`, or a site whose conversion is a form, a booking request, a call or a quote: follow `references/leadgen.md`.
- `ecommerce`: follow `references/ecom.md`.
- Anything else (media, content, apps without a store or service lead flow): explain that this audit covers ecommerce stores and service lead-generation sites. Do not produce an unrelated whole-site audit under this skill.

If `website_url` is another domain than the one the user named, say the data is for `website_url` and ask which site to audit. Stop. A subdomain of the named domain (app., shop., book.) is the same site; audit both hosts and say which pages sit on which.

Window: the 90 days ending `data_complete_to`. Pass `date_from` and `date_to` to every dated call.

Before any paid call, create the audit checkpoint described in `references/resume.md`. The core calls remain the full audit route; Free users may hit their weekly allowance and receive the evidence-backed partial result described under Limits. Do not silently reduce the audit to fit a plan.

## 1. Trust the data before using it

Run the checks in `data.md` → "Trust checks" before any analysis:
- What the counted conversion event really is. Read the page where it fires (thank-you page, booking step, form). If it fires before the real conversion (a booking started, a button click), say what it measures and look for an outcome event that can be validated. If its volume is far below a confirmation signal or known purchases, investigate the mismatch; do not substitute another event solely because its count looks plausible.
- Unqualified traffic: a device, source or country with 15% or more of sessions and engagement under half the site's average, or a conversion rate near zero against the rest; also several small sources of that kind that together pass 5%. Report the numbers with and without it and use the qualified figure from then on. Do not trust a "clean" verdict from a tool over these numbers.
- Out-of-market traffic: countries the business cannot serve. Show the conversion rate on the market it serves.
- More than one conversion type (enquiry and booking, store and call): one funnel per type, never summed.
- Tracking that starts inside the window (the conversion event's first day is later than `date_from`): use the days since it started for every rate, and say so once.
- A trend break: sessions or conversions down or up more than 30% against the prior period. Trace it by channel and campaign before anything else; it often is the main problem. A break that starts inside the window (a surge or a drop from a given date): rates on the days before it, the break reported as a finding with its date and the source that carries it.
- A step event that cannot be right (more leads than form starts, checkout starts far below a later checkout step, product views firing on the homepage): name it as unreliable, do not headline it, and measure that step another way (per 100 sessions, or from the page views of that step's page).
- The business serves one city, region or country: every rate after this check is on the served market, said once.

After these checks, classify the main conversion as **verified**, **partly verified**, or **unverified**. If the counted conversion disagrees with its confirmation signal and no replacement can be validated, or the only conversion path has impossible step counts that cannot be reconciled, switch to the tracking diagnostic below. If only one step is broken while purchases or leads remain verified, continue the audit using the verified outcome; omit rates and impact based on the broken step. Never turn a suspect event into a fact by scaling it to a page-view count.

### Tracking diagnostic

When the main conversion is unverified, do not force three to five CRO problems, a conversion benchmark verdict, a target conversion rate, revenue at risk, or an uplift forecast. Deliver a shorter HTML tracking diagnostic in the site's language: (1) what can be counted reliably with dates and sources, (2) one to three observed mismatches with their counts and the exact events or pages involved, (3) the minimum instrumentation fixes in priority order, and (4) a verification check for each fix. For leads, propose firing the conversion only after the server accepts the form, once per `submission_id`, then reconciling unique accepted submissions against unique conversion IDs by day, excluding identified test and spam submissions. For stores, reconcile unique paid `transaction_id` values with unique purchase events. The target is one event per accepted outcome; name known consent or integration exclusions separately instead of declaring mismatched totals equal. Treat thank-you page views as a secondary signal, not unique outcomes. Add a short page observation only when directly verified. State that conversion recommendations and impact require validated conversion measurement. Reuse the report template's layout and source footer, replace its example sections with these diagnostic sections, and remove all unused placeholders. Label it as a tracking diagnostic throughout. Do not force three problems when fewer are evidenced.

## 2. Numbers and funnel

Follow `data.md` → "Calls" for this kind of site. You need: sessions, device share, the conversion rate against the vertical benchmark, the funnel by step (overall, by device, by landing-page type or offer), sources and campaigns with their conversions, geography, top landing pages with their conversions, new versus returning. Write the rate of every consecutive pair of steps (page → app or form opened → contact given → paid or sent), per device, and next to the same pair in a sibling flow (another offer, another device). Rank the losses from that table, not from the first drop that looks large; the biggest loss in people is the first main problem candidate.

## 3. Read the pages

Follow `pages.md`: screenshots for the three or four pages that carry the report, HTML for the rest. At least: the homepage, the top two landing pages by sessions, the page with the main form or the cart, and every page the numbers point at. For each: the first-screen action on a phone, what the call to action opens, the form fields or the add-to-cart behaviour, prices and terms before the conversion step, proof and where it sits, tags present. Run the speed check on the three pages that matter most.

If the page that holds the main problem cannot be read (blocked, or built by script), say so in that section, write the cause as a hypothesis "to verify on a phone", and ask in the chat reply for a phone screenshot of that page.

## 4. Main problems

For a verified conversion, report up to five main problems, only as many as the evidence supports. Order them by supportable cost. Each is one sentence: the count, where it happens (between which two steps, on which page, on which device), and what it costs per quarter from the site's own numbers: money when the window has 30 or more conversions, otherwise a count ("about 7 more enquiries a quarter"). A problem without a count and a place is not a problem. A problem with no supportable cost says why and names the number to watch. For an unverified conversion, use the tracking diagnostic instead.

Rates that clear or blame a step are computed per page and per device, never pooled across pages: a form that sends 42% site-wide can send 21% on the page that matters. A suspect that the numbers clear goes into the report as "Not a problem: …" in one line.

If the site converts at or above the vertical top quartile and no step is far below its benchmark, say so in the first line of the report and turn the audit into minor issues plus a list of tests (`ecom.md` → "Good site").

## 5. Sections and fixes

For a verified conversion, write every section listed in the reference file for this kind of site, in that order, and tick its "Report checklist" before saving. A section without data is still written, as one line that says what was not measured and why. Every deep-dive section has three parts: the data (counts and rates), the cause on the page (what the visitor sees), and what to change with the expected effect. Fixes name the real block, button or field by its on-page text. Moving or reusing a block that already exists beats writing a new one. Copy in mockups comes from the site's own facts; unknown values are written in the report language as a request to confirm.

## 6. Plan and expected impact

Close with the plan: fixes in order (urgent and cheap first, then what is closest to the conversion, then acquisition), each with what changes and the expected impact. Every fix gets a range from a rate the site already reaches, or names the one missing input that would price it. Impact rules are in `writing.md` → "Impact": only rates the site already reaches somewhere (another page, another device, a sibling flow), the vertical median, or a stated range; no double counting; "Not quantified" where the data cannot carry a number. Name what is deliberately not on the list (often speed, a redesign, the form length) and why. A step the data could not measure (no click event on the main button, no checkout sub-steps) gets a "Now" row to add that event, so the next read can see it.

## 7. Build and deliver

Determine the report language from the visible copy on the homepage and the main conversion pages, not the browser locale. For a multilingual site, use the language of the conversion journey being audited; if two journeys have materially different languages, name the audited locale in the report and do not mix them. Fill `assets/report-template.html`: translate every visible string, including the title and H1, navigation, section headings, table labels, byline, status labels, mockup labels, dates, source footer and confirmation placeholders. Keep brand names, page copy quotations, URLs, event names and source names verbatim. Set `REPORT_LANG` to the matching BCP 47 language tag. The English strings in the references are structural examples, never mandatory copy. For a verified conversion, the dek uses a verified conversion rate and main finding; for a tracking diagnostic, it states the measurement gap without a rate. Run the final pass in `writing.md`. The chat reply uses the language of the user's message. Save a full audit as `<host>-conversion-audit.html` or a diagnostic as `<host>-tracking-diagnostic.html` and give the file.

Chat reply: three sentences at most. For a verified audit, give the conversion rate against the benchmark, the biggest supported loss and first fix. For a tracking diagnostic, give the central discrepancy and first instrumentation fix. Then the file.

## Limits

- Calls: the core calls in `data.md` (about 25) for a verified full audit; any further call only to settle a finding you are about to write, and no more than 45 in total. A tracking diagnostic stops once the discrepancy and repair check are evidenced; do not spend the rest of the full-audit call budget. Never loop calls over a list of pages, dates or countries; one grouped call answers that. All ConvRadar calls are read-only. Never call write tools (`cr_log_change`, `cr_record_verification`, `cr_mark_hypothesis_status`, `cr_save_ai_suggested`, `cr_update_change`).
- If a call returns a weekly limit, finish an honest partial report using verified findings and a plan; explicitly name the analyses left incomplete in the report and chat reply. Say that the full audit can continue when the allowance renews or the user has a plan with enough calls, without claiming a plan price or entitlement you have not checked. Do not present a partial report as a full audit or invent sections 01 and 02 when their data is unavailable.
- If the app interrupts a response before the file is built, use `references/resume.md` to continue from the next unfinished step. Do not automatically repeat calls already made or calls whose execution status is unknown.
- Ad accounts, heatmaps, CRM, email platform and phone outcomes are not in GA4. When a conclusion needs them, say which number would settle it, in one line, and do not estimate it.
