# Keep the Receipt

**Status: design stage. Nothing to install yet.**

A Chrome extension that keeps your receipts for you. Once a day it checks your recent orders in
the store sessions you're already logged into (Amazon US first), saves each invoice as a
searchable PDF, and sends it where you choose: a Downloads folder, a
[Paperless-ngx](https://docs.paperless-ngx.com/) server, a webhook, and later Google Drive,
Dropbox and others.

- **Automatic.** No button to press. It runs while Chrome is open and catches up after missed days.
- **Local.** No account, no backend, no telemetry. It never sees your store password.
- **Yours.** Every receipt is a PDF, plus the store's original HTML and a JSON record.

Why it exists and how it will work: [`docs/design.md`](docs/design.md). What else is out there:
[`docs/research/prior-art.md`](docs/research/prior-art.md).

Not affiliated with Amazon or any other retailer.

## License

[Apache-2.0](LICENSE).
