# How the report reads

A working document for the person who will change the site. Every sentence is a number, the cause of a number, or a decision. Anything else is deleted, not shortened.

## Sentences
- Number first, context after: "94 opened the form and 19 sent it", not "We can see that only a small share...".
- No narration: never "I found", "we checked", "looking at the data", "after reviewing".
- No labels on numbers: no "only", "just", "significant", "strong", "notably", "the majority". A verdict word is allowed once per outlier when it is a comparison with a stated benchmark ("below the 5 to 8% typical").
- No hedges on what the page shows. Hedge only the effect of a change.
- No sentence that announces what a section contains. No closing summary.
- Name things in full: the button text in quotes, the page path, the campaign name, the exact price.
- No em dashes in your own sentences. Ranges with "to": "35 to 50%". Names quoted from the site or the data (a campaign, a page title) keep their own punctuation.
- No rule-of-three phrasing inside sentences ("fast, simple and clear"), no "not just X but Y", no exclamation marks, no emoji. Sentence case headings. Lists of fixes or tests are as long as they need to be.
- Prose and tables; bullets only for "What to change".

## Numbers
- Period stated once in the header and footer, not on every number.
- Rates with their base where the base is small: "4 of 33 (12%)".
- Every number in the report comes from a call, the user, or arithmetic on those. No number from memory. The reference values written in this skill (5 to 8% product view to add to cart, 35 to 50% form completion, email converting 2 to 5 times the site) may be quoted as "typical"; nothing else without a source.
- Never subtract or divide across units (users against events, events against sessions) as if they were the same thing; state both and their ratio.
- Sessions per device come from `cr_get_device_breakdown`, not from event counts.
- Group totals add up to the total, or the remainder is stated ("top 60 countries; 61 sessions in the rest").

## Impact
- Use rates the site already reaches: another page, another device, a sibling flow, the same page last period. "Halfway to the rate desktop already reaches" is a good assumption; an industry multiplier is not.
- The vertical median or top quartile from `cr_compare_to_benchmark` is the second choice.
- A target taken from the best group (best country, best page) is only valid for a comparable group; otherwise call it an upper bound and give a lower one.
- Give ranges and the assumption in the same row. Cautious and full versions when the spread is wide.
- Fixes that act on the same people are not added together; say which one the total keeps.
- Under about 30 conversions in the window: counts, no money. Where nothing supports a number: "Not quantified" and the metric to watch.

## Mockups
- Rebuild the real block in HTML with its real headings, fields and prices, in the site's colours if known, then apply the changes. Pins carry the change number.
- "Now" beside "Proposed" for forms, booking steps, product decision blocks, cart.
- Copy in mockups is final copy from the site's own facts. `to confirm` only for a value the business must supply (a response time, a guarantee); never for a fact that can be read on the site or in the data. Each one listed once in the footer.

## Number check (before the final pass)
Make a list of every number in the report with its source (call, page, or the arithmetic and its inputs). Then:
- Recompute every impact line and every derived rate from its stated inputs with code; fix any that do not match.
- Remove any number taken from a source an earlier section called unreliable.
- A claim about why something changed (a drop, a rise) needs the same table for both windows; without the prior window it is written as a question, not a cause.
- A claim about time ("the last two weeks", "since August") is checked against the dated series.
- Every verdict word ("passes", "fails", "inside", "above", "below") matches the tool's own verdict line or the range it is compared with.
- A benchmark is compared only with the same metric (form completion = form starts to submits; not page-view ratios). If no like-for-like metric exists, say no benchmark applies.
- No impact line applies the site's rate to traffic the report called unqualified.
- A figure that appears more than once has one value, from one call; a total used in several places (sessions, leads) is fixed once at the top of the report and reused. Each rate names its base the first time and keeps it.
- No rate, headline or goal adds two conversion types together (bookings plus enquiries, orders plus calls).
- A rate takes its numerator and denominator from the same window and the same unit. When a section uses a shorter window (tracking start, a break), every number in it carries those dates, and nothing from that window is added to a number from another.
- Clicks counted as events are never read as "share of visitors who click"; without a distinct-clicker count, say "clicks per 100 views".
- The report avoids narration about how it was made. A partial report states which analyses remain incomplete; the footer lists data sources, dates and material measurement limits. Never hide an incomplete audit behind a full-audit title.
- An impact line counts only the increase: the new rate's result minus what the segment already produces.
- A page called missing or broken has its final status after redirects stated (a 301 to a page that loads is not a 404).
- Every "see section N" points to a section that exists with that number.
- Every "Not a problem" states the number it rests on; a metric the data cannot show is never cleared.
- A number from the web (a competitor price, a rating) is quoted with the page it came from and the date; one you did not read on a page this session is deleted.
- Any number without a source is deleted.

## Final pass
0. Count the words. Over the target: delete whole sentences, starting with the section that carries the least money, until it fits. Never shorten sentences instead.
1. Delete the first sentence of each paragraph and reread; if it still works, leave it deleted.
2. In a verified audit, every main problem has a count, a place and a supportable cost or the exact missing input. In a tracking diagnostic, show only observed measurement discrepancies, with no uplift or revenue claim.
3. Every "What to change" has an impact line or "Not quantified".
4. In a verified full audit, include "Not a problem" where a suspect was actually cleared, and a "Not on the list" line when evidence supports leaving work aside. Neither is required in a tracking diagnostic or a partial report.
5. Search the file for "—" and remove it.
6. Title and H1 name the report type in the report language; they match. A verified audit's dek has a verified conversion rate. A tracking diagnostic's dek states the measurement gap without a conversion rate.
7. Read the rendered text, not only the template source: title, H1, navigation, cards, tables, mockups, plan, date words and footer all use the site-page language. Keep literal page copy, brand names, URLs and event names as found. Set the `html` language tag to the audited locale. No English template labels remain in a non-English report.
