# Prior art and monetization research

Research date: 2026-11-14. Brief: [prior-art-brief.md](prior-art-brief.md). Every claim links to its source. Figures from
the Chrome Web Store were read on the research date and move daily. Anything marked **(unverified)** came from a search
snippet, a competitor's page or a third-party tracker and was not confirmed at the primary source.

## Summary

- Nothing found combines all four of Keep the Receipt's traits: Amazon PDF invoices, fully automatic daily runs, a
  Paperless-ngx output, and open source with no backend. Each neighbor has two or three.
- The closest overlap is **Invoice Radar** (350+ portals including Amazon, a Paperless-ngx export, local token storage),
  but it is a paid app from $140/year with a credit cap, and its Paperless export looks manual.
- The only large, long-lived open-source Amazon extension is **azad**: 60,000 users, 335 stars, Apache-2.0, and it
  produces a table/CSV, not PDFs. Its paywall is a thin client-side flag around ExtensionPay.
- The **Playwright/CLI scrapers** do produce Amazon invoice PDFs but need credentials in a script, break when Amazon
  changes its pages, and one major one (Docudigger) was archived in March 2026.
- **Monarch's own extension** (100,000 users, 3.3 rating) shows both the demand for retail sync and how hard it is to
  keep reliable.

## 1. Prior art

### Comparison table

"Auto" means it runs without the user clicking each time. Store figures are Chrome Web Store (CWS) unless noted.

| Name | What it does | Auto? | Retailers | Outputs | License / hosting | Price | Size signals | Notes on how it works |
|---|---|---|---|---|---|---|---|---|
| [azad / Amazon Order History Reporter](https://github.com/philipmulcahy/azad) | Scrapes order history into a sortable table; CSV/JSON export | Click-to-run (user opens orders page, picks a year) | 16 Amazon domains ([manifest](https://github.com/philipmulcahy/azad/blob/master/src/manifest.json)) | Table, CSV, JSON; no PDFs, no cloud/Paperless | Apache-2.0, local; talks to ExtensionPay and the author's site only | Free; premium about $15/year per a [README quote](https://github.com/philipmulcahy/azad/blob/master/README.md), $7.50 earlier per [issue 264](https://github.com/philipmulcahy/azad/issues/264) | [60,000 users, 4.5 from 452 ratings, v1.16.24, updated 2026-08-19](https://chromewebstore.google.com/detail/amazon-order-history-repo/mgkilgclilajckgnedgjgnfdokkgnibi); 335 stars, 18 contributors, created 2017 (GitHub API) | Permissions `cookies`, `storage`, `contextMenus` plus Amazon host permissions; fetches order and invoice pages in the user's session and parses HTML |
| [alex-peck/monarch-amazon-sync](https://github.com/alex-peck/monarch-amazon-sync) | Matches Amazon orders to Monarch transactions and writes item names into notes | Daily while browser is open, plus manual force-sync | Amazon | Monarch notes (no PDFs) | MIT; local, grabs Monarch API key from the open Monarch tab | Free | 94 stars; last push 2024-10-29; README calls it BETA and says it is not on the CWS | Install by loading unpacked zip. Source of order data is not explained in the README |
| [Monarch "Retail purchase sync"](https://chromewebstore.google.com/detail/monarch-money-retail-purc/imfcckkmcklambpijbgcebggegggkgla) | Matches orders to Monarch transactions, splits and recategorizes using AI | Optional every-24-hours sync ([help](https://help.monarch.com/hc/en-us/articles/36463599367188-Using-the-Retail-Sync-Extension)) | Amazon US (English site, no business accounts) and Target | Monarch only | Closed; item name, merchant, price, date sent for AI processing; needs a Monarch subscription **(unverified: that last point is inferred)** | Free with Monarch | [About 100,000 users, 3.3 from 99 ratings, v1.0.34, updated 2026-09-24](https://chromewebstore.google.com/detail/monarch-money-retail-purc/imfcckkmcklambpijbgcebggegggkgla); a [status incident](https://status.monarch.com/cmc241bs5000mga2td7rq9ih6) for failing Amazon syncs; recent reviews cite non-syncing per [ExtScope](https://www.extscope.org/extension/imfcckkmcklambpijbgcebggegggkgla) **(unverified, third-party)** | Does not save PDFs |
| [Tailride](https://chromewebstore.google.com/detail/tailride/pfkgmhfcmehahfoemipfmdbnddegaiec) | Pulls invoices from 20+ portals incl. Amazon (personal and Business) into a cloud dashboard with AI extraction | Scheduled while browser runs ([blog](https://tailride.so/blog/desktop-app-vs-browser-extension-vs-cloud-fetcher)); also one-click date-range bulk download ([Amazon page](https://tailride.so/tool/amazon-invoice-downloader)) | Amazon, Meta Ads, Google Ads, Adobe, Notion and others ([blog](https://tailride.so/blog/best-chrome-extensions-for-invoices)) | QuickBooks, Xero, DATEV, Google Drive, Sheets; Google Drive/OneDrive per [CWS listing](https://chromewebstore.google.com/detail/tailride/pfkgmhfcmehahfoemipfmdbnddegaiec) via search snippet **(unverified)** | Closed SaaS; files go to a Tailride dashboard | [Free for 10 invoices/month; $158/yr for 900, up to $1,658/yr for 18,000](https://tailride.so/pricing) | [2,000 users, 5.0 from 4 ratings, v1.2.5, updated 2026-11-01](https://chromewebstore.google.com/detail/tailride/pfkgmhfcmehahfoemipfmdbnddegaiec) | Uses the logged-in session; "no passwords stored". No Paperless or Dropbox found |
| [Invoice Radar](https://invoiceradar.com/) | Collects invoices from 350+ portals, email, scans; AI extraction | Yes for collection; its [Paperless-ngx export docs](https://invoiceradar.com/docs/export/paperless-ngx) describe "Export Now" and per-invoice export, and do not say whether sync is automatic | Amazon (US/DE/Ads/Seller) among 350+ | Paperless-ngx (token stored encrypted on device, sent direct, no intermediary), Dropbox, Google Workspace, Microsoft 365 and more | Closed app with open plugin ecosystem on GitHub; "local processing" claimed but not explained | [Trial with watermark; $140, $252, $490 or $1,400/yr by credit cap of 300 to 6,000 docs](https://invoiceradar.com/pricing) | Not measured | Described on a [Capterra-style listing](https://www.capterra.ca/software/1100976/invoiceradar-com) as desktop/web/mobile, not an extension |
| [SaveReceipts Web Clipper](https://chromewebstore.google.com/detail/ennegfkegadgkpeijnmainabinbblhjl) | Detects order-confirmation pages and saves the receipt to Google Drive plus a Google Sheet row | One click per receipt | Any site | Google Drive and Sheets only | Closed; says data goes straight to the user's Drive, not its servers | Requires SaveReceipts Pro, EUR 25/year | 4 users, no ratings, v1.0.0, updated 2026-04-29 | Page-level clipper, not an order-history crawler |
| [Receipt Wallet](https://chromewebstore.google.com/detail/ldlenndfbagaabopndhijodimjbliiil) | Saves receipts into a Drive folder while browsing | Click | Any | Google Drive | Closed | Not found | About 2 users, one rating, v1.0.0 (2026-03-24) per [chromeboard](https://chromeboard.com/extension/receipt-wallet-ldlenndfbagaabopndhijodimjbliiil) **(unverified, third-party)** | Negligible traction |
| "Order History Downloader" family: [OrderPro](https://chromewebstore.google.com/detail/amazon-order-history-down/gnemaookpcekfeidmgocagofgijpoced), [Amazon Japan Order History Downloader](https://chromewebstore.google.com/detail/amazon-japan-order-histor/kjdfladalknkkphpagfmfjndkaapocil) | Export order lists to CSV/Excel/Sheets | Click | Amazon, eBay, Walmart and more (OrderPro) | CSV, XLSX, Sheets; no PDFs per listing | Closed | In-app purchases | [OrderPro: 3,000 users, 4.4 from 39 ratings, updated 2026-11-10](https://chromewebstore.google.com/detail/amazon-order-history-down/gnemaookpcekfeidmgocagofgijpoced) | The exact name "Order History Downloader" matches no single extension |
| [oliverswitzer/amazon-downloader-extension](https://github.com/oliverswitzer/amazon-downloader-extension) | Downloads orders as CSV and invoices | Click, developer-mode install | Amazon | Local files | No license set | Free | 1 star; last push 2023-10-03 | Effectively dormant |
| [bilal-psd/amazon-order-exporter](https://github.com/bilal-psd/amazon-order-exporter) | Exports order history to CSV, all regions | Click | Amazon | CSV | MIT | Free | 1 star; pushed 2026-03-07 | No PDFs |
| [dcwangmit01/amazon-invoice-downloader](https://github.com/dcwangmit01/amazon-invoice-downloader) | Python CLI, Playwright drives Chromium, logs in, downloads invoice PDFs by date range | Run it yourself (cron possible) | Amazon.com | `YYYYMMDD_<total>_amazon_<orderid>.pdf` in a folder | MIT, local | Free | 80 stars; pushed 2025-11-03; "last verified August 2025" | Credentials via flags/env; pauses for 2FA and CAPTCHA; README warns Amazon page changes break it |
| [chrisns/amazon-invoice-downloader](https://github.com/chrisns/amazon-invoice-downloader) | Same title and goal | Run it yourself | Amazon | PDFs | MIT | Free | 2 stars; pushed 2026-11-10 | Actively touched |
| [ayelder/amazon-invoice-downloader](https://github.com/ayelder/amazon-invoice-downloader) | Bulk invoice download tool | Run it yourself | Amazon | PDFs | MIT | Free | 2 stars; pushed 2025-01-01 | Dormant |
| [Disane87/docudigger](https://github.com/Disane87/docudigger) | Docker/CLI scraper for Amazon invoice PDFs, aimed at taxes or a DMS | Scheduled via Scrape Dojo successor | Amazon only | PDF files; Paperless-ngx not mentioned | MIT | Free | 101 stars; **archived 2026-03-07**, points to Scrape Dojo | Shows maintenance fatigue on scraper tools |
| [eshaffer321/itemize](https://github.com/eshaffer321/itemize) | Itemizes and categorizes retail transactions in Monarch | Scripted | Walmart, Costco, Amazon (per search summary) | Monarch | No license set | Free | 16 stars; pushed 2026-11-12 | Not a PDF tool |
| [Paperless Web Capture](https://chromewebstore.google.com/detail/dkaokmnnioohgamnfjdkhhkhddielkbl) | Captures the current page or PDF to a Paperless-ngx server using `Page.printToPDF` | Click per page | Any | Paperless-ngx | Support link points to [aasmoe/paperless-web-extension](https://github.com/aasmoe/paperless-web-extension); license not checked | Free | 601 users, 5.0 from 2 ratings, v1.2.0 (2026-08-15) | Closest Paperless-ngx extension; not an order crawler |
| [Send to Paperless-ngx (Firefox)](https://firefox.marketplace.browsertotal.com/fr/firefox/addon/3794243344e548fc9dd9/reviews/) | Sends an open PDF to Paperless-ngx | Click | Any | Paperless-ngx | Not checked | Free | Not measured | Only a review mirror was found |
| [Staxxer Amazon Seller Fee Invoice Exporter](https://chromewebstore.google.com/detail/nokbiigpdngibiigdookaofkgcomfjai) | Bulk-downloads seller fee and ads invoices as ZIP from Seller Central | Click | Amazon Seller Central | ZIP of PDFs | Closed, free | Free; lead-gen for Staxxer | Roughly 400 to 775 users per [search results](https://extscope.org/zh/extension/nokbiigpdngibiigdookaofkgcomfjai) **(unverified)** | Sellers only, so not a consumer competitor |
| [Invoice Harvester](https://invoice-harvester-site.pages.dev/) | Finds purchase invoices from an Amazon account and saves PDFs locally | Click | Amazon sellers | Local PDFs | Closed, free | Free | Marked "coming soon" to the CWS | Marketing page only **(unverified)** |

Other items surfaced while searching, not profiled in depth: [BillCollector](https://redlib.hbubli.cc/r/Paperlessngx/comments/1ji92to/automatically_feed_paperlessngx_with_documents) (Selenium-based portal scraper that feeds Paperless-ngx; a commenter there said Selenium was unreliable for Amazon Business), [Monarch Money Amazon Connector](https://pypi.org/project/monarch-money-amazon-connector/) (beta Python package), [AmazaSpend](https://addons.mozilla.org/firefox/addon/amazaspend/) (Firefox; sends order data to its own service), [Veryfi extension](https://www.veryfi.com/chrome-extension/) (click-per-receipt, cloud extraction).

### Patterns worth noting

- **Gap confirmed.** No result pairs fully automatic Amazon invoice PDFs with Paperless-ngx in an open-source extension. The nearest are Invoice Radar (closed, paid), Paperless Web Capture (click per page) and the CLI scrapers (no browser session, so credentials and 2FA are the user's problem).
- **Extensions reuse the user's session.** azad, Tailride and monarch-amazon-sync all rely on the logged-in browser rather than stored passwords. Tailride's own comparison says the cost is that [collection only runs while the browser is open and pauses when the session expires or a security check appears](https://tailride.so/blog/desktop-app-vs-browser-extension-vs-cloud-fetcher). That is a vendor blog, so treat it as a competitor's framing, but it matches Keep the Receipt's design constraints.
- **Amazon removed its own export.** Order History Reports for consumer accounts [ended around March 20, 2023](https://tinyapps.org/blog/202303250700_amazon_order_history_reports.html), replaced by a data-request page. That is the demand driver for azad's growth after June 2023.
- **Reliability is the recurring complaint.** Monarch's extension has a public incident and a 3.3 rating; azad's open issues are mostly parsing fixes after Amazon markup changes (for example [issue 418](https://github.com/philipmulcahy/azad/issues/418), [issue 422](https://github.com/philipmulcahy/azad/issues/422)); Docudigger was archived.
- **Not searched exhaustively:** r/selfhosted, r/personalfinance and npm. Searches for r/paperlessngx and r/MonarchMoney threads returned only the items cited above. Firefox Add-ons returned no PDF-invoice tool for Amazon.

## 2. azad's premium tier in detail

**What it costs.** The README quotes a reviewer calling it ["well worth the $15/year"](https://github.com/philipmulcahy/azad/blob/master/README.md). An [unsubscribe issue](https://github.com/philipmulcahy/azad/issues/264) shows an earlier $7.50/year price, and the author saying he raised the rate for new subscribers because "some users have been naughty and dealing with them has consumed infeasible amounts of my energy: it is a bit of a 'tragedy of the commons' situation." The CWS listing says only ["Offers in-app purchases"](https://chromewebstore.google.com/detail/amazon-order-history-repo/mgkilgclilajckgnedgjgnfdokkgnibi) and that subscribing is necessary "ONLY if you want to use a small set of features aimed at commercial users."

**What is free and what is paid.** [commercial_features.md](https://github.com/philipmulcahy/azad/blob/master/doc/commercial_features.md) tells the history:
- 2022 survey of commercial users picked the roadmap.
- June 2023 (v1.9.26): ASIN/quantity/price and "scrape only the last 1-3 months" went behind a subscription, framed as a charity fundraiser for a hospice with a GBP 5,000 goal.
- When the goal was reached (announced in the 2023-08-19 update), those features became free for everyone.
- Today, per the [README](https://github.com/philipmulcahy/azad/blob/master/README.md), "only shipment/tracking data sits behind the paywall." Existing subscribers kept access to later batches without paying more.
- Open issues show a paid "Transactions" view too ([issue 338](https://github.com/philipmulcahy/azad/issues/338): "Paid for Premium but no Transactions showing").

**How payment and license checks work (code read via `gh api`).**
- [`extpay_client.ts`](https://github.com/philipmulcahy/azad/blob/master/src/js/extpay_client.ts) wraps the [ExtPay](https://github.com/glench/ExtPay) library with a fixed extension ID (`amazon-order-history-reporter-premium-annual`). `check_authorised()` calls `getUser()` and returns true if `subscriptionStatus == 'active'` or `user.paid`. A code comment admits the status is sometimes undefined and the author "has not found a way of undoing the damage."
- [`background.ts`](https://github.com/philipmulcahy/azad/blob/master/src/js/background.ts) `handleAuthorisationRequest` maps the feature id `premium_preview` to that result and stores it in local settings as the boolean `preview_features_enabled`.
- [`table_config.ts`](https://github.com/philipmulcahy/azad/blob/master/src/js/table_config.ts) and [`settings.ts`](https://github.com/philipmulcahy/azad/blob/master/src/js/settings.ts) then read that boolean to show or hide the shipment columns and the premium table types. There is no signature, no obfuscation, and no server-side data gating. Anyone building from source can flip the flag. The real protection is that the code is a modest feature, not that the check is strong.
- [`manifest.json`](https://github.com/philipmulcahy/azad/blob/master/src/manifest.json) allows `connect-src` to `extensionpay.com` and the author's own domain only. So the paywall costs one extra network destination and no backend of the author's own.
- Billing management is a Stripe customer-portal link opened by `display_console()`.

**License.** Apache-2.0, same as Keep the Receipt would be. The paywall sits inside an Apache-2.0 codebase with no licensing friction, because the paywalled feature is simply code that checks a flag.

**How the author talks about the split.** The README presents it as a charity-funded convenience: the author has "a day job and cannot accept compensation," the money "goes to the same charity (less handling fees)," and he asks supporters to donate rather than reward him. He also writes that he is "frequently assailed by folks who have given the minimum donation to the JustGiving page, and are angry that they have not been subscribed," and has put warnings in every place users pass through. Core features remain free.

**What users say and friction observed.**
- A CWS reviewer wrote the extension ["Does NOT lock essential features behind a paywall, like other extensions"](https://chromewebstore.google.com/detail/amazon-order-history-repo/mgkilgclilajckgnedgjgnfdokkgnibi/reviews) and "unlocks [commercial tools] for a donation to a good cause." This was the only premium comment among the visible reviews. Most reviews praise time saved.
- Friction sits in the issue tracker, not the reviews: [issue 349](https://github.com/philipmulcahy/azad/issues/349) (subscribers cannot log in to a developer-mode install; ExtPay says dev and prod auth are separate), [issue 335](https://github.com/philipmulcahy/azad/issues/335) (billing accepts payment but shows an error page), and [issue 264](https://github.com/philipmulcahy/azad/issues/264) (unsubscribe how-to).
- Scale: 60,000 users; the paid share and revenue are not published **(unverified, not found)**.

## 3. Monetization patterns for browser extensions

### Payment mechanics

- **Chrome Web Store payments are gone.** Google [announced the deprecation on Sept 21, 2020](https://9to5google.com/2020/09/22/google-chrome-paid-extensions-deprecation/); charging through the store stopped Feb 1, 2021, and developers were told to use third-party processors. The [official notice](https://developer.chrome.com/webstore/cws-payments-deprecation) is cited in coverage, but I could not fetch it directly (404).
- **ExtensionPay.** Open-source ExtPay.js plus a hosted service that tracks paid status on top of the developer's own Stripe account. [5% fee per charge](https://extensionpay.com/) on top of Stripe's processing **(the page does not say whether Stripe's fees are extra; competitors say they are)**; monthly, quarterly, yearly and one-time plans; free trials; discount codes; optional Stripe Tax; login by email link instead of license keys. Its own FAQ admits users can remove the payment check from open code, and that only the server-side paid status is tamper-proof. This is what azad uses.
- **Merchant-of-record platforms.** Third-party comparisons list [Paddle and Lemon Squeezy at 5% + $0.50](https://flaviocopes.com/payment-provider-fees/) and Gumroad at 10% base **(unverified: sources disagree on Lemon Squeezy's rate and Gumroad's flat fee; confirm on each vendor's page)**. They handle VAT and sales tax globally, which a solo developer selling worldwide via raw Stripe must do themselves. Lemon Squeezy is now associated with Stripe's own managed-payments product per [one 2026 analysis](https://alexcloudstar.com/blog/merchant-of-record-indie-saas-2026/) **(unverified)**.
- **License keys.** Common flow: purchase, emailed key, user pastes key into the extension, extension validates against a server ([guide](https://chrome-extension-guide.pages.dev/2025/02/16/chrome-extension-monetization-strategies-that-work-2025/)). Offline variants verify a signed token (Ed25519 or RSA) against a public key bundled in the app, which blocks forged keys but not someone deleting the check ([example libraries](https://docs.rs/crate/licenz-core/0.2.0)).
- **Open code and enforcement.** Practical reality across the sources: any check that lives only in client code can be patched. What works is either (a) accepting that and charging for convenience, support, updates or hosted services, or (b) keeping the paid value server-side (data, AI calls, sync).

### Models, with real numbers where public

| Model | Example and numbers | Notes |
|---|---|---|
| Freemium with a narrow paid tier, open source | azad: 60,000 users, about $7.50 to $15/year subscription for shipment/tracking and transaction views, presented as charity fundraising | Revenue not published. Author reports support burden and "tragedy of the commons" abuse |
| Freemium, closed source | GoFullPage listed at [about $10k/month, 4 million users, premium $1/month](https://extensionpay.com/articles/browser-extensions-make-money) (2021 snapshot); Night Eye about $3.1k/month; CSS Scan $100k+ one-time at $69 | Vendor-curated list; none of the eight are open source |
| Freemium at small scale | A [portfolio write-up](https://dev.to/_350df62777eb55e1/real-numbers-freemium-chrome-extension-monetization-after-6-months-5hga): 0.8% free-to-paid across five extensions with 2,900 installs combined, and $31.03 MRR across 38 extensions as of end of July 2026 after the author corrected an earlier $180/month claim | Honest low end. Adding a paywall later "drew negative reviews" |
| ExtensionPay itself | [Indie Hackers milestones](https://www.indiehackers.com/product/extensionpay/25000-payments-in-browser-extensions--MvJwDXPIrwfoFTC2Az4): over $25,000 cumulative developer earnings, about $240/month at one point, mostly from a few extensions | Undated snippet **(unverified)** |
| Subscription for business users | Invoice Radar $140 to $1,400/year; Tailride $158 to $1,658/year with a 10/month free tier | Priced per document volume, targets accountants and small businesses |
| Paywalling a once-free open-source extension | Dark Reader moved to paid subscriptions around mid-2024, reported at about $9.99/year by a [competitor's page](https://turnoffthelights.com/support/?p=17008) that also claims backlash **(unverified, competitor source; the price may apply to corporate users)** | Shows the community reaction risk. Its Open Collective had [raised $123,238 lifetime with an estimated annual budget of about $5,147](https://opencollective.com/darkreader) before moving payments to its own site |
| Donations and sponsorships | GitHub Sponsors [charges nothing on personal-account sponsorships and up to 6% on organization ones](https://docs.github.com/en/sponsors/getting-started-with-github-sponsors/about-github-sponsors). Ranges are small: a [full-time open-source maintainer](https://joshuakgoldberg.com/blog/2025-finances-in-review/) took about $11.8k from Sponsors in 2025 (not an extension author). A 2026 overview estimates [$0 to $50/month for most single-project maintainers](https://www.youngju.dev/blog/culture/2026-05-14-oss-funding-2026-github-sponsors-open-collective-polar-tidelift-ko-fi-developer-sponsorship-deep-dive.en) **(unverified, secondary blog)** | I found no verified income figure for an extension maintainer on Sponsors |
| Paid CWS listing with free source | Not available: CWS payments are retired, so a "paid listing" now means in-app purchase. I found no verified example of a paid listing that also ships free source | Possible variant: free source plus a paid pre-built, supported distribution via a license key **(untested idea)** |
| Open core / hosted service | The extension is free and open; a paid hosted tier adds server-side value. Tailride and Invoice Radar are the closed-source version of this in our space | Needs a backend |

## 4. Options for Keep the Receipt

Facts the options hinge on: Apache-2.0 code is public; no backend; users run it on their own browser; sessions are the user's own, so support is mostly "Amazon changed its page"; the destinations that matter to the target audience (Paperless-ngx, a Downloads folder) skew toward self-hosters who dislike subscriptions.

### Option A: Fully free, with donations (GitHub Sponsors, Open Collective, Ko-fi)
- **Revenue ceiling:** very low. Verified data points are small and none are for extension-only maintainers (see section 3).
- **Local-only, no backend:** untouched.
- **Support burden:** same as any free project; donations do not buy support obligations.
- **Apache-2.0:** no friction.
- **Fit:** strongest alignment with the self-hoster audience; weakest income.

### Option B: Freemium with a small client-side paywall (the azad pattern, ExtensionPay)
- **Revenue ceiling:** modest; azad at 60,000 users does not publish figures, and small freemium portfolios convert under 1% to about 2-5% depending on source.
- **Local-only, no backend:** nearly intact. The extension would contact ExtensionPay's servers for paid status, so the privacy claim becomes "no backend of ours; one payment-status lookup to a third party." That must be disclosed and adds a `connect-src` entry.
- **Support burden:** high relative to revenue. azad's tracker shows billing errors, dev-install login issues, undefined subscription statuses and users angry about the wrong donation path.
- **Apache-2.0:** fine, but the gate is trivially removable. Works only if the paid features are conveniences. Candidates: more destinations (Google Drive, Dropbox), more retailers, or multiple accounts. Gating the core Amazon-to-Paperless path would repeat the "locked essentials" criticism and would clash with the open-source positioning.
- **Fit:** the lowest-effort real revenue path, but the self-hosted audience is least likely to pay for software whose source they can read.

### Option C: Open core with an optional paid hosted add-on
Free local extension; paid hosting for things that need a server, such as always-on collection when the browser is closed, cloud inbox, or AI categorization.
- **Revenue ceiling:** highest of the options that stay open source, with recurring revenue that can't be patched out.
- **Local-only, no backend:** broken for paying users by definition, and it creates storage of financial documents (breach and compliance exposure, privacy policy, tax handling).
- **Support burden:** the highest; you become a service operator. The design doc's "runs locally, no backend" positioning would need a second product line.
- **Apache-2.0:** fine, since the paid part is a separate service.
- **Fit:** only worth it if the project chooses to become a business. Invoice Radar and Tailride already occupy this space at $140 and up per year.

### Option D: Paid, supported distribution of the same open code
The source stays free on GitHub and users can build it. A Web Store listing (or an auto-updating build) carries a one-time or annual license for people who want a maintained version, via ExtensionPay or a merchant-of-record platform, with a free tier cap (for example, the last N invoices or one retailer).
- **Revenue ceiling:** modest. Selling convenience to people who could build it works for tools with non-technical buyers, but this audience is technical.
- **Local-only, no backend:** same as Option B, one license lookup.
- **Support burden:** moderate; the "pay for updates when Amazon changes its pages" story is honest and defensible, because the maintenance is the real cost.
- **Apache-2.0:** fine, though some contributors dislike a paid build of free code.
- **Fit:** reasonable if reliability work is the main cost. No verified example found.

### Option E: Business tier for small businesses and bookkeepers
Same extension, with paid features for multi-account, Business accounts, retention policies, and accounting-tool destinations (QuickBooks, Xero), priced per seat or per year.
- **Revenue ceiling:** the highest per customer; Tailride and Invoice Radar price here at $158 to $1,658/year.
- **Local-only, no backend:** can stay local if accounting exports go direct to the user's own accounts; OAuth to those tools usually needs a registered app and often a server for the token exchange.
- **Support burden:** high; businesses expect support and accounting integrations break often.
- **Apache-2.0:** fine.
- **Fit:** the commercial market exists, but it is crowded with funded competitors and pulls the product away from its self-hosting audience.

### Recommendation (input, not a decision)

Start with **Option A plus a stated plan for Option B later**. Specifically:
1. Ship v1 fully free, local-only, no payment code, with GitHub Sponsors on the repo. This keeps the strongest differentiator (open, no backend, Paperless-ngx) intact, which is the point where competitors are weakest, and matches the audience.
2. Decide on a paid tier only after seeing real usage. If one is added, follow azad's lesson: keep the core free, and gate only extras that cost real maintenance (additional destinations such as Google Drive or Dropbox, extra retailers, multi-account). Use ExtensionPay or a merchant-of-record platform rather than building billing, accept that the check is patchable, and disclose the license lookup in the privacy text.
3. Avoid Option C unless the project deliberately becomes a service business.

Rationale: Amazon markup churn is the dominant ongoing cost, so any monetization must fund that work, and the verified evidence suggests donations alone will not. But a paywall added before there is an audience adds support cost (azad's tracker) with little revenue to offset it.

## Gaps and things to verify before relying on this

- azad's revenue, subscriber count and conversion are not public; none found.
- Verified income for an open-source extension maintainer on Sponsors or Open Collective was not found.
- Lemon Squeezy and Gumroad current fee structures conflict between sources; check vendor pages.
- Invoice Radar's Paperless-ngx sync may or may not be automatic; its docs only describe manual export.
- Tailride's Dropbox/Paperless support and exact privacy claims were not verified from primary sources beyond its own pages.
- Reddit threads were reached only through search summaries and mirror sites; r/selfhosted and r/personalfinance produced nothing usable.
- The CWS developer deprecation page returned 404 on fetch; the deprecation facts rest on news coverage.
