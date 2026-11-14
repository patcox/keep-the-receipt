# Brief: prior-art and monetization research

This is the brief for the research task that produces `docs/research/prior-art.md`. It's kept in the
repo so the task can be re-run or extended.

## Context

Keep the Receipt is an open-source (Apache-2.0) Chrome extension. Once a day it pulls the user's
order invoices from retailers (Amazon US first), using the session the user is already logged
into. It renders each one as a searchable PDF and sends it to an output the user chooses: a
Downloads subfolder, a Paperless-ngx server, a webhook, later Google Drive or Dropbox. It runs
locally and has no backend of its own. See `docs/design.md`.

## Questions

1. **Prior art: what already does some or all of this?** Search widely. An earlier pass missed a
   direct competitor because it searched only for "invoice downloader" wording. Cover:
   - Wording: receipt, invoice, order, purchase, order history, expense, web clipper.
   - Destinations: Google Drive, Dropbox, OneDrive, Paperless-ngx, email, accounting tools
     (QuickBooks, Xero, Expensify, YNAB, Monarch).
   - Places: the Chrome Web Store, Firefox Add-ons, GitHub, npm/PyPI, Reddit
     (r/paperlessngx, r/selfhosted, r/MonarchMoney, r/personalfinance).
   - Already known, so verify and extend: philipmulcahy/azad (Amazon Order History Reporter),
     alex-peck/monarch-amazon-sync, Monarch's "Retail purchase sync", SaveReceipts Web Clipper,
     Receipt Wallet, Tailride, Order History Downloader, Invoice Radar, oliverswitzer/
     amazon-downloader-extension, bilal-psd/amazon-order-exporter, the Playwright scrapers
     (dcwangmit01, chrisns, ayelder), Staxxer (sellers only).

   For each, record: what it does; automatic or click-to-run; which retailers and outputs;
   open source and license; local-only or a cloud service; price; size signals (Web Store users,
   rating, GitHub stars, last update); anything notable about how it works (permissions, how it
   gets PDFs).
2. **azad's premium tier in detail.** What it costs, what's free and what's paid, how payment and
   license checks work technically (read the code at github.com/philipmulcahy/azad), how the
   author talks about the split in the README, and what users say about it.
3. **Monetization patterns for browser extensions, especially open-source ones.** Cover:
   freemium feature gates, subscriptions vs one-time payments, open core (open extension + paid
   hosted service), donations and sponsorships (GitHub Sponsors, Open Collective), paid Web
   Store listings with free source on GitHub, and B2B/business tiers. Also the payment
   mechanics: Chrome Web Store payments were retired, so what's used now (ExtensionPay, Stripe,
   Paddle, Lemon Squeezy, Gumroad) and how license checks work in an extension whose code anyone
   can read. Give real examples with numbers where public.
4. **Which of these fit Keep the Receipt?** Lay out 3–5 options with tradeoffs (the revenue
   ceiling, how much it compromises "local-only, no backend", support burden, and how it sits
   with an Apache-2.0 codebase). Recommend one, but present it as input, not a decision.

## Output

- Write `docs/research/prior-art.md`, with a comparison table for question 1 and a section for
  each of questions 2–4.
- Cite every claim with a link. Mark anything you couldn't verify.
- This is a public repo: no personal data and no details about anyone's own setup.
- Don't commit. Report back with a 10-line summary and the three findings that matter most.
