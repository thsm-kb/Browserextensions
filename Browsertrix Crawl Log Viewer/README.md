# Browsertrix Crawl Log Viewer

**made by Thomas Smedebøl, Royal Danish Library**

Browsertrix Crawl Log Viewer is a Chrome/Chromium extension that turns Browsertrix crawl logs into a practical QA and diagnostics interface for web-archive work.

Instead of reading Browsertrix newline-delimited JSON by hand, the extension intercepts **Download Logs**, opens the log locally, and provides sortable events, page/resource statistics, MIME grouping, domains, URL provenance, soft-404 detection, runtime diagnostics, blocking/throttling signals, and an optional on-device Gemini Nano summary.

> Current version: **1.6.1**

## Why this exists

Browsertrix logs contain a great deal of useful crawl information, but important signals are spread across thousands of individual log events. This viewer is designed to help a curator or crawl engineer answer questions such as:

- Did the crawl complete successfully?
- Which pages and resources were captured?
- What consumed most of the crawl data?
- Which domains supplied the resources?
- Are there signs of blocking, rate limiting or throttling?
- Did Browsertrix hit a crawl limit?
- Does the log show RAM, CPU or storage pressure?
- Where did a resource or page URL originate?
- Does the target appear to use soft 404 responses?
- What are the few findings that deserve human attention?

The raw log table remains the forensic source of truth. Derived findings distinguish deterministic facts from heuristics and inferred relationships.

## Features

### Crawl log viewer

- Intercepts Browsertrix **Download Logs** on the configured Browsertrix Cloud host.
- Can also open local `.log`, `.txt`, `.json` and NDJSON files.
- Streams remote logs rather than waiting silently for the entire download.
- Prominent loading state with spinner/progress indication.
- Shows percentage and received/total bytes when a reliable `Content-Length` is available.
- Falls back to an indeterminate progress bar plus received byte count when total size is unknown.
- Keeps the original event `details` JSON available for inspection.

### Sort, filter and inspect

The main table exposes useful Browsertrix fields in a normalized form, including:

- timestamp
- level
- context
- message
- HTTP status
- worker
- resource/file type
- MIME group
- size
- domain/hostname
- URL/resource
- page
- origin / discovered from
- raw details

Columns are sortable. Search and filters update the displayed rows and the statistics derived from the current selection.

### Raw log view

A **Raw log** tab shows the original log text as code:

- line numbers identical to the table's source line, with a link from every expanded table row;
- JSON syntax colouring, with `logLevel` values and messages emphasized;
- plain-text or **regular-expression** search, optional case sensitivity, next/previous match navigation;
- **Only matching lines** (grep-style) and **Wrap lines** toggles, plus Go to line;
- gutter marks for warn, error/fatal and malformed (non-JSON) lines.

Search runs in a Web Worker. A regular expression that makes no progress for 8 seconds (catastrophic backtracking) is stopped instead of freezing the viewer. Rendering is block-virtualized, so logs with hundreds of thousands of lines stay responsive.

### Pages and resources

**Pages** are a first-class filter option rather than being treated as extensionless resources.

Resource statistics are URL-deduplicated and can include:

- unique resources
- resources with a known size
- combined known size
- average size
- median size
- largest resource

The viewer does not equate the number of log rows with the number of unique resources.

### URL-aware file type detection

File type detection is deliberately conservative:

1. Prefer MIME/resource metadata logged by Browsertrix.
2. Otherwise parse the URL with `new URL()`.
3. Inspect only the final filename component of the URL **pathname**.
4. Use only recognized extensions.
5. Unknown suffixes remain unknown.

Hostnames and top-level domains are never treated as file extensions. For example, `.dk`, `.com` and `.org` cannot become file types simply because they occur at the end of a hostname.

### Domain column

Each relevant row exposes the actual resource hostname, for example:

- `www.example.dk`
- `cdn.example.dk`
- `media.example.net`

Subdomains are intentionally preserved because CDN and third-party hosts are often important during crawl QA.

## MIME grouping

Browsertrix may expose MIME/content-type information directly. When it does, the viewer uses that information before extension inference.

MIME names and top-level media types are based on the **IANA Media Types registry**:

- https://www.iana.org/assignments/media-types/media-types.xhtml
- https://www.iana.org/assignments/top-level-media-types/top-level-media-types.xhtml

IANA defines registered MIME types; it does not define archival QA groupings such as “Office / documents”. The viewer therefore adds a transparent functional grouping layer intended for crawl analysis.

Current groups include:

| Group | Examples |
|---|---|
| Web pages | HTML/XHTML page material |
| Images | JPEG, PNG, GIF, WebP, SVG, AVIF, etc. |
| Office / documents | PDF, Word, Excel, PowerPoint, OpenDocument, RTF, etc. |
| Audio files | ordinary audio resources such as MP3, OGG, WAV, FLAC |
| Video files | ordinary video resources such as MP4, WebM, MOV |
| Streaming manifests / playlists | HLS, DASH and similar manifests/playlists |
| Streaming audio | media identified specifically as streaming audio |
| Streaming video | media identified specifically as streaming video |
| Streaming media segments | transport/media segments such as TS/M4S where identifiable |
| Archives / compressed | ZIP, TAR, GZIP, 7z, RAR and related formats |
| Structured data | JSON, XML and related machine-readable data |
| Scripts / web code | JavaScript and related code resources |
| Stylesheets | CSS |
| Fonts | WOFF/WOFF2/TTF/OTF and related types |
| E-books | EPUB and related formats |
| 3D / models | registered model media types and known formats |
| Messages / multipart | message/* and multipart/* material |
| Text | other text/* resources |
| Application / other | known application/* resources not assigned to a more useful QA group |
| Binary / unknown | generic binary content |
| Unclassified | insufficient information |

### Streaming classification is conservative

The viewer does **not** classify every `audio/*` or `video/*` response as streaming.

Streaming requires an actual streaming-specific signal such as an HLS/DASH playlist/manifest MIME type, a recognized segment MIME type, or a conservative URL-extension signal such as `.m3u8`, `.mpd`, `.m4s` or transport-stream segments.

An ordinary MP3 or MP4 remains an audio/video file unless the available evidence says otherwise.

## Soft-404 detection

The viewer can probe a deliberately nonexistent path on the first-seed origin:

```text
<first-seed-origin>/browsertrixsoft404detect
```

If that path returns HTTP 200, the host may be using a soft-404 pattern.

The viewer then looks for possible matching resources on the same host using normalized type and approximately equal size. The current tolerance is:

- **±8%**, or
- **±2 KB**,

whichever is larger.

Potential matches are labelled **SOFT404?**. They are suspects for review, not confirmed missing pages.

The test requests runtime host permission only for the concrete target host.

## URL provenance

The viewer attempts to explain where a page or resource came from.

Provenance confidence is explicit:

- **EXACT** — Browsertrix logs `details.page` for a resource, or explicit source/referrer/parent metadata is present.
- **INFERRED** — for a single-worker crawl, a queued page URL may be associated with the currently active page when timing makes that relationship defensible.
- **INITIAL** — URLs queued before ordinary page processing begins, typically seeds/initial URLs.
- **UNKNOWN** — no defensible origin is available.

Timing-based inference is never presented as exact, and the viewer does not guess page parentage in multi-worker crawls without explicit metadata.

## Crawl diagnostics

The viewer calculates deterministic QA facts before any AI interpretation.

### Blocking and throttling

Signals include parsed HTTP statuses and explicit log evidence such as:

- 401 / 403 / 407
- 429 rate limiting
- relevant 503 patterns
- WAF/challenge/block messages
- repeated navigation/fetch failures
- explicit throttling/rate-limit messages

A log message saying Browsertrix is **checking** for Cloudflare blocking is not itself treated as evidence of blocking.

### Runtime and limits

Where the log exposes the data, diagnostics can include:

- RSS and heap memory values of the Browsertrix Crawler Node.js process (debug-level `memoryStatus`; browser memory is not logged)
- peak/ending memory and suspicious memory growth
- explicit CPU telemetry
- explicit storage/quota/write/WARC-upload errors and disk-utilization stops
- crawl limits and whether they were hit, matched against Browsertrix's own stop messages (size, time, failed-page and disk-utilization thresholds, page limit); per-page "retry limit reached" is not a crawl limit, and a logged page limit of 0 means none was configured
- browser/worker crashes
- OOM/target/session failures

CPU pressure is not guessed from slowness, and an internal per-resource size threshold is not automatically described as a storage quota.

## Gemini Nano report

The viewer can optionally use Chrome's built-in **LanguageModel / Prompt API** when available.

Design principle:

> **JavaScript calculates; Gemini interprets.**

The full raw crawl log is not passed to Gemini. The extension first builds a compact deterministic fact object containing relevant crawl metadata, response-status counts, warnings/errors, blocking signals, limits, memory/CPU/storage telemetry when present, resource/MIME/domain summaries, large resources and soft-404 findings.

Gemini Nano then turns those facts into a short prioritized operational report.

If the local model is unavailable or needs downloading, the viewer handles that state without preventing normal log analysis.

## Privacy and security

The extension is intentionally conservative with credentials and host access.

### Browsertrix bearer token

Browsertrix log downloads can contain an `auth_bearer` token in the signed URL.

The extension:

- validates the Browsertrix log endpoint in the background service worker;
- stores the signed URL temporarily in `chrome.storage.session`;
- opens the viewer with only a random UUID in the URL hash;
- removes the signed URL from session storage as soon as the log has downloaded (it is kept until then, so a failed download can be retried by reloading the viewer tab; abandoned entries are pruned after one hour);
- fetches the log without browser cookies, since Browsertrix authenticates the download with the token in the URL;
- does not place the bearer URL in browser history, bookmarks or persistent extension storage.

### Host permissions

The configured Browsertrix Cloud host is a normal host permission.

`*://*/*` is declared as an **optional** host permission because a soft-404 probe may need to target any first-seed host. This declaration does not grant access to all sites automatically. Chrome requests access to the actual target origin at runtime.

### AI

The Gemini Nano feature uses Chrome's on-device model when supported. It is not implemented as a cloud AI API call.

## Installation

This project currently ships as an unpacked Chrome/Chromium extension rather than a Chrome Web Store package.

1. Download or clone the repository.
2. Open `chrome://extensions`.
3. Enable **Developer mode**.
4. Click **Load unpacked**.
5. Select the `browsertrix-log-viewer` directory containing `manifest.json`.
6. Open Browsertrix Cloud and click **Download Logs** for a crawl.

When upgrading an existing development installation:

1. Replace the extension files.
2. Open `chrome://extensions`.
3. Press **Reload** for Browsertrix Crawl Log Viewer.
4. Open a fresh Browsertrix log.

Reloading is particularly important after manifest permission changes.

## Using a local log

Open the extension directly from its toolbar action or viewer page and choose a local Browsertrix log file.

Expected primary format is Browsertrix NDJSON: one JSON object per line, typically containing fields such as:

```json
{
  "timestamp": "...",
  "logLevel": "debug",
  "context": "recorder",
  "message": "...",
  "details": {
    "url": "https://example.org/resource",
    "page": "https://example.org/page",
    "status": 200
  }
}
```

## Guide page

The extension includes an in-product **Guide** page covering:

- how interception works;
- table/filter/statistics behavior;
- MIME grouping;
- provenance confidence;
- soft-404 interpretation;
- runtime diagnostics;
- Gemini Nano;
- permissions/privacy;
- known limitations.

The extension's Chrome options page also opens this guide.

## Project structure

```text
browsertrix-log-viewer/
├── manifest.json
├── background.js
├── content.js
├── viewer.html
├── viewer.css
├── viewer.js
├── mime-groups.js
├── raw-view.js
├── raw-search-core.js
├── raw-search-worker.js
├── guide.html
├── guide.css
├── guide.js
├── README.md
├── README.txt
├── Instructions_for_agents.txt
└── examples/
    ├── README.txt
    ├── example-browsertrix-crawl.log
    └── reference-crawler-choice-bbc.json
```

### Main components

**`content.js`**  
Detects Browsertrix Download Logs actions, including Shoelace/open-shadow-DOM event paths, and hands valid log endpoints to the background worker.

**`background.js`**  
Validates endpoints, protects the signed log URL using session storage and opens the viewer.

**`viewer.js`**  
Streams/parses Browsertrix NDJSON, derives normalized rows and statistics, renders filters/table/details, performs provenance/diagnostics/soft-404 checks and coordinates Gemini Nano analysis.

**`mime-groups.js`**  
Central MIME extraction, recognized extension fallback, resource type normalization and functional MIME grouping.

**`raw-view.js`**  
Raw log tab: block-virtualized code view, JSON syntax colouring, search UI, wrap / only-matching / go-to-line, gutter markers.

**`raw-search-core.js` / `raw-search-worker.js`**  
Shared pattern compiler and incremental search; the worker runs full-log searches off the main thread and is terminated if a regex stalls.

**`guide.*`**  
Self-contained extension documentation with no external script dependency.

**`Instructions_for_agents.txt`**  
Technical handoff for another developer or AI agent. It documents design decisions, architecture, security constraints, heuristics, version history and regression requirements.

## Development and validation

At minimum, run these checks before packaging a release:

```bash
node --check background.js
node --check content.js
node --check viewer.js
node --check mime-groups.js
node --check raw-view.js
node --check raw-search-core.js
node --check raw-search-worker.js
node --check guide.js
python -m json.tool manifest.json > /dev/null
```

Also test with a real Browsertrix NDJSON log and verify:

- Pages appear in the file-type filter.
- `.dk`, `.com`, `.org`, etc. are never file types.
- MIME groups populate and filter statistics correctly.
- MIME group and Domain columns sort correctly.
- ordinary MP3/MP4 are not automatically classified as streaming.
- HLS/DASH/segment resources are recognized conservatively.
- `details.page` provenance is **EXACT**.
- single-worker queue inference remains **INFERRED**.
- multi-worker parentage is not guessed.
- soft-404 host permission is requested at runtime.
- local file loading works.
- Raw log tab: line numbers match table source lines, regex search, wrap and only-matching toggles, catastrophic regex is stopped.
- guide page shows the manifest version and required byline.
- Gemini unavailable/downloading states fail gracefully.
- bearer-token URLs never leak into the viewer URL or persistent storage.

## Example/reference files

The package contains two development references:

- `examples/example-browsertrix-crawl.log` — Browsertrix NDJSON suitable for viewer regression testing.
- `examples/reference-crawler-choice-bbc.json` — a separate BBC site-analysis/reference JSON used for broader URL/media-pattern testing. It is **not** a Browsertrix crawl-log fixture.

## Known limitations

- Browsertrix does not always log the parent/referrer for queued page URLs.
- Resource sizes are based on the best reliable size information present in the log; not every event contains a size.
- Soft-404 matching is heuristic and deliberately labelled as suspected.
- MIME grouping is conservative; uncommon vendor-specific `application/*` types may remain `Application / other` until explicitly categorized.
- CPU/storage capacity cannot be reported when the log does not expose it.
- Browsertrix logs a page's HTTP status only for failed or rate-limited pages, so blocking diagnostics cannot see pages that loaded with 4xx/5xx. Per-page statuses are in `pages/pages.jsonl` in the WACZ.
- "Page Load Blocked, skipping" carries no page URL in Browsertrix logs, so such pages cannot be counted as skipped.
- Gemini Nano availability depends on the user's Chrome/Chromium environment and local built-in AI support.

## Design principles

The project intentionally follows a few rules:

- Do not expose Browsertrix bearer tokens.
- Do not turn arbitrary URL suffixes or TLDs into file types.
- Do not call every audio/video resource “streaming”.
- Do not count log events as unique resources.
- Do not call soft-404 candidates confirmed soft 404s.
- Do not present inferred provenance as exact.
- Do not claim blocking just because Browsertrix ran a blocking check.
- Do not let an LLM calculate authoritative counts that deterministic JavaScript can calculate.

## Version history

| Version | Main changes |
|---|---|
| 1.0.0 | Initial intercepted Browsertrix log viewer |
| 1.1.0 | Resource type and size support |
| 1.2.0 | Soft-404 probe, dynamic totals, first-seed metadata |
| 1.3.0 | URL provenance / origin tracing |
| 1.3.1 | Runtime host-permission fix |
| 1.4.0 | Loading progress, Pages, URL-safe type detection, Domain, diagnostics, Gemini Nano |
| 1.5.0 | Guide page, agent handoff file, MIME grouping/filter |
| 1.5.1 | GitHub-ready `README.md` included in the package |
| 1.6.0 | Raw log view: code display, plain/regex search in a Web Worker, wrap and only-matching toggles, go to line, table-to-raw links |
| 1.6.1 | Review fixes: large debug logs no longer fail to load; accurate crawl-limit, page-outcome and disk-utilization detection; TypeScript `.ts` no longer "streaming"; blocking/RSS caveats; Gemini prompt fixes; lower memory; no cookies sent; `tabs` permission removed |

## Contributing / continuing development

Before making significant changes, read `Instructions_for_agents.txt`. It contains the rationale behind decisions that may otherwise look arbitrary, particularly around security, provenance confidence, soft-404 classification, MIME/streaming grouping and AI analysis.

When adding a new heuristic, prefer this pattern:

1. extract deterministic facts;
2. label uncertainty explicitly;
3. keep the raw evidence inspectable;
4. avoid turning absence of telemetry into a negative finding;
5. add a regression test/example where practical.

---

**Browsertrix Crawl Log Viewer**  
made by Thomas Smedebøl, Royal Danish Library
