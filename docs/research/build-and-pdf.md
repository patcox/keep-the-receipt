# Research: build tooling and PDF library

Answers [`build-and-pdf-brief.md`](build-and-pdf-brief.md). Researched 2026-11-14. Versions, dates and
sizes below are as of that day. Claims marked **(unverified)** weren't confirmed.

Sources used throughout:
- npm registry metadata (`https://registry.npmjs.org/<pkg>`): version, publish time, unpacked size, dependencies.
- GitHub repo pages and API (releases, commits, open-issue counts, security advisories).
- Throwaway tests run on 2026-11-14 in a scratch directory outside this repo (details under
  "What I ran"). Bundle sizes are esbuild `--bundle --minify --platform=browser` output, gzipped
  with `gzip -9`. They are my measurements, not bundlephobia numbers.

## Summary and recommendations

| Question | Recommendation |
|---|---|
| Build tool | **Plain esbuild** with a short build script and a hand-written `manifest.json`. Revisit **WXT** if Firefox becomes a real v1.x goal. |
| Test runner | **Vitest** with happy-dom (or jsdom) for parser tests against HTML fixtures. |
| PDF library | **pdfmake** (0.3.x). Runner-up: jsPDF + jspdf-autotable. |
| Fonts | Bundle **Noto Sans** (OFL) Regular and Bold as per-script WOFF files from Fontsource: Latin, Latin-ext, Cyrillic, Greek, Vietnamese. About 118 KB per weight, 240 KB total, subset again by the PDF library per document. CJK and emoji are out of v1 (see below). |

## 1. Build tooling

### Comparison

| | esbuild (plain) | Vite + CRXJS | WXT | Plasmo |
|---|---|---|---|---|
| Latest version / date | 0.28.2, 2026-08-08 ([npm](https://registry.npmjs.org/esbuild)) | `@crxjs/vite-plugin` 3.0.0, 2026-09-24 ([npm](https://registry.npmjs.org/@crxjs%2Fvite-plugin)); Vite 8.3.8, 2026-11-11 ([npm](https://registry.npmjs.org/vite)) | 0.21.5, 2026-10-26 ([npm](https://registry.npmjs.org/wxt), [releases](https://github.com/wxt-dev/wxt/releases)) | 0.90.5, 2025-05-17 ([npm](https://registry.npmjs.org/plasmo), [releases](https://github.com/PlasmoHQ/plasmo/releases)) |
| Runtime dependencies (direct) | 0 | CRXJS: 14 (rxjs, rollup, fs-extra, ...), plus Vite: 5 (rolldown, lightningcss, ...) | 28 (c12, chokidar, linkedom, ...), peer: vite | 30 (`@parcel/*`, `sharp`, `inquirer`, ...) |
| Maintenance signal | Active, frequent releases | CRXJS 2.x released steadily Jun-Jul 2026, 3.0.0 in Sep 2026; last commit seen 2026-11-02; 18 open issues ([repo](https://github.com/crxjs/chrome-extension-tools)) | Last commit 2026-11-13; 189 open issues, 10.7k stars ([repo](https://github.com/wxt-dev/wxt)) | Last commit I could find is 2025-05-17 ("chore: bump packages"); 370 open issues ([repo](https://github.com/PlasmoHQ/plasmo)). Looks dormant. |
| MV3 service worker | Yes, as an ordinary entry point | Yes, background worker with HMR client (3.0.0 notes mention reconnect after dev-server restart, [changelog](https://github.com/crxjs/chrome-extension-tools/blob/main/packages/vite-plugin/CHANGELOG.md)) | Yes (built-in `background` entrypoint) | Yes |
| Offscreen document | Just another HTML page + script you list as entry points | Works as an extra HTML page; the old "add offscreen permission" issue is closed ([#712](https://github.com/crxjs/chrome-extension-tools/issues/712)). **(unverified in practice)** | No first-class entrypoint type; use an "unlisted page" (`/{name}.html`, [docs](https://wxt.dev/guide/essentials/entrypoints.html)). Offscreen support issue closed ([#963](https://github.com/wxt-dev/wxt/issues/963)). **(unverified in practice)** | Open RFC since 2024 ([#527](https://github.com/PlasmoHQ/plasmo/issues/527)); bug open ([#885](https://github.com/PlasmoHQ/plasmo/issues/885)) |
| Dev reload / HMR | `context.watch()` rebuilds; you reload the extension by hand (or via a `chrome.runtime.reload()` helper). No HMR. | HMR for pages and content scripts, worker reload | Dev server with reload; opens a browser | HMR via Parcel |
| `manifest.json` | Hand-written file, copied | Hand-written, wrapped in `defineManifest` (CRXJS reads it and rewrites paths) | **Generated** from entrypoint options plus a config block ([docs](https://wxt.dev/guide/essentials/entrypoints.html)) | Generated from `package.json` + conventions |
| Firefox (later goal) | Possible: add a second manifest and build target; you maintain it | 3.0 line added Firefox E2E coverage (commit 2026-10-26, [repo](https://github.com/crxjs/chrome-extension-tools/commits/main)); less central than in WXT | First-class: `wxt -b firefox`, MV2/MV3 flags ([docs](https://wxt.dev/guide/essentials/target-different-browsers.html)) | Firefox target exists **(unverified)** |
| Learning curve for a contributor reading the repo | Lowest: one build script | Medium: Vite config + plugin | Medium-high: framework conventions (`entrypoints/`, `defineBackground`, auto-imports) | High |

### Notes

- **Test runner needs Vite regardless.** Vitest 5.1.0 ([npm](https://registry.npmjs.org/vitest)) lists `vite` as a peer
  dependency and builds on it. So "plain esbuild" does not mean "no Vite in `node_modules`". It means the extension build
  itself has no Vite plugin layer between the source and the shipped files.
- **DOM for parser tests.** Both happy-dom 21.0.4 ([npm](https://registry.npmjs.org/happy-dom), 7 deps) and jsdom 31.0.1
  ([npm](https://registry.npmjs.org/jsdom), 18 deps) provide `DOMParser`. In production, parsing runs in the offscreen document with the
  browser's real `DOMParser`. Tests are a stand-in, so pick the smaller, faster one (happy-dom) and keep jsdom as the fallback if a fixture
  parses differently. I did not run the real fixtures through either. **(unverified)**
- **Offscreen on Firefox.** Chrome's offscreen API doesn't exist in Firefox, where the background script can use `DOMParser` directly.
  So a Firefox port needs a code path anyway, whatever the build tool. **(unverified: from my own knowledge, not re-checked here)**
- **Why not Plasmo:** last release May 2025, 30 dependencies including Parcel and `sharp`, open offscreen RFC. It fails the
  "maintained, few moving parts" constraint.
- **Why not CRXJS or WXT now:** both are healthy. CRXJS was in limbo for a while before the 2.x/3.0 releases, and its v3 is only two months
  old. WXT is the strongest framework, with generated manifest, Firefox, and a good dev loop, but it brings 28 dependencies and
  its own conventions for a project with four entry points. If the Firefox goal gets a date, switch to WXT then; the source (a
  service worker, an offscreen page, a popup, an options page) ports over with modest changes.

### Recommendation

Plain **esbuild** plus a ~40-line build script (bundle four entry points, copy `manifest.json`, HTML and icons to `dist/`, `--watch` in dev).
It has zero runtime dependencies, the manifest is a plain file contributors can read, and the build script is itself a good
learning artifact. Cost: no HMR (reload the extension after a rebuild) and Firefox is on us. Use **Vitest + happy-dom**
for tests.

## 2. PDF library

### Comparison

All figures: sizes are esbuild-minified browser bundles of the library as imported by the test code (own measurements, 2026-11-14).

| | pdf-lib | jsPDF + jspdf-autotable | pdfmake | Others considered |
|---|---|---|---|---|
| Version / last release | 1.17.1, **2021-11-06** ([npm](https://registry.npmjs.org/pdf-lib)); last commit 2021-11 ([repo](https://github.com/Hopding/pdf-lib)). Unmaintained. Active fork `@cantoo/pdf-lib` 2.11.1, 2026-09-15 ([npm](https://registry.npmjs.org/@cantoo%2Fpdf-lib), [repo](https://github.com/cantoo-scribe/pdf-lib)). | jsPDF 4.2.1, 2026-03-17 ([npm](https://registry.npmjs.org/jspdf), [repo](https://github.com/parallax/jsPDF)); autotable 5.0.9, 2026-10-15 ([npm](https://registry.npmjs.org/jspdf-autotable)) | 0.3.14, 2026-11-08 ([npm](https://registry.npmjs.org/pdfmake)); last commit 2026-11-14 ([repo](https://github.com/bpampuch/pdfmake)) | PDFKit 0.20.2, 2026-08-30 ([npm](https://registry.npmjs.org/pdfkit)); `@pdfme/generator` 6.2.8, 2026-10-21 ([npm](https://registry.npmjs.org/@pdfme%2Fgenerator)); `@react-pdf/renderer` 4.9.0 ([npm](https://registry.npmjs.org/@react-pdf/renderer)) |
| Bundle, min / gzip | 438 KB / 183 KB, plus **fontkit 987 KB / 383 KB** needed for custom fonts. Total **1.38 MB / 552 KB**. Fork: 1.60 MB / 641 KB. | **853 KB / 259 KB** (jsPDF alone 821 KB / 249 KB) | **811 KB / 299 KB** (includes PDFKit and fontkit) | PDFKit is what pdfmake wraps. pdfme is a template/designer system. react-pdf needs React. Not pursued. |
| Runs without DOM / `window` | Yes (Node test, no `window`) | Yes in Node build; the browser build's HTML/canvas features (`html()`, canvg, html2canvas, DOMPurify) are optional and not used **(unverified in an actual Chrome service worker)** | Yes. The browser bundle ran in Node with no `window`/`document`. | PDFKit browser build exists |
| `eval` / `new Function` (CSP) | One `Function(...)` call in a `Function.prototype.bind` polyfill, reached only if `bind` is missing, so dead in Chrome. Test passed with code generation from strings disabled. | One `Function("return this")()` global fallback, behind `globalThis` check, so not reached. Test passed with code generation disabled. | `new Function("return this")` global fallback inside `try`, and a `Function("return this")()` fallback; neither runs when `globalThis` exists. Test passed with code generation disabled. | not tested |
| Layout: tables, wrapping, pagination | **None.** Drawing primitives only. You write wrapping (`widthOfTextAtSize`), column layout and page breaks. My 25-line wrapper handled one page; a real multi-page items table is more. | Wrapping and page breaks for tables via autotable. Plain `text()` wraps with `splitTextToSize`. Header/footer by hand or autotable hooks. | **Declarative**: tables with column widths, wrapping, multi-page flow, repeating header rows, page header/footer functions, all built in. | pdfme: template-driven, aimed at form-style docs |
| Custom TTF + subsetting | `registerFontkit` + `embedFont(bytes, {subset: true})`; accepts TTF and WOFF (WOFF tested). Output for the test receipt: **9.2 KB**. | `addFileToVFS` + `addFont(..., 'Identity-H')`; font data must be base64. Output for the test receipt: **120.6 KB** with Noto Sans, 290.5 KB with Noto Sans JP, so it does not subset aggressively. | Fonts from a virtual file system (base64) or files; subsets automatically via PDFKit. Output: **11.4 KB** (TTF), 4.5 KB with Fontsource Latin WOFF. | |
| Font fallback for mixed scripts | None. I split runs by glyph coverage myself. | None; one font per `text()` call or table cell. | None, but a cell accepts an array of runs with different `font` values; I split by glyph coverage and it worked. | |
| Text extraction | Clean (checked with pdfjs, see below) | Clean, but the autotable cell with the Japanese name rendered with the Noto Sans font lost the CJK characters, as expected for missing glyphs | Clean, wraps without inserting stray characters | |
| Metadata | `setTitle/Author/Subject/Keywords/CreationDate`. No PDF/A support. Verified title/author/keywords read back. | `setProperties` (title, author, keywords, creator); an XMP metadata plugin exists. No PDF/A claim. Read back OK. | `info: {title, author, keywords, creationDate, ...}`; docDefinition also takes `subset: 'PDF/A-2b'` and `tagged: true`, passed to PDFKit ([source](https://github.com/bpampuch/pdfmake/blob/master/src/Printer.js)). Read back OK. PDF/A output generated but **not validated** (no veraPDF) and I could not find the `pdfaid` marker in it. **(unverified)** | PDFKit documents PDF/A-1/2/3 support in its [changelog](https://github.com/foliojs/pdfkit/blob/master/CHANGELOG.md) |
| Security record | none noted | **Many advisories in 2026**: HTML injection, PDF object injection (`addJS`, annotations), AcroForm JS injection, GIF DoS ([advisory list](https://github.com/parallax/jsPDF/security/advisories)). These are in modules we wouldn't use, but it signals the attack surface of a library that takes arbitrary strings. | none noted | |

### Notes

- **pdf-lib is the wrong bet** despite the popularity: no release since 2021, the heaviest bundle once fontkit is added, and no layout engine, so
  we would write and maintain a table and pagination routine. The maintained fork `@cantoo/pdf-lib` fixes the maintenance problem but is
  heavier (641 KB gzipped) and still has no layout.
- **jsPDF + autotable** is the closest competitor: smallest gzipped bundle (259 KB), a real table engine and a long history. Its
  weaknesses here are per-PDF size (the font embeds mostly unsubset: 120 KB per receipt before any CJK) and the advisory record.
- **pdfmake** gives layout for free, so the receipt becomes a short declarative document definition. It subsets fonts, produces tiny files
  and is the most actively committed project of the set. Its weaknesses: a larger gzipped bundle than jsPDF, a declarative API with
  its own idiom to learn, and no automatic font fallback (handled below).
- **Newer options.** `@pdfme/generator` is template-oriented and heavier on concepts for a four-section receipt. `@react-pdf/renderer`
  needs React. Neither fits "few moving parts."

### Recommendation

**pdfmake.** Reasons: tables, wrapping and pagination without us writing a layout engine; automatic subsetting with small outputs; PDF
metadata; a PDF/A option to evaluate later; actively maintained (release 2026-11-08); no `eval` path actually reached, and the
browser bundle ran in a no-DOM, code-generation-disabled test. If the 299 KB gzipped weight or API shape bother a reviewer, jsPDF +
autotable is the fallback, accepting larger files.

## 3. Fonts: non-Latin text, quotes, ™, emoji

The standard 14 fonts cover only WinAnsi, so we need an embedded font. The three libraries all embed a TTF; pdf-lib and pdfmake subset it per document.

### Measured sizes

| Font file | Size |
|---|---|
| Noto Sans Regular, full TTF (Latin, Greek, Cyrillic, Vietnamese; [package](https://www.npmjs.com/package/@expo-google-fonts/noto-sans), MIT AND OFL-1.1) | 629 KB raw, 296 KB gzipped |
| Inter Regular, full TTF | 342 KB raw, 162 KB gzipped |
| Noto Sans JP Regular, full TTF | 5.73 MB raw, 3.30 MB gzipped |
| Fontsource Noto Sans WOFF, Regular, five script files (latin 16.7 KB, latin-ext 78.4 KB, cyrillic 9.0 KB, greek 9.9 KB, vietnamese 7.1 KB; [package](https://www.npmjs.com/package/@fontsource/noto-sans), OFL-1.1) | **121 KB** |
| Same, Bold | **126 KB** |
| Fontsource Noto Emoji (monochrome) WOFF2, one file | 476 KB |

### What worked in tests

- A receipt with curly quotes, an em dash, ™, "é", Cyrillic and Japanese produced correct selectable text in pdf-lib, jsPDF and pdfmake.
- The Fontsource **WOFF** Latin file embedded and extracted correctly in both pdf-lib and pdfmake (output 4.5 KB for a one-line pdfmake document).
- No library does glyph fallback. The approach that worked: before drawing, split each string into runs by which font has the glyph
  (fontkit's `hasGlyphForCodePoint`), and drop characters no font has. In production, compute the per-font coverage sets at build time
  instead of loading fontkit just to ask.
- **Emoji.** None of Noto Sans or Noto Sans JP has them; my test dropped the two emoji and kept the rest of the name. A color emoji font
  (CBDT/COLRv1) won't render through these libraries **(unverified)**. Monochrome Noto Emoji exists (476 KB) but I didn't test it in a PDF.

### Recommendation

- v1 ships **Noto Sans Regular + Bold, Latin / Latin-ext / Cyrillic / Greek / Vietnamese WOFF, about 247 KB in the extension package**
  (pdfmake then subsets them per document).
- **CJK, Arabic, Hebrew, Indic scripts and emoji are out of v1.** Unrepresentable characters are dropped, or replaced by a placeholder,
  in the PDF only. The original invoice HTML and the JSON record next to it keep the exact text, so nothing is lost. Say so in the
  docs. If CJK shoppers matter later, add an optional font pack the user enables, since a full CJK font is several MB. How to deliver
  it (bundled and optional vs. fetched at runtime) is a question for later.
- Bundled font files are *data*, not remote code, so MV3's remote-code rule isn't triggered. **(unverified: the policy text was not re-read this session; see the MV3 improve-security guide at developer.chrome.com)**

## 4. CSP check summary (MV3: no remote code, no `unsafe-eval`)

| Library | Finding |
|---|---|
| pdf-lib | One `Function(...)` call, in the `Function.prototype.bind` polyfill. Not reached in Chrome. |
| jsPDF | One `Function("return this")()` global fallback behind a `globalThis` check. Not reached. Optional lazily imported modules (html2canvas, canvg, DOMPurify) aren't used by us. |
| pdfmake | `new Function("return this")` in a `try` fallback, plus one `Function("return this")()` fallback; both guarded by `globalThis`. |
| Method | Searched the esbuild output for `eval(`, `new Function(`, `Function("`. Then ran the pdf-lib and jsPDF tests from `node_modules` and the pdfmake bundle with `node --disallow-code-generation-from-strings`, so any reached `eval`/`Function` would throw. All passed. This approximates, but is not identical to, Chrome's extension CSP. A real load of the built extension in Chrome is still the final check. **(unverified)** |

## What I ran (and what I couldn't)

- Scratch directory (not in the repo): `/tmp/claude-1000/-home-pat-claude-projects-keep-the-receipt/0cfe1f08-7242-5586-b78f-8ae151aa4504/scratchpad/pdf/`.
  It held one script per library, rendering the same two-plus-one item receipt (a long English name with quotes, em dash, ™ and "&"; a
  Japanese + accented Latin + Cyrillic name; a name with two emoji).
- **`pdftotext` is not installed here.** I extracted text with `pdfjs-dist` in Node instead. That checks that the text layer exists and reads
  in a sensible order, but it is not the same extractor Paperless-ngx uses. Re-check with `pdftotext` before shipping. **(unverified with pdftotext)**
- Everything ran in Node, not a Chrome service worker or offscreen document. Real-browser behavior (memory, time, `fetch` of font files from
  the extension package) **(unverified)**.
- I didn't test Firefox, CRXJS, WXT or Plasmo builds, only read their release data, repos and docs. WXT docs for dev-reload behavior could
  not be fetched.
- Open-issue counts are from the GitHub API's `open_issues_count`, which includes pull requests.
- No PDF/A validator (veraPDF) was available, so PDF/A conformance is untested.
