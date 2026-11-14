# Amazon US order and invoice pages

Research for [`amazon-pages-brief.md`](amazon-pages-brief.md). Written 2026-11-14. Evidence is other projects' code, issue trackers and commit history. We could not log in to Amazon, so nothing here was checked against a live account. Anything not confirmed in code is marked **(unverified)**.

## Sources read

Each was read at the commit shown. Downloaded code was read, never run.

| Short name | Repo and commit | License | Last commit | Notes |
|---|---|---|---|---|
| azad | [philipmulcahy/azad @ 87d90af](https://github.com/philipmulcahy/azad/tree/87d90afa8ee648ec48b040e338275ba8797ae0d4) | Apache-2.0 | 2026-08-18 | Chrome extension, content script on amazon pages |
| amazon-orders | [alexdlaird/amazon-orders @ 4d3e8e7](https://github.com/alexdlaird/amazon-orders/tree/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc) | MIT | 2026-11-11 | Python, `requests` based. Most current evidence; not in the brief's list but the best-maintained source |
| amazon-auth | [alexdlaird/amazon-auth @ 275d0bf](https://github.com/alexdlaird/amazon-auth/tree/275d0bf) | MIT (assumed, same author) | 2026-11-11 | Session and block detection used by amazon-orders |
| monarch-amazon-sync | [alex-peck/monarch-amazon-sync @ a271922](https://github.com/alex-peck/monarch-amazon-sync/tree/a2719228130dd2b244331bd7039160873ab4e2c0) | MIT | 2024-10-29 | MV3 service worker plus alarms. Stale: two years old, and Monarch now ships an official add-on ([issue 43](https://github.com/alex-peck/monarch-amazon-sync/issues/43)) |
| order-history-exporter-for-amazon | [Xenolphthalein/order-history-exporter-for-amazon @ 39142ed](https://github.com/Xenolphthalein/order-history-exporter-for-amazon/tree/39142ed3a969c55846f12a172eea83d5720dc080) | Unlicense (public domain text) | 2026-10-22 | MV3 extension, content script, uses the invoice popover |
| amazon-order-exporter | [bilal-psd/amazon-order-exporter @ 0d872e7](https://github.com/bilal-psd/amazon-order-exporter/tree/0d872e7ac5afceff21b8b22f8014a1468f106649) | see repo | 2026-03-08 | MV3, drives a background tab |
| amazon-downloader-extension | [oliverswitzer/amazon-downloader-extension @ b1fb67b](https://github.com/oliverswitzer/amazon-downloader-extension/tree/b1fb67bd) | see repo | 2023-10-03 | Abandoned. Only evidence: it fetched the card's `b_invoice` link |
| Playwright scrapers | [dcwangmit01 @ 969f9f1](https://github.com/dcwangmit01/amazon-invoice-downloader/tree/969f9f1e4833c974c50a25ba788d5c42a55cd49c), [chrisns @ 6a6f644](https://github.com/chrisns/amazon-invoice-downloader/tree/6a6f64491c3f6b5b16a89dd6659a8227cb63d2b7) | MIT | 2025-08 / 2026-11 | Print the invoice page to PDF; chrisns targets amazon.co.uk |

Not reviewed: ayelder's scraper (repo not found under that name) and Docudigger (archived 2026-03-07).

## Headline finding: the order list is not reliably plain server HTML

Three separate signals say a bare `fetch()` plus `DOMParser` of the order list is the riskiest part of the design:

1. **Client-side-encrypted cards ("CSD").** Amazon sometimes serves `div.order-card` shells whose contents are an encrypted payload plus a `<noscript>` fallback naming `disableCsd`. amazon-orders detects this (`div.csd-encrypted-sensitive` present and the order number unreadable) and re-requests the same URL once with `disableCsd=no-js`, which renders readable cards. Their report: the no-JS form "has rendered readable on every encrypted page we have hit", while "the plain fetch encrypts intermittently" ([issue 164](https://github.com/alexdlaird/amazon-orders/issues/164), [orders.py `_fetch_orders_page`](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L861), [constants.py L52-53](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L52)). Monarch's sync hard-coded `disableCsd=no-js` back in 2024 ([amazonApi.ts L8](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/src/shared/api/amazonApi.ts#L8)), so this is not new.
2. **Skeleton shell.** One reporter on amazon.com got a ~660 KB page with only loading placeholders and no cards ([issue 134](https://github.com/alexdlaird/amazon-orders/issues/134)). They later retracted the diagnosis: the account had no orders and was an Amazon Business account. So the skeleton is real for some layouts (Business) but is not shown to affect ordinary consumer accounts. **(unverified for consumer accounts)**
3. **azad renders the list in a hidden iframe**, waiting for `.yohtmlc-order-id`, rather than reading the raw response ([order_list_page.ts L35-L46](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_list_page.ts#L35)). Detail and invoice pages use plain XHR.

Design implication: `listOrders()` needs (a) a CSD-encrypted check with a one-shot `disableCsd=no-js` retry, (b) a "skeleton or empty" state that is reported as a layout failure, not as "zero orders", and (c) a documented fallback of loading the list in a real tab/iframe if plain fetch stops working. See the recommendation.

## 1. Order list

**URL.** The current list page on amazon.com is `https://www.amazon.com/your-orders/orders`.

- azad: `https://%(site)s/your-orders/orders?timeFilter=year-%(year)s&startIndex=%(startOrderPos)s&language=en_US` ([order_list_page.ts L184-L205](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_list_page.ts#L184)).
- amazon-orders: same path, `timeFilter`, plus `page` and `startIndex` ([constants.py L44-L62](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L44), [`_order_history_url`](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L613)). Page size is 10 (`ORDER_HISTORY_PAGE_SIZE`). It adds `page=N&startIndex=10*N` for N > 0.
- Legacy `/gp/css/order-history` and `/gp/your-account/order-history` still work as entry points (Monarch and the Playwright scrapers use them; amazon-orders issue 134 notes they return the same page as the new URL). Prefer the `/your-orders/orders` path.

**Time filter values** (all confirmed in code): `last30`, `months-3`, `year-YYYY` ([constants.py L56-L58](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L56), [`_history_filter_value`](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L587)). bilal's exporter uses `last-30` (hyphen) in one place ([service-worker.js L116](https://github.com/bilal-psd/amazon-order-exporter/blob/0d872e7ac5afceff21b8b22f8014a1468f106649/background/service-worker.js#L116)), which conflicts with amazon-orders; trust `last30`. **(unverified live)**. The year options are discoverable from `select#time-filter option` on the page ([fixture](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/orders/order-history-2026-220.html)). That page also states the count: `<b>213 orders</b> placed in`.

**Paging.** `startIndex` in steps of 10, and/or `page=N` (zero-based in amazon-orders). A user report from August 2026 shows the Next link as `?page=1&ref_=ppx_yo2ov_dt_b_pagination_1_2` ([azad issue 416](https://github.com/philipmulcahy/azad/issues/416)), so the page parameter is what the live Next link uses today. Safest: follow the page's own Next link (`ul.a-pagination li.a-last a`), as bilal's exporter does ([scrape-orders.js L145-L155](https://github.com/bilal-psd/amazon-order-exporter/blob/0d872e7ac5afceff21b8b22f8014a1468f106649/content/scrape-orders.js#L145)). Stop on an empty page. Do not rely on a total count: Amazon removed `.num-orders` from some responses in August 2026 (azad issue 416, fixed in [71fe212](https://github.com/philipmulcahy/azad/commit/71fe212), 2026-08-18). amazon-orders still reads `span.num-orders` as an optional gate ([selectors.py L67](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L67)).

**For a daily sync** use `timeFilter=last30` (or `months-3` for slack) and page until a page is empty or all orders on it predate the window. Orders are newest first. Use `year-YYYY` only for backfill.

**Order card (selectors and fields).** Confirmed from two independent parsers and the amazon-orders fixtures:

| Field | Selector (current first) | Source |
|---|---|---|
| Card | `div.order-card` (also `.js-order-card`, older `div.order`) | [selectors.py L64](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L64), azad [xpaths.ts](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/xpaths.ts) (updated 2026) |
| Order ID | `[data-component='orderId']`, `.yohtmlc-order-id [dir=ltr]`; or parse `orderID=` from any link in the card | [selectors.py L179](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L179), [order_header.ts](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_header.ts) |
| Date | label "Order placed" then the value; azad has three strategies and a 2026-08-16 change to `li.order-header__header-list-item` | order_header.ts (`strategy2`, "as of 2026/08/16") |
| Total | `div.yohtmlc-order-total span.value`, or the "Total" column of the header | [selectors.py L184](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L184) |
| Details link | `a.yohtmlc-order-details-link` | selectors.py L171 |
| Invoice link | "View invoice" link in the card, `a[href*='b_invoice']` or `.../gp/css/summary/print.html?orderID=…&ref=ppx_yo2ov_dt_b_invoice` | [fixture](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/orders/order-history-2024-0.html), oliverswitzer [main.js L8](https://github.com/oliverswitzer/amazon-downloader-extension/blob/b1fb67bd/main.js#L8) |
| Status | `.yohtmlc-shipment-status-primaryText`, `span.delivery-box__primary-text` (text such as "Delivered November 2", "Cancelled") | [selectors.py L255](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L255) |
| Ship to | present on physical orders, absent on gift-card and digital cards | fixtures |

A cancelled card has no total or "ship to": the fixture shows only date, order number and "Your order was cancelled. You have not been charged for this order." ([order-history-canceled-order.html](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/orders/order-history-canceled-order.html)). Ship-to names and addresses appear in the card, so fixtures need scrubbing.

**Logged-out detection.** Reliable signals, in order:

1. The response was redirected and `response.url` contains `/ap/signin` or `signin` (azad [signin.ts](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/signin.ts), [request.ts L51](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/request.ts#L51); amazon-auth tests the URL prefix).
2. The body contains `form[name='signIn']` ([amazon-auth selectors.py L81](https://github.com/alexdlaird/amazon-auth/blob/275d0bf/amazonauth/selectors.py#L81)).
3. Weak: Monarch looks for `h1:contains("Sign in")` ([amazonApi.ts L47](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/src/shared/api/amazonApi.ts#L47)); users on other TLDs got false "auth failed" with it ([issue 31](https://github.com/alex-peck/monarch-amazon-sync/issues/31), a .ca account).

Beware "partial logout": Amazon can log a user out of some page families (e.g. digital) while others still work ([azad issue 411](https://github.com/philipmulcahy/azad/issues/411), maintainer's explanation). Per-page-type failures should not all read as "logged out". Redirects to anything else must not be reported as "order not found" ([amazon-orders issue 137](https://github.com/alexdlaird/amazon-orders/issues/137), fixed in 4.7.1 by recording the redirect URL). Keep the final URL in the error.

## 2. Invoice page

There are three candidate pages. Sources disagree on which carries what.

### a. Printable Order Summary: `/gp/css/summary/print.html?orderID=…`

- Still the URL in use. amazon-orders (`ORDER_INVOICE_URL`, [constants.py L46](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L46)), azad (`getOrderPaymentUrl`, [url.ts L73](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/url.ts#L73)), Monarch ([L11](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/src/shared/api/amazonApi.ts#L11)), and the cards' own "View invoice" links all use it.
- amazon-orders added a `get_invoice()` for it on 2026-06-07 ([issue 86](https://github.com/alexdlaird/amazon-orders/issues/86), the requester calls it a "single, stable HTML format").
- **Layout changed.** Monarch (2024) parsed an old table layout (`td b:contains("Order Placed:")`, "Credit Card transactions", `#pos_view_section`). The 2026 fixture is the same component layout as the order-details page. azad's older payment code still has the table-era strategies ([payment.ts strategy_1/2](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/payment.ts)). So the old parsers are obsolete for new pages, but may still be needed for old orders **(unverified)**.
- **2026 layout** ([get-invoice.html fixture](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/invoices/get-invoice.html), sanitized by a contributor; server-rendered, no JS needed to see these values): `div#orderDetails` containing `data-component` blocks: `orderDate`, `orderId`, `shippingAddress`, `viewPaymentPlanSummaryWidget` (payment method, "ending in NNNN"), `chargeSummary` (`div.od-line-item-row` rows), `shipments` > `shipmentStatus` (`h4.od-status-message`), `purchasedItems`, `itemTitle`, `orderedMerchant` ("Sold by"), `supplierOfRecord`, `quantity`, `unitPrice`, `deliveryFrequency`, `cancelled`. It contains a print button and no site header, which is why it renders cleanly.

### b. Invoice popover: `/your-orders/invoice/popover?orderId=…`

- An AJAX HTML fragment (`ul.invoice-list`) listing the order's documents: "Printable Order Summary" (links to `print.html` above), seller-issued PDFs under `/documents/download/<uuid>/invoice.pdf` ("Invoice 1", "Credit note"), and "Request Invoice" ([fixture](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/invoices/invoice-pdfs-multiple.html)). Added to amazon-orders 2026-10-19 ([commit e518e6f](https://github.com/alexdlaird/amazon-orders/commit/e518e6f)).
- The PDFs are mostly issued on EU sites; for a typical US order the list holds only the Printable Order Summary ([invoice-pdfs-none.html](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/invoices/invoice-pdfs-none.html)). **US PDF availability is unverified.**
- Works without the page-scoped `relatedRequestId` token when session cookies are valid ([exporter invoiceUtils.ts](https://github.com/Xenolphthalein/order-history-exporter-for-amazon/blob/39142ed3a969c55846f12a172eea83d5720dc080/src/utils/invoiceUtils.ts)).
- Returns an error status (400 or 5xx) or redirect for orders with no list (e.g. Whole Foods in-store); amazon-orders then falls back to `print.html` directly ([orders.py L367-L396](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L367)).
- Download links are one-time: "Amazon generates a new one on each request, so don't store it" ([invoice.py](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/entity/invoice.py)).
- chrisns used a third path, `/gp/shared-cs/ajax/invoice/invoice.html?orderId=…&relatedRequestId=NEEDED`, on amazon.co.uk to reach PDF links ([cli/__init__.py L153](https://github.com/chrisns/amazon-invoice-downloader/blob/6a6f64491c3f6b5b16a89dd6659a8227cb63d2b7/src/amazon_invoice_downloader/cli/__init__.py#L153)). It is the pre-popover predecessor. Skip it.

### c. Order details: `/gp/your-account/order-details?orderID=…` (also `/your-orders/order-details`)

- What amazon-orders and azad parse for items, totals, shipments and statuses ([constants.py L45](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L45), azad `getDefaultOrderDetailUrl`). azad takes everything except payment from here and uses the print page only for payments ([order_details.ts](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_details.ts), [payment.ts `fetch_payments`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/payment.ts)).
- **Retention gap.** One reporter found `get_order()` fails for orders older than "roughly 30 days" on their account while the payment feed still lists them ([issue 137](https://github.com/alexdlaird/amazon-orders/issues/137), [139](https://github.com/alexdlaird/amazon-orders/issues/139)). The boundary is unconfirmed and may be account-specific. No other source mentions it, and azad scrapes multi-year histories. **(unverified; matters for backfill)** Because we only sync recently shipped orders, it probably does not affect the daily path, but a backfill must treat a redirect off the detail URL as "not retrievable", not an error to retry.

### What the parsers read, and how stable

| Field | Where | Notes |
|---|---|---|
| Order ID, date | all three | Stable in every parser. |
| Items: name, qty, unit price | details `purchasedItems`/`itemTitle`/`quantity`/`unitPrice`; print page has the same components | Table-era print page parsed by index (`td` position); fragile. |
| Seller | `orderedMerchant` ("Sold by:"), `supplierOfRecord` ("Supplied by") | [Seller entity](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L286) |
| Subtotal, shipping, promotions, tax, grand total | `chargeSummary > div.od-line-item-row` matched by label text | Labels vary: "Grand Total" vs "Total for this Order" (digital); "Estimated tax to be collected" vs "Tax Collected" (digital); also "Total before tax", "Shipping & Handling", "Promotion applied", "Subscribe & Save", "Gift Card Amount" ([selectors.py L218-L235](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L218), [issue 108](https://github.com/alexdlaird/amazon-orders/issues/108)). Match by label text, never by position. |
| Payment method: card type, last 4 | `[data-testid='payment-instrument-name']` and `-number`; regex `(?:ending in\s+\|•+\s*)(\d+)`; Whole Foods: `#wfm-0-card-brand`, `#wfm-0-card-tail` | [selectors.py L204-L212](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L204). **Fragile**: see below. Keep last 4 as a string. |
| Shipped date(s), per-shipment grouping | details `shipments` blocks (`div.shipment`, `[data-component='orderCard'] [data-component='shipments'] .a-box`) with `od-status-message` text and tracking links | The print page also has `shipmentStatus`. |
| Recipient / address | `shippingAddress` | Personal data. Do not store; the design stores card type and last 4 only. |

**Payment parsing is the weakest field.** azad has had repeated breakage: "Unknown;" in the payment column in June 2025 ([341](https://github.com/philipmulcahy/azad/issues/341)) and October 2025 ([367](https://github.com/philipmulcahy/azad/issues/367)), "obfuscated html" in payment sections on the detail page ([384](https://github.com/philipmulcahy/azad/issues/384)), gift-card payment missing ([386](https://github.com/philipmulcahy/azad/issues/386), fixed), and blank payments again on 2026-11-02 ([422](https://github.com/philipmulcahy/azad/issues/422), open). The maintainer wrote that "the information we seek is on the pages the extension is looking at, but it's not managing to find it" and, in 367, that the fix might need "cooked" (post-JavaScript) HTML. Whole Foods and Fresh orders have no card details on the detail or invoice pages (issue 367). Plan: payment method is optional in `Order`; the PDF shows it when found and omits it otherwise; never fail a sync on it.

## 3. Order types

Same invoice URL for all? **Mostly yes for `print.html`, with exceptions.**

| Type | Order ID | List page | Invoice and detail | Notes |
|---|---|---|---|---|
| Standard physical | `NNN-NNNNNNN-NNNNNNN` | normal card | `print.html` and `order-details` | Baseline. |
| Multi-shipment | same | one card, several shipments (`div.shipment`) | one invoice; per-shipment items/status on the detail page | amazon-orders had a bug where multi-item shipments rendered with no items on the list ([110](https://github.com/alexdlaird/amazon-orders/issues/110), [124](https://github.com/alexdlaird/amazon-orders/issues/124), fixed). Payments can be several transactions, one per shipment (azad `get_transactions`; Monarch's design was built around this). |
| Subscribe & Save | normal | card appears on charge date | normal; discount line "Subscribe & Save:" ([order_details.ts](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_details.ts)); item `subscription_frequency` added 2026-11-10 | azad issue 409 (closed, "not reported as such") shows it was not parsed for a while. Pending upcoming S&S deliveries appear on the list as orders that have not shipped. |
| Digital (Kindle, apps, video, music) | `D01-NNNNNNN-NNNNNNN` | listed under the "digital" order filter: `/gp/legacy/order-history?...digitalOrders=1` or `?orderFilter=digital` | **different**: `/gp/digital/your-account/order-summary.html?orderID=D01-…&print=1` (azad `getOrderPaymentUrl`; [item.ts L443](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/item.ts#L443)). amazon-orders reads `order-details?orderID=D01-…` with the same layout but different labels (issue 108, fixed). | azad code skips fetching payments for `D…` IDs and builds a string from the list total instead. Which digital URL still works today is **unverified**. Digital orders have no shipment, so "shipped" never applies. |
| Whole Foods / Fresh | normal-looking IDs; Whole Foods in-store receipts use `/wholefoodsmarket/receipts/order/` and `/fopo/order-details` | card exists | **not the standard page**: dedicated receipt route ([constants.py L60](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L60)); invoice popover errors out ([orders.py L379-L391](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L379)); items often unparseable (`order-details-wholefoods-unparseable.html`; azad [407](https://github.com/philipmulcahy/azad/issues/407), [367](https://github.com/philipmulcahy/azad/issues/367)). | amazon-orders skips item parsing for these and keeps only total and count. |
| Gift-card purchase (eGift sent) | normal | card without ship-to; "Sent" status, amount, recipient email | invoice exists ([fixture](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources/orders/order-history-egift.html)); the card contains a recipient email address (scrub in fixtures) | Not "shipped"; treat "Sent" as complete. **(unverified)** |
| Gift-card *payment* | any | n/a | A gift-card-paid order shows a "Gift Card Amount" line and maybe $0 grand total, and no card ([azad 386](https://github.com/philipmulcahy/azad/issues/386), [amazon-orders 152](https://github.com/alexdlaird/amazon-orders/issues/152)). | Payment is "Amazon Gift Card". |
| Refund / return | same order | card shows "Return complete" / "Refunded" | Refund total line on detail page; shipment "Refund for this return"; Monarch fetched `/spr/returns/cart?orderID=…` for "refund issued on" text ([L146-L170](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/src/shared/api/amazonApi.ts)) | A refund changes the order after we archived it. v1 should archive the original invoice and not try to track refunds. |
| Cancelled | same | status "Cancelled"; no total | totals missing; `h4.a-alert-heading` or `od-status-message` with "Cancelled" ([selectors.py L108-L120](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L108), [issue 132](https://github.com/alexdlaird/amazon-orders/issues/132)) | Skip (dcwangmit01 also skips them: [L257](https://github.com/dcwangmit01/amazon-invoice-downloader/blob/969f9f1e4833c974c50a25ba788d5c42a55cd49c/src/amazon_invoice_downloader/cli/__init__.py)). |

**Order-ID formats** (amazon-orders `ORDER_NUMBER_REGEX`, [constants.py L117](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/constants.py#L117)): `[A-Z0-9]{3}-\d{7}-\d{7}` (physical `111-…`, `112-…`, `113-…`, `114-…`; digital `D01-…`; Fresh/other three-char prefixes), older `\d{4}-\d{6}-\d{7}`, and 19-digit IDs. Our ID validator must be loose; do not hard-code `\d{3}`. An amazon-orders issue shows a transaction-page regex that missed `D01-` IDs silently ([130](https://github.com/alexdlaird/amazon-orders/issues/130)).

## 4. Shipped status

No parser exposes a single "shipped" flag. They read text and a tracking link.

- **List card:** the status line (`.yohtmlc-shipment-status-primaryText`, `span.delivery-box__primary-text`): "Delivered <date>", "Arriving <date>", "Shipped", "Cancelled". There is one per shipment.
- **Detail / print page:** `h4.od-status-message` in `shipmentStatus` (the fixture shows "Arriving Monday" for a shipment still in transit) and a tracking link: `a[href*='/progress-tracker/package?']` or `a[href*='ship-track?itemId=']` ([selectors.py L248-L254](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/selectors.py#L248)). A not-yet-shipped shipment has a `/progress-tracker/package/preship/cancel-items` link instead, which must not be mistaken for tracking.
- **azad** classifies `Delivered.YES` from the class `shipment-is-delivered` or the text "delivered", `NO` from "arriving", `RETURNED` from "return complete", else `UNKNOWN` ([shipment.ts `is_delivered`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/shipment.ts)).
- **Status text disappears:** the delivery status becomes `None` "some weeks after delivery", and the tracking link goes missing on older orders ([shipment.py](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/entity/shipment.py)). A missing status therefore means "old", not "unshipped".
- A charge usually happens at ship time. In amazon-orders' fixtures an in-transit order's print page shows the grand total and a card already. Whether the total can still change before shipping is **unverified**.

Rule for v1: an order is ready when every non-cancelled shipment's status starts with "Delivered", "Shipped" or "Out for delivery", or its status is missing and the order is older than ~30 days. Anything containing "Arriving", "Not yet shipped", "Preparing" or "Processing" waits. Unknown statuses wait and log, and the failure UX (layout changed) fires if no order on a page yields any recognized status. Exact wording must be captured from fixtures; **no source lists the full set of status strings.**

## 5. Rate limits and blocks

**What tools do.**

| Tool | Concurrency / delay |
|---|---|
| azad | Up to 6 concurrent XHRs ([request_scheduler.ts L79](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/request_scheduler.ts#L79)), 4 list pages at once ([order_list_page.ts L114](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_list_page.ts#L114)), no delay, 20 s timeout. It makes many extra requests per order (tracking page per shipment, product page for categories). |
| Monarch | `promise-parallel-throttle` defaults; no explicit delay. |
| bilal | 1.5 s between navigations ([L6](https://github.com/bilal-psd/amazon-order-exporter/blob/0d872e7ac5afceff21b8b22f8014a1468f106649/background/service-worker.js#L6)), retries, a dedicated background tab. |
| Playwright scrapers | random 2 to 5 s "human latency" between actions ([dcwangmit01 L89](https://github.com/dcwangmit01/amazon-invoice-downloader/blob/969f9f1e4833c974c50a25ba788d5c42a55cd49c/src/amazon_invoice_downloader/cli/__init__.py#L89)). |
| amazon-orders | concurrent order builds with a configurable pool; no per-request delay found (**unverified** beyond the code reviewed). |

**What users report.**

- A first-time azad user hit HTTP 503 after a second large run and asked if they were locked out; they never replied, and the maintainer suggested clearing Amazon cookies ([azad 392](https://github.com/philipmulcahy/azad/issues/392)). Not confirmed as throttling.
- Intermittent "TypeError: Failed to fetch" from Monarch's service worker, which "randomly started working" for some users ([monarch 38](https://github.com/alex-peck/monarch-amazon-sync/issues/38)). Cause unknown; could be network, cookies or blocking.
- Stale or broken cookies and partial logout are the common cause of "zero orders" and redirect loops, cured by clearing Amazon cookies and signing in again (azad 411, 407, 382; Monarch 6). azad ships a "force logout" button for this.
- No source reports a CAPTCHA while browsing orders with an established session. CAPTCHAs, WAF and bot challenges are reported mainly at sign-in, for scripted logins. **Absence of reports is not evidence of safety.**

**How a robot-check page is detected.** amazon-auth's markers (all in code, [constants.py L114-L119](https://github.com/alexdlaird/amazon-auth/blob/275d0bf/amazonauth/constants.py#L114), [selectors.py L75-L101](https://github.com/alexdlaird/amazon-auth/blob/275d0bf/amazonauth/selectors.py#L75)), with fixtures in amazon-orders' `tests/resources/auth/`:

- text matching "verify that you're not a robot" plus "Enable JavaScript";
- `/_sec/verify?provider=interstitial` (Akamai interstitial);
- `window.gokuProps` and `script[src*="awswaf.com"]` (AWS WAF challenge); cookie `aws-waf-token`;
- `form[action$='validateCaptcha']` or an input with id starting `captchacharacters` (classic CAPTCHA);
- HTTP 4xx/5xx responses, and a redirect off the requested path.

**Recommendation.** One request at a time (already in the design); 2 to 5 s jittered delay between requests; cap of ~20 invoice fetches per run, fewer list pages (a 30-day daily window needs one or two); stop the whole run and surface a "Amazon asked for verification, open amazon.com and complete it, then press Sync" state on any robot-check marker, 429 or 503 (no automatic retry loop, back off to the next daily run). Fetch only the pages we need: no tracking pages, no product pages.

## 6. Background fetch from an extension

- **Monarch is the only MV3 service worker example.** It calls plain `fetch(url)` with no `credentials` option from the service worker, with `host_permissions` for `https://amazon.com/*` and `https://www.amazon.com/*`, on an `alarms` timer ([manifest.js](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/manifest.js), [background/index.ts](https://github.com/alex-peck/monarch-amazon-sync/blob/a2719228130dd2b244331bd7039160873ab4e2c0/src/pages/background/index.ts)). It worked for many users and it parsed with `cheerio` because a service worker has no `DOMParser`. That the logged-in cookies were sent is implied by its parsing of order pages, but no source states it. **No explicit statement about SameSite behavior was found in code or Chrome's network-requests docs** ([developer.chrome.com](https://developer.chrome.com/docs/extensions/develop/concepts/network-requests) covers only host permissions).
- Every other extension makes same-origin requests from a content script on an amazon.com tab (azad; the exporter uses `credentials: 'include'`, [content.ts L1009, L1150](https://github.com/Xenolphthalein/order-history-exporter-for-amazon/blob/39142ed3a969c55846f12a172eea83d5720dc080/src/content/content.ts#L1009)). That does not test our extension-origin path.
- The only reported problem near this is monarch issue 38 ("Failed to fetch" with valid auth), unresolved and not diagnosed.
- bilal's exporter avoids the question by navigating a background tab and scraping the rendered DOM.

Treat extension-origin cookie sending as **(unverified)**. Set `credentials: 'include'` explicitly, and verify early with a manual test on a logged-in profile. If it fails or encrypted pages turn out to be common, the fallback is an offscreen document or a pinned background tab loading the page for real. That also covers the client-rendered case.

## Recommendation for v1

**List URL.** `https://www.amazon.com/your-orders/orders?timeFilter=last30` (or `months-3`), then follow the Next link (`page=N`). Always send `language=en_US`, as azad does for .com. If the response has encrypted cards (`div.csd-encrypted-sensitive` and no readable order number), retry once with `&disableCsd=no-js`; if the retry is still encrypted or the page is a skeleton, report "layout changed" and keep the HTML. Backfill uses `timeFilter=year-YYYY`.

**Invoice URL.** `https://www.amazon.com/gp/css/summary/print.html?orderID=<id>`. Take the link from the card's "View invoice" anchor when present and fall back to constructing it. It is the page Amazon itself prints, it is server-rendered, it exists for all standard orders and carries items, charge summary, payment method and shipment status in one request. Save its raw HTML next to the PDF. Do not fetch the popover in v1 (it adds a request and only matters for seller PDFs, mostly EU). Order details (`/gp/your-account/order-details`) is the fallback if the print page lacks a field, but costs a second request per order.

**Order types in v1.**
- In: standard physical orders, multi-shipment, Subscribe & Save once charged.
- In, best effort: gift-card-paid and partly-paid orders; orders with a refund (archive the original).
- Out: cancelled (skip); Whole Foods, Fresh and other non-standard receipt routes; digital orders (`D01-…`, Kindle, video, apps). They use different pages and labels, have no shipment concept, and are the most-reported breakage in azad. This answers design.md open question 4 with "out for v1, revisit once the physical path is stable". Detect them by the order-ID prefix and the Whole Foods routes, and list them as "skipped (unsupported type)" instead of failing silently.

**Fields we can rely on** (stable across parsers and fixtures): order ID, order date, grand total, item names, quantities and unit prices, seller, subtotal / shipping / tax lines matched by label, and a per-shipment status string. **Optional, best effort**: payment card type and last 4, shipped date, promotions, gift-card amount. Not stored: ship-to name and address.

**Biggest risks.**
1. The list page may be CSD-encrypted or client-rendered, so plain `fetch` + `DOMParser` could return unusable HTML. Mitigation above; keep a tab/iframe fallback ready.
2. Extension-origin fetch may not send cookies as hoped. **(unverified)** Test first.
3. Layout churn: azad needed fixes on 2026-08-16 and 2026-08-18 and has an open payment bug on 2026-11-02. Use label-text matching, multiple selectors per field, and a "found nothing" failure state.
4. Anti-bot behavior is under-documented for logged-in browsing. Go slow and stop at the first challenge.
5. Retention window: details/print pages for older orders may stop working **(unverified)**; backfill depth is uncertain.

## Code worth adapting (credit in `NOTICE` if used)

azad is Apache-2.0, so adapted code needs its notice carried into `NOTICE`. amazon-orders is MIT (selectors only; keep the copyright line if copying selector lists).

| What | Where | Take or leave |
|---|---|---|
| Fallback-strategy runner (`firstMatchingStrategy`, stats on which strategy hit) | [`src/js/strategy.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/strategy.ts) | Take the idea. It underlies azad's resilience. |
| Order ID, date, total extraction with three strategies each; multi-language date labels | [`src/js/order_header.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_header.ts) | Adapt (XPath). |
| Shipment extraction, delivered/arriving/returned classification | [`src/js/shipment.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/shipment.ts) | Adapt `is_delivered`; leave tracking-page fetches. |
| Totals by label (`od-subtotals`, grand total variants, tax, gift, S&S) | [`src/js/order_details.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_details.ts) | Adapt; the XPath list is a ready catalog of label variants. |
| Payment extraction (four strategies, including the text-topology one) | [`src/js/payment.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/payment.ts), [`topology.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/topology.ts) | Adapt strategy 3 only (current layout). Leave the table-era strategies. |
| URL templates, site table | [`src/js/url.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/url.ts), [`order_list_page.ts`](https://github.com/philipmulcahy/azad/blob/87d90afa8ee648ec48b040e338275ba8797ae0d4/src/js/order_list_page.ts) | Reference; we write our own. |
| CSD check and fallback | amazon-orders [`orders.py`](https://github.com/alexdlaird/amazon-orders/blob/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/amazonorders/orders.py#L741) | Reimplement in TS (MIT; Python). |

Leave behind: the request scheduler, iframe worker, CSV/table UI, caching and Chrome-popup plumbing; they serve azad's click-to-export model, not ours.

**Fixtures.** amazon-orders' [`tests/resources`](https://github.com/alexdlaird/amazon-orders/tree/4d3e8e7e2942d5cb27ca5aea57d1a45c0d65a5cc/tests/resources) hold MIT-licensed, already sanitized pages (print summary, order history cards for several types, an encrypted page, auth/WAF/CAPTCHA pages, an "unable to load order details" error). They are a good seed for our parser tests, but contain placeholder-looking names and addresses. Our PII check must pass on them before we commit any, and we should add our own current captures.

## Not confirmed

- Everything about live behavior for a consumer US account in November 2026: no live access.
- Whether the plain `fetch` of `/your-orders/orders` returns readable cards or encrypted/skeleton ones for ordinary accounts, and how often.
- Cookie sending and SameSite behavior for extension-origin `fetch`.
- The full set of shipment status strings, and whether order totals change between purchase and shipment.
- The 30-day detail-page retention claim (single, unreplicated report).
- Whether US orders ever list seller PDF invoices in the popover.
- Which digital-order invoice URL works now (`/gp/digital/your-account/order-summary.html` vs `order-details`).
- Real throttling thresholds. The only 503 report is unexplained.
- Licenses for the bilal and oliverswitzer repos (not read), and amazon-auth's (assumed MIT).
