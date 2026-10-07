# Changelog

All notable changes to **Crawler Choice** are documented here.

Crawler Choice is a Chrome extension for determining whether a website should be harvested with **Heritrix** or **Browsertrix**, using structured sampling, static discovery, browser rendering, interaction testing, network observation and optional on-device AI.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [Semantic Versioning](https://semver.org/).

| Version | Highlights |
|---|---|
| [1.10.0](#1100) | URL check of Heritrix's speculative URLs · srcset on the browser side |
| [1.9.0](#190) | Unique-URL comparison in the report · Gemini Nano off by default · guide close button |
| [1.8.0](#180) | Raw side = Heritrix 3.14.1 link-extraction emulation |
| [1.7.3](#173) | Burger, dropdown and submenu testing |
| [1.7.0](#170) | Second sitemap pattern · consent banners · AJAX hosts in Heritrix seeds |
| [1.6.0](#160) | Guaranteed sample from the dominant sitemap pattern |
| [1.5.0](#150) | DevTools-protocol measurement, request guard, scoring rework |
| [1.0.0](#100) | Security redesign: Incognito-only browser tests |
| [0.1.0](#010) | First prototype |

---

## 1.10.0

### Added
- **URL check** of the URLs only Heritrix finds through its heuristics (strings in JavaScript, meta content, form values, flashvars), behind a new setting *Check Heritrix's speculative URLs* (on by default). It samples up to 60 of these URLs, seeded per site so repeated runs test the same ones.
  - Each URL gets a polite `HEAD` request, or a `GET` that stops after 4 KB when HEAD is refused. Requests never carry cookies, go through the same per-host queue as other fetches, skip robots.txt-disallowed URLs when that setting is on, and the whole check is capped at 45 s.
  - A **control probe** with a made-up URL per host learns how the site answers "not found": 404, a redirect to the front page, a 200 "soft 404", or blocking.
  - Each URL is classified as *live*, *fictive* or *unknown*. The fictive share is extrapolated to all speculative URLs with a 95 % Wilson interval and finite-population correction, and is exact when the whole pool was tested.
- **Popup:** a *Checked URLs* box listing the control probes and every tested URL with its verdict, HTTP status and redirect target. The evidence JSON has the same data as `urlCheck`.
- **srcset on the browser side:** an emulation of browsertrix-behaviors' `autofetcher`, collected after load and after all interaction. It uses the same selector (images, picture/video/audio sources, noscript images, open shadow roots), the same `srcset`/`data-srcset` splitting regex, `data-src`, and `src` when a srcset exists or inside `noscript`.

### Changed
- **Short report:** two lines under the recommendation.
  - Line 1: the browser total versus the Heritrix total, and Heritrix with the estimated fictive URLs removed.
  - Line 2: found by both, only by the browser (page links, srcset), and only by Heritrix, split into *from HTML attributes* and *speculative*, with the URL-check result.

### Tests
- 163 unit tests, plus jsdom tests for the autofetch emulation.

## 1.9.0

### Added
- **Unique-URL comparison** directly under the recommendation in the short report. It compares distinct same-site URLs found by the browser with those found by the Heritrix emulation, over all samples tested both ways.
  - Browser side: DOM links (load snapshot, open shadow roots, every interaction phase) plus every requested URL.
  - Heritrix side: every outlink of the emulation, of any hop type.
  - Evidence JSON: `urlComparison`, with up to 100 example URLs from each "only" side.
- **Guide:** a close button (×, top right; Esc also works), in `guide.js` because MV3 pages cannot run inline scripts.

### Changed
- **Gemini Nano is off by default.** Settings now carry `settingsVersion` 2. Settings saved by earlier versions are reset to Gemini-off once, because their stored value was only the old default.
- The guide's settings table and its section on unique-URL totals were updated.

## 1.8.0

### Changed
- **The raw side now equals what Heritrix 3.14.1 would discover.** The raw HTML of every sample is no longer reduced to a DOM `<a href>` scan. Its URLs come from a port of Heritrix 3.14.1's own link extraction (`lib/heritrix-extractor.js`):
  - **`ExtractorHTML`:** Heritrix's own regular expressions in source order, so the first `<base>` only affects later links. This covers link-rel rules, srcset splitting, GET-only form actions, object/applet codebase, meta refresh and speculative meta content, and the value/flashvars heuristics.
  - **Inline `ExtractorJS`:** quoted strings, `unescapeEcmaScript`, `speculativeFixup` with Heritrix's 2010 TLD list, and `isVeryLikelyUri`.
  - **`ExtractorCSS.processStyleCode`** for style attributes and elements.
  - **URL handling:** `UsableURIFactory` fixups, `data:` exclusion, and `maxOutlinks` of 6000 with duplicates counted, as in Heritrix.
  - **Records** carry `rawValue`, `absoluteUrl`, `baseUrlUsed`, `sourceType`, `element`, `attribute`, `context`, `hopType`/`hop`, `extractor` and `speculative`.
  - `heritrixAbsoluteUrls()` returns only the URLs, and a standalone `.js` mode resolves against the via URI.
  - Where the 3.14.1 code differs from common descriptions, the code is followed and commented.
- **Comparison basis** (`lib/raw-basis.js`): membership uses all same-site Heritrix outlinks (any hop), while counts use only page-like ones (L, R, and speculative non-assets), so they're comparable with rendered `<a href>` links. "Discoverable in the raw HTML" for runtime-only resources uses the same Heritrix set.
- Meta robots `nofollow` follows the *Respect robots.txt* setting.

### Added
- Data tables generated from the Heritrix and webarchive-commons 3.0.3 sources (`lib/heritrix-data.js`).
- A report line describing the comparison basis on every run.
- Evidence JSON per sample: a `raw.heritrix` summary and up to 400 `raw.heritrixOutlinks` records, plus new numeric features for Gemini Nano.
- Guide: a new section, *Comparison basis*, covering the raw side, the rendered side, how they are compared, and the limits.

### Tests
- 146 tests, including Heritrix 3.14.1's own `ExtractorHTMLTest`, `ExtractorJSTest`, `ExtractorCSSTest` and `UriUtilsTest.testIsDataUri` cases ported as an oracle, with a mutation check, and the scoring effect.

## 1.7.3

### Added
- **Menu phase:** up to two menu triggers per sample are opened, and up to two submenu controls revealed by each are clicked. Each is a measured phase, and the menu is then closed again (Escape, then the toggle).
  - **Triggers:** burger/off-canvas toggles first, recognised by label, ARIA, id/class wording in several languages, `aria-controls` pointing to a nav or drawer, icon-only header toggles, or CSS checkbox burgers. Navigation dropdowns come after.
  - **Never clicked:** links that navigate, risky wording, and search, cart, account or language toggles.
  - The phase has a 10 s budget.
- New evidence: `MENU_LOADS_CONTENT` (strong) and `MENU_RENDERS_LINKS` (weak).
- The Browsertrix YAML suggests a custom menu behavior with the trigger selector. Evidence JSON: `rendered.menus`.

### Changed
- Menu phases count new content inside fixed/sticky overlays, where off-canvas drawers and dropdowns live; consent and ad containers stay excluded. Menus are no longer part of the generic click test.
- Data-loading menus block `NO_INTERACTION_DYNAMICS`.

### Fixed
- Content in off-canvas menus was always discarded, and menu toggles inside sticky headers were never tested.

### Tests
- 98 tests.

## 1.7.2

### Changed
- Guide: the author line now sits between the version label and the title.

## 1.7.1

### Changed
- **Heritrix seeds:** runtime data (AJAX) hosts are now written as plain seed URLs (`https://api.x.dk/`, or `http://` only if the host was never seen over https) instead of `+http://(…` SURT directives.
  - Each seed has its own comment line above it, with no trailing comments on seed lines.
  - Third-party hosts stay commented out (`#https://…`).
  - `ajaxHosts` entries in the evidence JSON gain an `https` flag.

## 1.7.0

### Added
- **Second sitemap pattern:** whole-site runs also include a page from the second most common sitemap URL pattern (role `secondary-pattern`), placed like the first. A sample covering the first pattern is never displaced by the second.
- **Consent banners** (new `probe/consent.js`, injected into every frame, so CMP iframes such as Sourcepoint are handled).
  - **Method:** modelled on *I don't care about cookies* (rule-based clicking, hiding and scroll unlocking, retried while the page loads) and on Consent-O-Matic / DuckDuckGo autoconsent (present/showing detectors per CMP).
  - **CMP rules:** about 28, weighted towards Danish and European sites, including Cookiebot, Cookie Information, OneTrust, Usercentrics, Didomi, Sourcepoint, Quantcast, consentmanager, Google FC, CookieYes, Complianz and Borlabs.
  - **Wording:** exact-match button wording in da/nb/sv/is/en/de/nl/fr/fi/es/it/pl, open shadow roots, and detect-and-click rounds that wait longer when a CMP script is present.
  - **Accept mode:** *Try accept all* then hides a remaining banner and backdrop and unlocks scrolling.
  - **Safeguards:** only overlay-like banners with a recognised button are touched, and links that navigate are never clicked.
  - The default stays *Leave untouched*. Each sample's outcome is recorded, and the report has a summary line.
- **Heritrix seeds** include the runtime data (XHR/fetch/EventSource) hosts seen in the browser tests. Same-site hosts are active, third-party hosts are commented out, and hosts already implied by the "/" seed are noted. They are also listed as a comment in the Browsertrix YAML and as `ajaxHosts` in the evidence JSON.

### Changed
- `www.` and the apex host count as one host when grouping sitemap patterns.
- The export field `dominantPattern` is replaced by `sitemapPatterns` (an array, by rank), and samples carry `sitemapPattern`.
- Cookie Information and Sourcepoint endpoints were added to the noise filter.
- Guide: author line, settings table, consent and pattern sections, and updated protocol, security and export text.

### Security
- **Tighter request guard during consent handling** (`lib/consent-guard.js`). Instead of allowing every non-GET request while a banner is handled, only CMP endpoints and recognisably consent-related same-site requests pass, and each one is logged as a security event.

### Tests
- 89 tests (new: consent guard, consent wording, AJAX hosts/seeds, top-2 patterns).

## 1.6.0

### Added
- Whole-site runs always include one page from the **dominant sitemap URL pattern** (new pure module `lib/urlPattern.js`).
  - **Patterns:** the page name is dropped, identifier segments are truncated (`/produkt/48213/` → `/produkt/`), dates become placeholders (`/{yyyy}/{mm}/`), and locale prefixes are kept.
  - **Counting:** over the full merged sitemap set, after the usual sample filters.
- If a selected sample already matches the pattern, nothing changes. Otherwise the pick replaces the lowest-priority slot, never the front or current page. The pick is seeded per site and taken from the newest half when `lastmod` exists.
- New sample role `dominant-pattern`, rendered right after the front/current page.
- The report and evidence export show the pattern, its share and how it was covered. Pattern labels come from sitemap URLs and are never sent to Gemini Nano.

### Tests
- New `tests/urlPattern.test.mjs` and `tests/samples-dominant.test.mjs`.

## 1.5.1

### Added
- **Test scope** selector: *Whole site* (unchanged behaviour) or *Active tab URL only*.
  - Single-page tests skip sitemap discovery and sampling, render only the active tab URL, and disable the sample count, robots.txt and early-exit options.
  - Settings are normalised in the new pure `lib/settings.js`.

### Changed
- In single-page mode, the "multiple strong signals" rule needs two strong codes on that one sample instead of two samples. Decisive signals and the Nano rules are unchanged.
- Results, report, popup headline and history are labelled as page-only. The Browsertrix YAML seeds the analysed URL with `scopeType: page`, and the Heritrix seeds contain only that URL.

### Tests
- 4 new tests (42 in total).

## 1.5.0

### Added
- **DevTools-protocol network measurement** (`lib/cdp.js`).
  - A complete request log for every frame and worker (auto-attached OOPIFs and workers), replacing the 250-entry Resource Timing buffer.
  - A request guard via `Fetch.requestPaused` that fails every non-GET/HEAD/OPTIONS request (read-only GraphQL excepted).
  - Focus emulation, active lifecycle state and a fixed viewport, so timers, rAF and IntersectionObserver keep running.
- **Pre-request containment.**
  - Tab-scoped `declarativeNetRequest` session rules block off-scope top-level navigations before they are sent.
  - Attachment and non-HTML documents are failed at the response stage.
  - All Incognito downloads during a session are cancelled and erased, and popups and windows are closed.
  - A bundled **Public Suffix List** (ICANN and private) is used for scope.
- **Lifecycle and state** (`lib/state.js`).
  - In-memory state with serialised, throttled flushes to `storage.session`, and Port-based popup updates instead of polling.
  - Startup recovery: interrupted runs are marked, and orphan windows, rules and debugger sessions are cleaned up.
  - A cancel button, a global deadline, and per-sample and per-probe timeouts; a concurrent run is rejected immediately.
  - A polite fetcher with 429/503 back-off.
- **Export and sampling.**
  - Exports: evidence JSON, a Browsertrix YAML config hint and Heritrix seeds.
  - Listing, tag, category and archive pages are sampled from the rendered navigation and from sitemaps.
  - Opt-in consent dismissal, scroll-lock detection, and inner scroll-container support.
- **Gemini Nano:** download from the popup (with user activation) when the model is `downloadable`, and availability is shown.

### Changed
- **Scoring rework** (`lib/scoring.js`).
  - A 3 s idle baseline per sample is subtracted from every phase.
  - Only net-new nodes count: still connected, and outside overlays, dialogs, consent and ad containers.
  - New links must be absent from the raw HTML and from the pre-phase DOM.
  - Per-sample evidence with prevalence weighting, and symmetric Heritrix evidence.
  - Nano can veto the non-decisive path, and there are fixture-based regression tests.
- **Gemini Nano.**
  - A hard timeout with `AbortSignal`, and the input quota is measured so the payload is trimmed to fit. The policy is sent once.
  - Runs in the service worker when the Prompt API is exposed there, otherwise in the offscreen document, which is closed after each run.
- **Code quality.**
  - The 3,000-line `background.js` was split into tested modules, and probe helpers are defined once in `probe/injected.js`.
  - Magic numbers were collected in `CONFIG` and the scoring thresholds `T`, and dead code was removed.
  - The page-language heuristic was extended beyond Danish and English (nb, sv, de, nl, fr).

### Fixed
- **Media segments** are classified by MIME/resource type. `/_next/static/chunks/*.js`, `chunk-vendors.js` and `.ts` source files no longer produce `CONTINUOUS_STREAMING_MEDIA`.
- **Challenge detection** uses vendor-specific markers, or challenge wording on small pages only.
  - Akamai CDN URLs and reCAPTCHA scripts no longer produce `BROWSER_BYPASSES_CHALLENGE`.
  - Status-only blocks are ignored when our own requests were throttled.
- **XML parsing.**
  - Entity decoding is single-pass (`&amp;lt;` → `&lt;`), with numeric entities and CDATA support.
  - Namespaced `<ns:loc>` elements are read, and `image:`/`video:`/`news:` locs are ignored.
- **Self-inflicted 429/503 errors:** bounded concurrency for raw and sitemap fetches, and robots.txt is fetched before blind probes.
- **`SCROLL_FETCHES_ELEMENTS` false positives** from consent banners, carousels and analytics are fixed by baseline correction, overlay filtering and analytics filtering.
- **Throttling of minimized/inactive tabs:** test windows are now unfocused but visible by default (one per worker, active tab), with focus emulation.
- **Response handling.**
  - `res.ok` is checked: non-2xx samples are only used as challenge evidence.
  - Truncated raw HTML disables raw-vs-rendered comparisons.
  - Decoding is charset-aware.
- **Scoring.**
  - `NO_INTERACTION_DYNAMICS` no longer requires zero XHR/fetch.
  - Evidence is counted per sample instead of once per code, and the Heritrix maximum is no longer structurally capped.
- **Sampling:** listing pages (`tag`, `page`, `media` …) are no longer excluded.
- **Popup.**
  - The HTTP check is kept on re-render, and an unhandled `start()` rejection is fixed.
  - Text areas are only rewritten when their content changes.

### Security
- No MAIN-world injection anywhere; DOM probes run in the isolated world.
- Network guard and navigation blocking no longer depend on page JavaScript or post-hoc `tabs.onUpdated`.
- Gemini Nano receives only numbers, booleans and whitelisted enums (no page text, labels or URLs).
- Redundant `tabs` and `activeTab` permissions removed; host permissions narrowed to `http(s)://*/*`.
- Messages are accepted only from the extension's own pages.

### Tests
- 38 `node:test` tests, including fixture replays of the v1.4 false positives.

## 1.4.0

### Changed
- Added locale-aware sitemap selection instead of language-only selection.
- Sitemap priority now prefers the exact detected locale, e.g. `da-DK`. It falls back to neutral/default sitemaps, then a matching English locale such as `en-DK`, and finally other English variants when necessary.
- Added sitemap content-family classification, such as homepage, category listing, product page, promotion, store information, news and sustainability.
- Large sitemap indexes now prioritise content-family diversity instead of simply following sitemap-index order.
- Preserved bounded sitemap fetching and time limits.

### Verified
- Regression-tested all decisive Browsertrix indicators and early-exit behaviour.
- Existing security, streaming, SPA, page-flipper, scrolling, interaction, polling, sampling and Incognito functionality was retained.

## 1.3.0

### Added
- Automatic detection of the website language.
- Language detection uses page metadata first and conservative text heuristics as a fallback.
- Sitemap language selection automatically uses the detected language, with English as a fallback.
- The analysis report states the auto-detected language and the sitemap selection priority.

### Changed
- Unmarked/default sitemaps remain eligible as language-neutral sources.

## 1.2.0

### Added
- Improved sampling across sites exposing multiple sitemaps.
- Up to the first 10 suitable page URLs from each inspected sitemap are added to the sampling pool.
- Round-robin sitemap-source diversity when selecting final samples.

### Changed
- Large sitemap indexes are handled with bounded expansion rather than fetching every child sitemap.
- The child-sitemap fetch allowance scales with the configured sample count.
- Concurrent sitemap fetching, shorter child-sitemap timeouts, and an overall time budget for sitemap expansion.
- Filtering and prioritisation of language-specific sitemap variants.

## 1.1.0

### Fixed
- Valid HTML pages are no longer marked as failed simply because an automated interaction triggers a security safeguard.
- Unsafe actions are now contained to the individual sample instead of aborting the complete analysis.

### Changed
- Added a *browser-test-limited* status for pages where active testing could not safely complete.
- Popup, navigation and download safeguards now preserve already collected passive/raw evidence.
- Other samples continue after a sample-specific security limitation.

## 1.0.0

### Added
- **Major security redesign.**
- Browser tests now run exclusively in a fresh Incognito session.
- HTML safety preflight before opening sample URLs in Chrome.
- Protection against document, archive, executable, media and other non-HTML sample URLs.
- Content-Disposition, MIME-type, redirect and file-extension checks.
- Emergency download detection and cancellation.
- Protection against popups, form submission, unsafe navigation, beacons and mutating requests during controlled interaction tests.
- Configurable sample count from 5 to 20.

### Changed
- Testing fails closed rather than falling back to the normal authenticated browser session.

## 0.9.0

### Added
- Early exit when decisive Browsertrix evidence is detected. Remaining expensive browser tests and the Gemini Nano analysis are skipped once Browsertrix is certain.
- An in-extension English guide explaining the decision model and criteria.
- Explicit listing of all selected sample URLs and their test status.
- Comparison between statically discoverable resources and browser-only runtime resources.
- Detection of client-rendered application shells.
- Open Shadow DOM inspection.
- Filtered content-polling detection.

### Changed
- Polling, scrolling and DOM-mutation analysis filter common advertising, analytics, telemetry and tracking noise.
- Network activity receives strong weight only when associated with archival content or meaningful page changes.

## 0.8.0

### Added
- Dedicated detection of interactive document viewers and page-flippers.
- Detects non-href next/forward controls that load new page images, tiles, document resources, canvas content or viewer state.
- Page-flipper detection works inside accessible iframes.
- Detection of viewer changes that do not use XHR/fetch.
- An interaction-risk sample for catalogue, publication, viewer, live, video and similar content types.

### Changed
- Page-flipper interaction with newly loaded page resources is treated as decisive Browsertrix evidence.

## 0.7.0

### Changed
- Disposable browser samples moved from the user's normal tab bar to a dedicated minimized, unfocused Chrome window.
- Multiple samples can still be tested concurrently.
- The test window is automatically destroyed after analysis or failure.
- Analysis no longer silently falls back to visible tabs if isolated testing cannot be established.

## 0.6.0

### Added
- Streaming-media detection: HLS/DASH manifests, segmented media, and media resources discovered only through browser execution.
- Tests whether video/audio segments begin loading only after playback starts.
- Warning: *"Streaming media detected; crawler capture may be incomplete."*

### Changed
- JavaScript-discovered streaming manifests and playback-triggered media segments are strong/decisive Browsertrix indicators.
- The entire extension UI and reporting were translated to English.

## 0.5.0

### Added
- Controlled interaction testing for Load More controls, JavaScript pagination, tabs, accordions, and menus and dropdowns.
- SPA / client-side routing detection, with a red SPA warning when Browsertrix is recommended.
- Filtering of analytics/tracking requests from interaction evidence.

### Changed
- Ordinary `<a href>` navigation explicitly does not count as high-fidelity evidence.
- Strong interaction evidence requires a non-standard browser action followed by meaningful dynamic loading.
- Current-page testing moved to a disposable test tab to avoid modifying the user's open page.

## 0.4.0

### Added
- Repeated scroll → network request → new meaningful content became decisive Browsertrix evidence.

### Changed
- Replaced limited scroll probing with progressive full-page scrolling: all selected browser samples are scrolled until the page bottom is stable or an infinite-scroll safety limit is reached.
- Limits for scroll duration, number of scroll steps, and repeated bottom-loading batches.
- Cumulative observation of dynamically added elements and resources.
- Strong scroll evidence can no longer be outweighed by unrelated Heritrix-positive scoring from other samples.

## 0.3.0

### Added
- `MutationObserver`-based measurement of elements appearing during scrolling.
- Stronger detection of DOM growth, new links, text and XHR/fetch caused by scrolling.

### Changed
- Dynamically loaded content during scrolling is weighted much more strongly toward Browsertrix.
- Scroll-triggered network loading combined with meaningful new content can independently establish the need for high-fidelity crawling.

## 0.2.0

### Added
- A field listing all discovered sitemap URLs.
- Expanded sitemap discovery using robots.txt, common sitemap paths, sitemap indexes, redirects, arbitrary/query-string sitemap URLs, `.xml.gz`, and recursive sitemap indexes.
- The current browser tab is always included as a sample.

### Changed
- A Browsertrix recommendation now requires high confidence that high-fidelity crawling is necessary.
- Gemini Nano acts only as supporting evidence and cannot independently select Browsertrix.
- Improved deterministic structural sitemap sampling.

## 0.1.0

### Added
- Initial Crawler Choice prototype.
- A clear binary recommendation: **HERITRIX** or **BROWSERTRIX**.
- A conservative decision model: default to Heritrix unless there is convincing evidence for browser-based crawling.
- Structured site sampling based on sitemaps and site architecture instead of random URLs.
- Raw HTML versus browser-rendered DOM comparison.
- Basic detection of JavaScript rendering, XHR/fetch, lazy loading, scrolling, iframes and framework signals.
- Parallel fetching for faster analysis.
- A short recommendation report and live progress/status display.
- An orange extension icon while analysis runs, and a green icon when complete.
- Gemini Nano / Chrome Prompt API as a local secondary interpretation layer.
