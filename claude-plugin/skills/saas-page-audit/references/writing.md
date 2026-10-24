# How the report reads

The report is a working document for the person who will change the page. Every sentence is a number, the cause of a number, or a decision. Anything else is deleted, not shortened.

## Length

- Whole report: about 1,000 to 1,800 words including mockup text. One screen per section where possible.
- Section 01: four numbers and at most one line under them.
- Section 02: a two-column table, Problem | Evidence. One problem per row; the problem in at most 8 words, the evidence in at most 25.
- Section 03: two tables and one line.
- Section 04: a three-column table #, Fix, Effort, one line under it about the pins, then mockups.
- Section 05: a three-column table and one paragraph of at most four sentences.

## Sentences

- Number first, context after: "4% of plan viewers start a trial", not "We can see that only a small share...".
- No narration: never "I found", "we checked", "looking at the data", "after reviewing".
- No labels on numbers: no "only", "just", "significant", "strong", "notably", "the majority".
- No hedges on what is on the page ("may", "might", "seems"). Hedge only the effect of a change.
- No sentence that announces what a section contains. No closing summary.
- Name things in full: the block's on-page heading in quotes, the exact price, the exact plan name.
- Every share names its base: "61% of those who reach the plans leave", not "61% of visitors leave".
- No em dashes. Ranges with "to" or a hyphen: "2 to 5 days", "9-17 h".
- No rule-of-three lists, no "not just X but Y", no exclamation marks, no emoji.
- Sentence case for headings. American spelling.

## Tables

- Numbers only in number columns. No "what to watch" or "why it matters" columns.
- Highlight (`tr.hi`) only rows that break trust or money: contradictions, surprise costs, broken promises.
- Mobile: tables scroll inside `.tw`; never wider than the page.

## Mockups

- Rebuild the real block in HTML in the site's own colours and fonts, with its real headings and prices, then apply the fixes. Keep every existing line of the block unless a numbered fix removes it. A generic wireframe is not a mockup.
- Pins carry the fix number. Every fix with a visible change has a pin somewhere.
- Copy in mockups is final copy, written from the page's own facts. Unknown values: `$X`, `to confirm`.
- "Now" beside "Proposed" only for the next step (checkout, form), where the difference is the point.

## Final pass

1. Delete the first sentence of each paragraph and reread; if it still works, leave it deleted.
2. Every problem row has a quote, a price, a count or a position.
3. Every fix number in the table has a pin or a mockup, and every pin has a fix.
4. Every `$X` and `to confirm` is listed in the footer.
5. No em dashes: search the file for "—".
6. The H1 is one sentence without a number; the title is `<Page name> Page Audit`.
7. No byline, no signature, no "prepared by", no company or product brand anywhere in the report. The masthead carries only the date; the footer carries only sources, dates and the values to confirm.
