# Brief: build tooling and PDF library

Research task that produces `docs/research/build-and-pdf.md`. Kept in the repo so it can be re-run.

## Context

Keep the Receipt is a Chrome MV3 extension in TypeScript (see `docs/design.md`): a service
worker, an offscreen document for `DOMParser`, a popup and an options page. It builds its own
receipt PDF from parsed order data (items table, totals, header, footer), with a real text layer
so Paperless-ngx doesn't need to OCR it. The PDF is generated in the service worker or the
offscreen document. Constraints: few moving parts, free, maintained, good for contributors who
want to learn from the repo.

## Questions

1. **Build tooling.** Compare for an MV3 extension of this size:
   - plain esbuild (+ a small script that copies `manifest.json` and HTML, maybe `--watch`),
   - Vite + CRXJS (`@crxjs/vite-plugin`), status of its v2 release and maintenance,
   - WXT (wxt.dev),
   - Plasmo.
   For each: MV3 service worker + offscreen document support, HMR/reload in dev, how many
   dependencies it pulls in, maintenance (last release, open-issue health), Firefox support
   (later goal), and how it handles `manifest.json` (hand-written vs. generated). Also: test
   runner that fits (Vitest with jsdom/happy-dom for parser tests against HTML fixtures).
2. **PDF library.** Compare pdf-lib, jsPDF (+ jspdf-autotable), pdfmake, and anything newer
   worth considering:
   - minified+gzipped size, maintenance (last release, maintainer activity),
   - works in an MV3 service worker (no DOM, no `window`, no `eval` under the extension CSP),
   - text layout: tables, line wrapping, multi-page flow; how much code we'd write ourselves,
   - fonts: the standard 14 fonts only cover WinAnsi; Amazon item names contain non-Latin
     characters, emoji, curly quotes, ™ etc. How does each embed a TTF and subset it? Size cost
     of embedding e.g. Noto Sans or Inter?
   - text extraction quality (does the output give clean selectable/searchable text, e.g. with
     `pdftotext`)?
   - PDF/A or metadata (title, author, creation date, keywords) support.
3. Any CSP issues: MV3 forbids remote code and `unsafe-eval`; flag libraries that use `eval` or
   `new Function` (some PDF libs do for fonts or templates).

## Method

Use primary sources: npm registry (`npm view <pkg> time dist.unpacked-size`), bundlephobia or
pkg-size figures, GitHub repos (releases, issues), the libraries' docs. If useful and quick, build
a tiny throwaway test in the scratch dir (not in the repo): render a two-item receipt with a
non-Latin item name in a service-worker-like environment (Node without DOM) and check the text
with `pdftotext` if available. Put scratch work under the scratchpad directory given to you.

## Output

Write `docs/research/build-and-pdf.md`: a comparison table for each question, every claim cited
with a link and what the source is, **(unverified)** where not confirmed, and a recommendation
for each with reasons. Use American English.
