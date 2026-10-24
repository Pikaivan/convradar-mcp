# Reading the pages

## Screenshots first, for the pages that carry the report
`cr_capture_screenshots(url, page_type)` then, about 20 seconds later, `cr_get_screenshots(request_id)` (poll again if it says capturing). It renders the page in a real browser on a phone and a desktop, scripts included. Take it for three to five pages: the homepage, the main money page (or the top product page for a store), the page with the form, the booking app or the cart, and the page behind the biggest loss if it is another one. From the images report: what the first phone screen shows and whether it has an action; on which screen the first button, the form, the price and the proof appear ("the first booking button is on screen 5 of 14"); anything covering the page (cookie banner, chat bubble, pop-up); whether what looks tappable is tappable. Use positions in screens only from screenshots you saw.

First phone screen checklist, for each screenshot set: the main action is a filled, high-contrast button (not text over a video or an outline); an action stays reachable when the header scrolls away (sticky bar or header); the whole card or banner is tappable where it looks tappable; nothing covers the action (cookie banner, chat bubble, pop-up); proof is named (a person, a company, an outlet) rather than anonymous stars or one-word quotes; on a form, the send button is visible without scrolling and the number of fields matches what the page promised.

## HTML for everything else
Use the browser if the chat has one. Otherwise fetch the HTML: `cr_capture_via_web_fetch(capture_set_id="CS-homepage" | "CS-landing-page" | "CS-pdp-mobile" | "CS-cart" | "CS-checkout-flow", url)`, or the chat's own web fetch. HTML shows text, links, forms and scripts in page order; it does not show what fits on a phone screen, so give positions as order and share of page length ("the first button sits after 1,200 words"), never as screens you did not see.

## On every page you read
- First action: the first button or form in page order, its text, and what it opens (a link, a pop-up id, an external app).
- Calls to action: every button that leads to the conversion, its text, its target. Several different promises opening the same generic form is a finding.
- Forms: fields (required ones), heading, the line about what happens next, consent text, autocomplete attributes on email and phone inputs, anything that looks like an error before typing.
- Before the step where the visitor pays or commits, check each and say where it appears, if at all: the total for the order or group (not only a per-person or per-unit price), fees, the amount due now, refund terms, reschedule or cancellation terms, payment methods, what happens right after paying. Terms that exist only in the footer or on a separate page count as missing from the flow.
- Proof: ratings, badges, client logos, named reviews; their position in the page.
- Tags in the HTML: `fbq(` or `connect.facebook.net` (Meta pixel), `analytics.tiktok.com` (TikTok), `AW-` in gtag config (Google Ads), `bat.bing.com` (Microsoft UET), Klaviyo or other email capture.
- Message match: for every paid campaign in the campaign table with sessions and few or no conversions, the campaign's name words against the landing page's title and H1; a phrase the campaign buys that the page never says (or says once deep in the page) is a finding.

## Same fact, every page
Prices, fees, delivery, duties, returns, warranty, minimums and terms often appear on several pages (product page, policy page, FAQ, checkout, booking app). Read each place and quote any two that disagree; a contradiction on money or terms is a main-problem candidate.

## Pages built by script
When the HTML holds no content (a booking app, a cart rendered in the browser), read the page's script bundle for its visible strings (headings, field labels, button texts, step names) and say that the order of screens is inferred. Do not claim screens you did not see.

## Lead generation
- The form or booking flow step by step, as far as HTML or the browser shows: what each step asks, where the email or phone is asked, what comes before it.
- The thank-you page: what the visitor is told happens next.
- Content pages with traffic and no leads: is there any link or button toward the service.

## Stores
- Homepage: categories on the first screen, what the hero links to.
- Category page: filters, sort, product cards (price, stock, rating), number of products.
- Product page: price, stock, delivery, returns, warranty near the add-to-cart button; number of add-to-cart buttons; reviews; media; description structure; compatibility or size information.
- Add to cart: form action and the theme's cart behaviour (Shopify `cart-drawer` or `/cart/add`, Magento `minicart`, WooCommerce `added_to_cart`); what follows the click on each page type.
- Search: fetch `<site>/search?q=<top product type>` (or the site's own search URL from the HTML) and one misspelled query; count results; read the empty-results page.
- Cart and checkout: fetch the cart page; note express payment buttons, shipping protection or add-ons, delivery and returns lines. Never enter payment details or submit a checkout.

## Competitor price check (stores)
For the one or two products with the most views and a weak add-to-cart rate, search the product name and compare the price with two other sellers and the brand's own site. Report the prices and the dates.
