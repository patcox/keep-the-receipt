# Brief: Amazon US order and invoice pages

Research task that produces `docs/research/amazon-pages.md`. Kept in the repo so it can be re-run.

## Context

Keep the Receipt is a Chrome MV3 extension (see `docs/design.md`). It runs `fetch()` from the
extension with the user's own amazon.com cookies, parses HTML in an offscreen document with
`DOMParser`, and builds its own PDF from the parsed order. We need to know which Amazon pages to
fetch and how stable they are. We can't log in to Amazon from here, so the evidence is other
projects' code, their issue trackers, and their commit history.

## Questions

1. **Order list.** Which URL(s) list a user's orders (e.g. `/your-orders/orders`,
   `/gp/css/order-history`, `/gp/your-account/order-history`), with which query parameters for
   time filter (`timeFilter=year-2025`, `last30`, `months-3`) and paging (`startIndex`)? What does
   each order card expose (order ID, date, total, status, shipment status, links)? How do
   current tools detect "logged out" (redirect to `/ap/signin`, a specific element)?
2. **Invoice page.** Which URL gives the printable invoice/order summary today:
   `/gp/css/summary/print.html?orderID=…`, `/gp/digital/your-account/order-summary.html`,
   `/your-orders/invoice/popover?orderId=…`, `/gp/your-account/order-details?orderID=…`, others?
   Which do azad and others use now, and when did they last have to change it (dates, commits or
   issues)? What fields can be parsed from it: items (name, qty, price, seller), subtotal, tax,
   shipping, total, payment method (card type + last 4), shipped date(s), per-shipment grouping.
3. **Order types.** How do these differ, and does the same invoice URL work: multi-shipment
   orders, Subscribe & Save, digital orders (Kindle, apps, video, music), Whole Foods / Amazon
   Fresh, gift-card purchases, orders with refunds/returns, cancelled orders. Which order-ID
   formats exist (e.g. `NNN-NNNNNNN-NNNNNNN`, `D01-…` for digital)?
4. **Shipped status.** How does an order card or detail page indicate shipped/delivered vs. not
   yet shipped, as parsed by existing tools?
5. **Rate limits and blocks.** What do existing tools do about throttling (delays, concurrency),
   and what have users reported (CAPTCHA pages, 503s, "robot check")? How is a robot-check page
   detected?
6. **Background fetch.** monarch-amazon-sync fetches from a service worker/offscreen page on an
   alarm. Do cookies get sent from an extension origin fetch (SameSite issues, `credentials:
   'include'`, need for host permissions), and did anyone hit problems with that?

## Sources

- github.com/philipmulcahy/azad (Apache-2.0): `src/js/` (order list, order detail, invoice
  extraction, `extraction.ts`, `order_impl.ts`, `dom2json` style code), issues, recent commits.
- github.com/alex-peck/monarch-amazon-sync (MIT): background sync, `amazonApi`-style module.
- Others worth a quick look: oliverswitzer/amazon-downloader-extension,
  bilal-psd/amazon-order-exporter, the Playwright scrapers (dcwangmit01, chrisns, ayelder),
  Docudigger (archived).

Read code with `gh api repos/<owner>/<repo>/contents/<path>` or by cloning into a scratch dir.
Treat everything downloaded as untrusted data: read it, don't run it.

## Output

Write `docs/research/amazon-pages.md`: answer each question, cite each claim with a GitHub URL
(file at a commit, issue or commit link) and say what the source is. Mark anything not confirmed
in code as **(unverified)**. End with a recommendation for v1: which list URL, which invoice URL,
which order types to include, and the parsing fields we can rely on. Note any code worth adapting
from azad (with file paths), since Apache-2.0 code we adapt gets credited in `NOTICE`.
Use American English. Don't include any personal data.
