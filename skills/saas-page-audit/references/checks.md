# Checks for one SaaS or service page

Work through all of them. Each produces a problem row with evidence, a fix, or nothing. Call a part of the page competent only when no check below flags it.

## 1. Block map and placement

List the page's blocks top to bottom with their on-page headings and where each starts as a share of the visible text, counted from the H1 to the footer and rounded to 5%. If the user gave a heatmap, use its page-height scale for every position instead and say so in the footer. Mark:
- the hero's main promise (who it is for, what makes it different);
- the decision block: plans and prices, the offer, or the form where the visitor commits;
- the blocks that answer the top buyer questions (check 7) and the block that proves the hero's promise;
- the human path: call, demo, consultation, chat;
- whether a contents bar or jump links exist.

Problem rows this produces:
- **Buried answer.** Match each top-3 buyer question (check 7) and the hero's promise to the first place that answers it: a heading, a comparison-table row or a card line. A one-line answer higher up turns the row into "full answer after the plans". A header-menu or hero link that jumps to the block counts as a path to it, and a form in the hero does not make every later block buried. That block starts deeper than 40% of the page, or after the decision block, and no contents bar leads to it. Evidence: its heading, its position, and the heatmap share that never gets there if known. Fix: move that existing block up, right after the hero, and add a contents bar under the hero. Both get a mockup.
- **Human path out of reach.** The call, demo or consultation block sits after the decision block and nothing inside the decision block points to it. A sticky or floating contact bar, a header link or a chat widget does not count as a link inside the decision block. Visitors who leave at the decision block never see it, with or without click data. Fix: one line above the plans linking to it, and a talk-to-us button on any plan without a self-serve start; the existing block stays.
- **Decision block far down.** It starts below 60% of a long page and the hero has no jump link to it.

## 2. Claim inventory

Make one table of every factual claim and every place it appears: hero, cards, comparison or specs table, FAQ, footnotes, badges, the next step. Collect:
- each price per term (monthly, yearly, multi-year), with and without discount;
- plan names;
- limits and specs per plan (seats, users, storage, projects, requests);
- support and response promises (hours, channels, response time, uptime);
- trial, guarantee, contract, cancellation and refund terms;
- fees, taxes, setup costs, and any "no extra fees" statement;
- delivery or setup time;
- counts and years (customers, years in business);
- statements about what the company is or does in-house.

Any claim with two values is a problem. A qualifier is not a second value ("Min. 100" and "100 per design" say one thing). Price, plan name, support promise, contract and trial terms each get their own row: "Cards: $49 / $99. Comparison table: $45 / $89." Smaller mismatches (badge wording, typos, phone numbers, a wrong cell in a comparison table, backup or spec details) go into fix 1's list, not into rows.

Terms of service and legal pages: read them only to confirm or contradict the page's claims on price, renewal and cancellation. They never become a row or a fix on their own; a difference goes to the footer as "to confirm".

## 3. The next step

Follow every main call to action. For each target (checkout, signup, trial form, booking, contact form) record:
- the price and term shown, and every line added to it (fees, tax, setup, add-ons);
- any cost, condition or required field that appears there for the first time;
- what the button promised against what the step asks;
- when billing starts and when the trial ends;
- the plan and term the step preselects against the one clicked.

A cost first seen at the next step, or one that contradicts a page claim, is a top row with the arithmetic: "$49 on the card, $58 at checkout with a setup line". If the target is behind a login or cannot be fetched, say so and do not guess its content. Before saying a step drops or keeps the visitor's choice, read the URL and the form's redirect or next parameter: a choice carried but not shown is "not shown", never "dropped".

When the next step is a form (signup, trial, quote, booking): list the required fields, what the button that led here promised against what the form asks, anything that must happen before the visitor can use the product (email or phone code, approval, a call), and where the submit button sits on a phone and a laptop screen. Form event totals alone cannot show a drop between people; use joined visit or user counts for a quantitative form finding.

## 4. Clocks

Compare every clock the offer starts (trial days, money-back days, intro months) with how long the customer waits for first value (setup, build, onboarding, approval, migration), from the page, the FAQ or the next step. A trial that can end before the product is usable is a row: "14-day trial from signup; onboarding takes 2 to 3 weeks".

## 5. Units and extras

This check is for plan cards with units (seats, credits, limits, terms), not for price tables of physical goods. Each line on a plan card must be readable without a sales call. Flag units with no definition ("seats", "credits", "events", "users", "projects"), paid extras or overage prices that exist somewhere but not on the card, and spec lines with no explanation, link or tooltip. The fix states what one unit is (per person, per login, per month) and the price of the next one, and gives each flagged line one sentence of explanation taken from the page's own specs and FAQ. Evidence: the line as written and where its price or definition appears ("nowhere before checkout"). Never write an extra or overage price, even as `$X`, unless the page or the next step shows that the extra is sold. With heatmap clicks, add the click count on those lines.

## 6. Decision block mechanics

- Button text per plan; a plan that routes to sales while others are self-serve, and whether the page says why.
- The billing switch default and whether the shown price is the one charged first. Leave the default alone if the user says it is being tested.
- Whether cards repeat the same lines in the same order so plans can be compared.

## 7. Buyer questions

Search for about 20 public discussions from the last 24 months where people choose this kind of product (Reddit, Hacker News, niche forums, review sites; not Google Maps). At most 3 of the 20 may be reviews of the audited company itself. Count only discussions you actually read, not search snippets. Code each discussion for these question types, then merge and count:
- total cost for their size;
- who is responsible for what;
- how the people who use it every day get in and work (access, tools, workflow);
- time to go live and onboarding;
- fit with what they already use;
- contract and lock-in;
- proof they can show their own customers or auditors.

Keep the 3 or 4 most raised. Row label is the buyer's question in plain words with the count ("8 / 20"). Page column: how the page answers it and where ("At 20%, 'How it works'", "Not answered", "FAQ only"). Counts are discussions, not market share; say so in the footer.

## 8. Competitors

The 3 nearest competitors a buyer compares in the same search. Columns: this company plus three. Rows: entry price with its term and what it includes, how you start (self-serve, trial, contact sales). Before writing "not published", read the competitor's pricing page and FAQ in full. Compare like with like before ranking (managed with managed, self-serve with self-serve). One line under the table, at most 15 words, on where this page stands, checked against every cell.

## 9. Numbers worth a row

From `data.md`: locate a drop between visits, decision block, next step and conversion only when these counts follow the same visits or users. Form events are event counts, not unique people; repeated starts, submit attempts and automatic GA4 events do not by themselves locate a drop. If the path cannot be joined, name the unmeasured step and make the measurement fix. Device or source gaps only on 30+ visits per segment. Unqualified traffic goes into section 01, not into rows.

## 10. Speed

When quota use is allowed and the call budget permits, make one `cr_check_page_speed` call on the page or the next step selected in step 1b, mobile. A row only if the decision block or form is not usable for 4+ seconds on mobile; then the evidence is the time. If it passes, it is not mentioned. If the call fails, skip it.

## Measurement

If the decision block's buttons, the billing switch or the next step are not tracked as events in this GA4 property (check the event names from `data.md` call 3), the last fix adds those events, even when the user pasted numbers from elsewhere. Carry page variant, plan and term through the next step and deduplicate success by account or order ID. A success page that shows up as a landing page does not prove conversions went uncounted; investigate the path before claiming a tracking failure.

## Uplift and A/B line

- The moved rate is the one at the step where visitors stop. Target: a rate another page, plan or segment on the same site already reaches, or a stated modest step labelled as an assumption (a third more on a rate under 10%, a fifth more on a higher one).
- Extra conversions per quarter = quarterly volume in the same joined cohort at that step × (target − current) × later measured cohort rates, which stay as they are. If the path or variant cannot be joined, do not calculate extra conversions.
- First-year value = extra paid conversions × entry monthly price × 12, or × deal value for services. A free signup or trial has no assumed paid value. Round to two significant figures with "~".
- A/B feasibility: visitors per month reaching the changed block. Under ~1,000 a month for a rate under 10%, a test cannot detect a 30% lift in 8 weeks: write that there is too little traffic for an A/B test and give the before/after comparison instead (ship together, compare 8 weeks before and after on the named rate and its current value).
