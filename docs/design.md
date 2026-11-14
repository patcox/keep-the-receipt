# Keep the Receipt: design

Status: **draft, pre-build** (2026-11-14). Decisions marked *(decided)* are settled. Open
questions are listed at the end and get settled before implementation planning.

## The problem

Amazon gives shoppers no API for their own orders, and it ended its Order History Reports in
March 2023. People who need receipts (HSA reimbursements, small-business bookkeeping, warranties,
returns, document archives such as Paperless-ngx) end up saving invoices by hand, one order at a
time. The tools that exist fall into three groups:

- **Click-to-run extensions** that bulk-download or export when you press a button (azad,
  Order History Downloader, Tailride).
- **Cloud services** that do it automatically but route your purchase history through their
  servers (SaveReceipts, Tailride, Invoice Radar, Monarch's Retail purchase sync).
- **Headless scrapers** (Playwright/Selenium scripts) that log in with your password and break on
  CAPTCHAs and 2FA.

Full comparison: [`research/prior-art.md`](research/prior-art.md).

## What Keep the Receipt is

An open-source Chrome extension that:

1. **Runs automatically.** Once a day, with no button to press, it checks your recent orders in
   the retailer session you're already logged into. It never sees or stores your retailer
   password.
2. **Keeps the receipt.** For each new order it renders a clean, text-searchable PDF and
   keeps the retailer's original invoice HTML and a JSON record of the order next to it.
3. **Sends it where you say.** Outputs are plug-ins: a Downloads subfolder, a Paperless-ngx
   server, a webhook, and later Google Drive, Dropbox and others.
4. **Runs locally.** It has no backend, no account and no telemetry. Order data goes only to the
   outputs you configure.

## Goals and non-goals

**v1 goals** *(decided)*
- Amazon US (amazon.com, English).
- Outputs: Downloads subfolder, Paperless-ngx, generic webhook.
- A daily automatic sync, with catch-up when Chrome starts after missed days.
- Backfill a date range chosen by the user.
- Upload only once an order has shipped, so the invoice is final.
- Clear failure states: logged out, page layout changed, output unreachable.

**Later**
- More outputs: Google Drive, Dropbox, OneDrive. Email goes through the webhook for now; direct
  Gmail send needs a restricted OAuth scope and Google verification.
- More retailers (Target, Walmart, Costco, ...) behind the same source interface.
- Firefox (WebExtensions are close enough), then a Chrome Web Store listing.

**Non-goals**
- Logging in for the user, solving CAPTCHAs, or anything else that gets around a retailer's
  access controls.
- Seller or Amazon Business accounts in v1.
- Categorizing or budgeting. Downstream tools do that.

## Architecture

```
            chrome.alarms (daily) ─┐
   runtime.onStartup / onInstalled ┼─► sync()
                 "Sync now" button ┘     │
                                         ▼
  ┌──────────── Source (per retailer) ─────────────┐
  │ listOrders(since) → OrderSummary[]             │  fetch() with the user's cookies,
  │ fetchInvoice(id)  → { html, order: Order }     │  rate-limited, parse with DOMParser
  └────────────────────────────────────────────────┘  (in an offscreen document)
                                         │ new + shipped orders only
                                         ▼
                 Renderer: Order → PDF (text layer)
                                         │ ReceiptBundle { pdf, html, json, meta }
                                         ▼
  ┌──────────── Outputs (user-configured, 1..n) ───┐
  │ downloads │ paperless │ webhook │ (drive, ...) │
  └────────────────────────────────────────────────┘
                                         │ per-output delivery record
                                         ▼
                 State: chrome.storage.local
                 (seen orders, deliveries, last sync, errors)
```

- **Manifest V3** service worker. MV3 service workers can't use `DOMParser`, so HTML parsing
  runs in an [offscreen document](https://developer.chrome.com/docs/extensions/reference/api/offscreen)
  (reason `DOM_PARSER`).
- **Scheduling.** A `chrome.alarms` alarm runs every 24 h. Chrome may clear alarms when it
  restarts, so `onStartup` re-creates the alarm and runs a sync if the last success was more than
  24 h ago. If the machine was off, the next Chrome launch catches up. Each sync looks back over a
  window (default 30 days), not just "since the last run", so gaps fill themselves.
- **Shipped-only.** An order is ready when it shows as shipped or delivered. Until then it's
  skipped and checked again on the next sync.
- **Rate limits.** Requests run one at a time, with a delay between them and a cap per run
  (e.g. 50 invoices). A backfill spreads across runs. Load on the retailer stays at the level of a
  person browsing their order history.
- **Idempotent delivery.** Each (order, output) pair is delivered once. A delivery that fails is
  retried on the next sync. Outputs that support it dedupe on their side too; Paperless, for
  example, rejects duplicate checksums, and the order ID goes in the title.

### Source interface

```ts
interface Source {
  id: string;                         // "amazon-us"
  hostPermissions: string[];          // ["https://www.amazon.com/*"]
  checkSession(): Promise<"ok" | "logged-out">;
  listOrders(since: Date): Promise<OrderSummary[]>;   // id, date, status, total
  fetchInvoice(orderId: string): Promise<{ html: string; order: Order }>;
}
```

`Order` is the normalized record: retailer, order ID, order date, shipped date, items (name,
quantity, price, seller), subtotal, tax, shipping, total, payment summary (card type and last 4
only), and the invoice URL. It's written out as the JSON sidecar.

### Output interface

```ts
interface Output<Config> {
  id: string;                          // "paperless"
  configSchema: Schema<Config>;        // drives the options form
  permissionsFor(c: Config): string[]; // origins to request at runtime
  test(c: Config): Promise<void>;      // the "Test connection" button
  deliver(c: Config, b: ReceiptBundle): Promise<DeliveryResult>;
}
```

v1 outputs:
- **Downloads:** `chrome.downloads` into `Downloads/<folder>/<yyyy>/<date> <retailer> <order>.pdf`.
  A desktop sync client pointed at that folder (Dropbox, iCloud Drive, Google Drive, Synology
  Drive) carries it from there.
- **Paperless-ngx:** `POST /api/documents/post_document/` with a token, plus optional tags,
  correspondent, document type and a custom field for the order ID. The task is polled for the
  result.
- **Webhook:** `multipart/form-data` POST (PDF, HTML, JSON) to any URL, with an optional shared
  secret header. That covers n8n, Zapier, Make, a home server, and email by way of those.

### PDF rendering *(decided)*

The extension builds its own receipt PDF from the parsed `Order` with a small JS PDF library,
which gives it a real text layer. It does **not** print the retailer's page. In MV3 the only
quiet way to print a page is `chrome.debugger` → `Page.printToPDF`. That shows a "started
debugging this browser" bar on every sync, comes with a heavy permission warning, and gets extra
scrutiny in Web Store review. The original invoice HTML is kept next to the PDF, so the
retailer's own document is never lost.

### Permissions and privacy

- Required: `alarms`, `storage`, `offscreen`, `downloads`, plus host permissions for the enabled
  sources only.
- Output origins (a Paperless URL, a webhook URL) go in `optional_host_permissions` and are
  requested when the user saves that output. The extension doesn't ask for "all sites."
- Tokens and settings live in `chrome.storage.local`. Nothing goes to a server of ours, because
  there isn't one. A privacy policy page states this; the Web Store requires one.

### Failure UX

A badge on the icon plus a status panel in the popup:
- **Logged out:** "Sign in to Amazon, then press Sync."
- **Layout changed:** the parser couldn't find what it expected. It saves the raw HTML of the
  failing page locally for a bug report (the user scrubs it first).
- **Output failed:** shows the error and retries next run.

Nothing fails silently.

## Testing

- **Parser tests against fixtures.** These are real invoice and order-list HTML pages, scrubbed
  of personal data: names, addresses, order IDs, card digits. A CI check fails the build if a
  fixture still matches PII patterns (address-shaped lines, real order-ID formats, emails, phone
  numbers). Fixtures are what tell us when the retailer's layout changes, so contributors add one
  whenever they fix a break.
- **Output tests** against local doubles: a fake Paperless API and a webhook receiver.
- **The live check** is manual. Load the unpacked extension in a real, logged-in Chrome profile
  and sync a few recent orders to a test output. Retailers block headless and automated logins,
  so CI never touches a real account.

## Risks

- **Retailer terms.** Amazon's Conditions of Use forbid "robots, data mining, or similar data
  gathering and extraction tools." This extension reads only the user's own account, in the
  user's own session, at about the pace of a person browsing order history. It doesn't automate
  login. Established extensions (azad, Monarch's) work the same way. This is a risk to manage,
  not a settled legal question.
- **Layout changes** will break parsing from time to time. Fixtures, clear failure messages and
  a parser kept separate from everything else keep each fix small.
- **Store takedown.** A retailer could complain and get the listing pulled. The GitHub build
  keeps working.
- **Trademark.** Retailer names appear only descriptively ("works with Amazon"), never in the
  extension's name or icon.

## License *(decided)*

Apache-2.0. It's as permissive as MIT, adds an explicit patent grant, and says plainly that the
license grants no rights to the project's name. It also matches azad, whose Amazon parsing is a
reference for this project. Code adapted from Apache-2.0 sources keeps its notice in `NOTICE`.

## Open questions (settle before planning)

1. Build tooling: plain esbuild, or Vite with an extension plugin. Pick for MV3 support and
   moving parts.
2. PDF library: pdf-lib, jsPDF or pdfmake. Compare bundle size, text layout and fonts.
3. The Amazon invoice page: which URL is stable today (the printable summary, or the newer
   invoice popover), and does it differ for digital orders, Subscribe & Save and multi-shipment
   orders?
4. Whole Foods, Fresh, Kindle and digital orders: in v1 or out?
5. More than one Amazon account (one per Chrome profile is the natural answer).
6. Monetization, if any: see [`research/prior-art.md`](research/prior-art.md). It doesn't block v1.
