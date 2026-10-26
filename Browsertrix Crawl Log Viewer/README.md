<div align="center">

# Browsertrix Crawl Log Viewer

**Read a Browsertrix crawl log as answers, not as thousands of JSON lines.**

[![Chrome desktop](https://img.shields.io/badge/Chrome-desktop-4285F4?logo=googlechrome&logoColor=white)](#-requirements)
[![Manifest V3](https://img.shields.io/badge/Manifest-V3-34A853)](#)
[![Gemini Nano](https://img.shields.io/badge/Gemini%20Nano-optional%2C%20on--device-8E75B2?logo=googlegemini&logoColor=white)](#gemini-nano-optional)
[![Any Browsertrix](https://img.shields.io/badge/Browsertrix-any%20installation-orange)](#connect-your-browsertrix)
[![No dependencies](https://img.shields.io/badge/runtime%20dependencies-none-brightgreen)](#-development)

by **Thomas Smedebøl**, Royal Danish Library

[Requirements](#-requirements) · [Install](#-install-in-developer-mode) · [Usage](#-usage) · [How it works](#-how-it-works) · [Safety](#-safety-and-privacy) · [Development](#-development)

</div>

---

## ✨ What it does

A Browsertrix crawl log is newline-delimited JSON: one event per line, often hundreds of thousands of them. The answers a curator or crawl engineer needs are spread across those lines. **Browsertrix Crawl Log Viewer** catches **Download Logs** in Browsertrix and opens the log in a viewer built for crawl QA:

| | |
|---|---|
| 📊 **Table** | every event, sortable and filterable, with status, type, MIME group, size, domain, page and where each URL came from; columns can be dragged into any order |
| 📜 **Raw log** | the original log as code, with line numbers, syntax colouring and plain-text or regular-expression search |
| 🩺 **Diagnostics** | crawl outcome, limits, blocking and throttling signals, memory, storage problems, timeouts and soft-404 suspects |
| 🤖 **Gemini Nano** | an optional short report from Chrome's on-device model, built from the diagnostics, not from the raw log |

> [!NOTE]
> The table and the raw log stay the source of truth. Every derived finding says whether it is a logged fact, a heuristic or an inference, and inferred relationships are never shown as exact.

---

## 📋 Requirements

> [!IMPORTANT]
> Browsertrix Crawl Log Viewer runs in **desktop Google Chrome** and works with **any Browsertrix**: Webrecorder's hosted service or your own installation. After installing, you connect it to your Browsertrix once (step 5 below). No Browsertrix address is built in.

| Browser | Supported |
|---|:---:|
| Google Chrome on Windows, macOS or Linux | ✅ |
| Chrome on Android or iOS | ❌ *no extension support* |
| Other Chromium browsers (Edge, Brave, Opera, Vivaldi …) | ❌ *not supported, not tested* |

### Gemini Nano (optional)

The viewer can ask Chrome's on-device model for a **short report**. It runs only when you press the button, and every other feature works without it. It needs **Chrome 138 or later**, where the Prompt API is available to extensions, and the computer must meet [Chrome's requirements for built-in AI](https://developer.chrome.com/docs/ai/get-started):

| | Requirement |
|---|---|
| 💻 Operating system | Windows 10/11, macOS 13 (Ventura)+, Linux, or ChromeOS on Chromebook Plus |
| 💾 Storage | at least **22 GB free** on the volume holding the Chrome profile |
| ⚙️ Hardware | GPU with **more than 4 GB VRAM**, *or* **16 GB RAM** and **4+ CPU cores** |
| 📶 Network | unmetered connection for the one-time model download |

After the download the model runs locally, and no data leaves the computer.

> [!TIP]
> If Gemini Nano can't run, the button reads **Why is Gemini Nano unavailable?** Clicking it explains the reason for *your* browser and what to do: use Google Chrome instead of another browser, update an old Chrome, turn built-in AI back on (or ask your IT department if a policy switched it off), or check the computer requirements above.

---

## 🚀 Install in developer mode

The extension is not in the Chrome Web Store; it is loaded as an **unpacked** extension.

1. [Download](https://github.com/thsm-kb/Browserextensions/raw/refs/heads/main/Browsertrix%20Crawl%20Log%20Viewer/Browsertrix%20Crawl%20Log%20Viewer%20v1.16.0.zip) and unzip the extension to a folder of your choice.
2. **Open** `chrome://extensions` in the address bar.
3. **Turn on Developer mode** with the switch in the top-right corner.
4. Click **Load unpacked** and select the folder that contains `manifest.json` (the folder is called **Browsertrix Crawl Log Viewer**).
5. **Connect your Browsertrix (required).** The **Settings** page opens by itself; see [Connect your Browsertrix](#connect-your-browsertrix) below.
6. **Pin it.** Click the 🧩 puzzle-piece icon in the toolbar and pin **Browsertrix Crawl Log Viewer**.
7. *(Optional)* In the viewer, press **Analyze with Gemini Nano**. If the model isn't downloaded yet, the button reads **Download model & analyze** and the viewer shows the download progress.

> [!WARNING]
> Step 5 is required. Without a connected address the extension doesn't react to **Download Logs** anywhere (you can still open log files with **Open local log**).

### Connect your Browsertrix

1. **Copy the address.** Open your Browsertrix in a tab and copy the address from the address bar. Any page inside Browsertrix will do; only the first part, such as `https://browsertrix.example.org`, is used.
2. **Paste it** into the field on the **Settings** page and click **Add**.
3. **Click Allow.** Chrome asks whether the extension may *"Read and change your data on"* that site. This lets it notice the **Download Logs** button and download the log for you. It gets access to that one site only.
4. **Check the result.** Settings tests the address and shows **Browsertrix confirmed**. If it says the address *did not answer like Browsertrix*, check for a typo.
5. **Try it.** Open a crawl in Browsertrix and click **Download Logs**. The viewer opens. Browsertrix tabs that were already open work without reloading.

> [!TIP]
> Open **Settings** again at any time with the **Settings** button in the viewer, or via `chrome://extensions` → **Details** → **Extension options**. You can add several addresses (for example a test and a production Browsertrix) and remove them again. The addresses are stored only in your Chrome profile.

<details>
<summary><b>Updating to a new version</b></summary>

Replace the folder's contents with the new version and click the **↻ reload** arrow on the extension's card in `chrome://extensions`. Your Browsertrix addresses, column order and raw-log search options are kept.

</details>

<details>
<summary><b>Updating from version 1.8 or older</b></summary>

From version 1.9 the extension has no built-in Browsertrix address. After updating, the **Settings** page opens: add the address of your Browsertrix once, as described in [Connect your Browsertrix](#connect-your-browsertrix).

</details>

<details>
<summary><b>Updating from version 1.7 or older</b></summary>

Older versions were unzipped to a folder called `browsertrix-log-viewer`. Chrome identifies an unpacked extension by its folder, so loading the new **Browsertrix Crawl Log Viewer** folder *next to* the old one installs a second copy, and both would open a viewer when you click **Download Logs**. Do one of the following:

- **Remove** the old card in `chrome://extensions`, then load the new folder (your column order is reset once), or
- **copy** the contents of the new folder into your existing `browsertrix-log-viewer` folder and click **↻ reload** (everything is kept).

</details>

<details>
<summary><b>Managed (company / institution) computers</b></summary>

If your organisation's Chrome policies block developer mode or unpacked extensions, the extension can't be installed. Ask your IT department.

</details>

---

## 🧭 Usage

1. Open a finished crawl in your Browsertrix and click **Download Logs**.
2. Instead of a file download, the viewer opens in a new tab and shows the download progress.
3. Read the diagnostics at the top, then use the **Table** and **Raw log** tabs to dig in.

To open a log you already have, click the **Browsertrix Crawl Log Viewer** icon and choose **Open local log**. Browsertrix NDJSON logs (`.log`, `.ndjson`, `.txt`, `.json`) are accepted.

> [!TIP]
> The **Guide** button in the viewer explains every column, filter and finding, with the caveats that matter when interpreting them.

### Normal and large logs

The viewer picks a mode by size and shows it next to the file name:

| | Normal mode | **Large-log mode** |
|---|---|---|
| Used for | logs up to 500 MB | logs above 500 MB |
| Raw text | in memory | **stays on disk**: a local file is read directly; a log downloaded from Browsertrix is written, while it downloads, to the extension’s private disk storage (only this extension can read it) and deleted when the tab closes |
| Rows | full detail in memory | **compact**: shared text, numbers, and only the few fields the analyses need; a line’s full details are read from disk when you open it |
| Memory | about 3–4 × the log size | about **180 bytes per log line** (a 1.7 GB log: roughly 1.2 GB instead of 5–7 GB) |

Everything works in both modes: table, filters, sorting, totals, diagnostics, status overview, provenance, the raw log and regex search. In large-log mode, searching streams the file from disk and opening details reads that line from disk, so they take a moment longer. In testing, both modes gave identical results.

### While a crawl log loads

Loading a large log takes a while, so the viewer shows **friendly cards in the middle of the screen while the crawl log loads**: more than 120 short tips for crawl engineers and curators (robots.txt, sitemaps, HTTP, scope, Heritrix, Browsertrix, behaviors, CSS selectors, WARC/WACZ, QA and web technology) plus good-to-know facts about this viewer.

- Cards appear only when loading takes more than about 1.5 seconds. Click **Next** (or press →) for another card, **×** (or Esc) to close them for this load.
- When the log is ready the card closes by itself, unless you are reading it; then it offers **Show my log**.
- **Optional news** (off by default): turn on **Include web-archiving news** in Settings to mix in fresh items from web-archiving sources: new releases of Heritrix, Browsertrix, Browsertrix Crawler, NetarchiveSuite, pywb, ReplayWeb.page and SolrWayback; the Webrecorder, IIPC, Internet Archive and Library of Congress blogs; and ArchiveTeam Deathwatch shutdown warnings. Only items **published within the last 30 days** are shown, at most 10 per source, each with a link to the original. Gemini Nano writes the short card texts on your computer when it is available.
- Turn the cards off in **Settings → While a crawl log loads**, or with *Don’t show cards while loading* on any card.

> [!NOTE]
> News feeds are fetched at most once a day and never while a log is loading. Turning news on asks Chrome for access to those news sites; turning it off deletes the cached news and gives the access back. Fetching a feed tells that site your IP address, as visiting it would; no crawl log data is ever sent.

### What you get

**The table.** One row per log event. Sort by any column; drag a column header left or right to move it (Alt+←/→ with the keyboard). Columns can only be moved, never hidden. Filter by text, level, context, file type and MIME group, and the totals follow the filter: unique pages, unique resources, combined, average, median and largest size. The **fields** button at the end of each message shows the event's full `details` JSON and a link to the line in the raw log.

**HTTP status.** A status dropdown filters by status group (1xx–5xx), by *Suspected blocking / throttling / rate limiting*, by *No status logged*, or by a single code such as *403 · Forbidden*. The **HTTP status overview** in the diagnostics shows, for the whole log, how many different URLs got each kind of status: bars show the tendency (hover for the exact count; small groups always stay visible), and clicking a bar filters the table. *Suspected blocking* uses exactly the same rules as the blocking diagnostics: 401, 403, 407, 429 or 503, and challenge or rate-limit messages.

**Resource totals.** Under a MIME group or file type, the totals line shows unique resources and how many log lines they come from. A file linked directly from a page (a video, PDF or image) is fetched by Browsertrix as a "page"; the viewer counts it both as a page and as a resource of its own type, so it is no longer missing from, for example, *Video files*. Browsertrix logs no size for such directly fetched files.

**The raw log.** The log exactly as Browsertrix wrote it, as code. Line numbers match the table's source line. Gutter marks show warnings, errors and lines that aren't valid JSON.

| Raw-log control | What it does |
|---|---|
| 🔎 Search | plain text by default; **.\*** for a regular expression, **Aa** to match case; Enter and Shift+Enter move between matches |
| ≡ Only matching lines | shows only lines with a match, like `grep` |
| ↩ Wrap lines | wraps long lines instead of scrolling sideways |
| # Go to line | jumps to a line number |

**The diagnostics**, calculated by the viewer itself:

| Finding | Based on |
|---|---|
| 🏁 Crawl outcome | pages completed, failed, skipped or not confirmed complete |
| 🛑 Limits | Browsertrix's own stop messages: size, time, failed-page and disk-utilization thresholds, page limit |
| 🚧 Blocking / throttling | 401/403/407/429/503 statuses, WAF and challenge messages, rate-limit messages |
| 🧠 Memory | RSS and heap of the crawler process, when the crawl was logged at debug level |
| 💾 Storage | write, quota and WARC-upload errors and disk-utilization stops |
| 👻 Soft 404 | whether the first seed's host answers 200 for a page that doesn't exist, and which resources look like that answer |
| 🧭 Provenance | where each page or resource URL was discovered, labelled EXACT, INFERRED, INITIAL or UNKNOWN |

---

## 🔬 How it works

```mermaid
flowchart LR
    A[Download Logs<br/>in Browsertrix] --> B[Signed URL kept in<br/>session memory]
    B --> C[Streamed download<br/>with progress]
    C --> D[Parse NDJSON<br/>one event per line]
    D --> E[Table · Raw log]
    D --> F[Deterministic<br/>diagnostics]
    F --> G[Optional<br/>Gemini Nano report]
```

1. **Interception.** On each Browsertrix address you connected in Settings, a content script catches the **Download Logs** click. The service worker checks that the link comes from that same connected address and is a real Browsertrix log endpoint, then keeps the signed link in Chrome's session memory under a random key; only that key goes into the viewer's address.
2. **Download.** The viewer streams the log and shows percentage and bytes, or received bytes when the size is unknown. Lines are parsed while they arrive, in short slices with visible progress, so the page never freezes, even for logs of several hundred MB.
3. **Parsing.** Each line is one event. Status, size, domain, page, file type and MIME group are taken from what Browsertrix logged; a file type is inferred from a URL only for recognized extensions, never from a hostname such as `.dk`.
4. **Diagnostics.** Findings are matched against Browsertrix Crawler's actual messages and stay conservative: a Cloudflare *check* is not a block, and a page giving up after retries is not a crawl limit.
5. **Gemini Nano.** The model receives only the compact fact summary from step 4, never the raw log, and writes a short prioritised report. *JavaScript calculates; Gemini interprets.*

<details>
<summary><b>MIME groups</b></summary>

MIME names and top-level types follow the [IANA Media Types registry](https://www.iana.org/assignments/media-types/media-types.xhtml). IANA does not define QA groupings such as *Office / documents*, so the viewer adds a documented functional layer on top.

| Group | Examples |
|---|---|
| Web pages | HTML/XHTML |
| Images | JPEG, PNG, GIF, WebP, SVG, AVIF … |
| Office / documents | PDF, Word, Excel, PowerPoint, OpenDocument, RTF … |
| Audio files / Video files | ordinary MP3, OGG, WAV, FLAC / MP4, WebM, MOV |
| Streaming manifests / playlists | HLS, DASH |
| Streaming audio / video / media segments | media with a streaming-specific signal, TS/M4S segments |
| Archives / compressed | ZIP, TAR, GZIP, 7z, RAR … |
| Structured data | JSON, XML … |
| Scripts / web code · Stylesheets · Fonts | JavaScript, TypeScript · CSS · WOFF/WOFF2/TTF/OTF |
| E-books · 3D / models · Messages / multipart | EPUB · model/\* · message/\*, multipart/\* |
| Text · Application / other · Binary / unknown | other text/\* · unassigned application/\* · generic binary |
| Crawler-made | records the crawler creates itself: screenshots, extracted page text, page info |

Streaming is conservative: an ordinary MP3 or MP4 is an audio or video file, not streaming, unless a streaming manifest, segment MIME type or segment URL says otherwise.

**Crawler-made** items are records Browsertrix Crawler creates itself and stores in the WARC: screenshots (view, thumbnail, full page), extracted page text and page info. They are labelled with their own type, such as *Crawler-made · screenshot (view)*, recognised only by the crawler's own log signals, and never counted as pages, images, resources or soft-404 suspects. The log records no size for them.

</details>

<details>
<summary><b>Soft-404 test</b></summary>

The viewer requests `<first-seed origin>/browsertrixsoft404detect`, a path that should not exist. If it answers **200**, the host may serve soft 404s. Resources on the same host with the same type and about the same size (±8 % or ±2 KB, whichever is larger) are marked **SOFT404?**. They are suspects for review, not confirmed missing pages. Chrome asks for permission to that one host before the test runs.

</details>

---

## 🔒 Safety and privacy

- **The download token never leaves the extension.** Browsertrix puts an `auth_bearer` login token in the log link. The extension passes the link from the Browsertrix page to the viewer through Chrome's in-memory session storage, which only the extension's own pages can read. The viewer deletes it there the instant it picks it up, and an entry nobody picks up is deleted after 2 minutes. After that the link exists only in the viewer tab's memory until the log has downloaded. It is never written to disk, the viewer's address, history, bookmarks, the page, the console, exports or the Gemini prompt.
- **Large logs: content on disk, never the token.** A log downloaded from Browsertrix is copied, while it downloads, into the extension’s private disk storage, which no website or other extension can read, so the viewer can switch to large-log mode if it passes 500 MB. The copy holds log content only (never the signed link or its token). For a normal-size log it is deleted as soon as the log has loaded; in large-log mode when the tab closes or another log is opened; any left-over copy is deleted after a day.
- **A careful download.** The log is fetched without cookies, without telling Browsertrix which page asked (no Referer), without Chrome's disk cache, and without following redirects, so the link can't be bounced anywhere else.
- **Nothing downloads behind your back.** If the viewer can't be opened, the extension shows a short notice and downloads nothing. It does not fall back to a normal download, which would store the link with its token in Chrome's download history.
- **Error messages never show the link.** Every message the extension shows or logs is scrubbed of tokens first.
- **No site access by default.** The extension is installed without access to any website. It gets access only to sites you approve in Chrome's prompt: each Browsertrix address you add in Settings, and the crawled site for the optional soft-404 test. Removing an address also removes that access.
- **No built-in addresses.** The extension contains no Browsertrix address. Yours are stored only in your Chrome profile and are never sent anywhere.
- **Only links from your Browsertrix.** A log link is opened only if it comes from the same connected address as the page it was clicked on, and only `https://` addresses are accepted (plain `http://` only for `localhost`).
- **News is opt-in.** Web-archiving news on the loading cards is off by default. When on, feed content is treated as untrusted text: it is never rendered as HTML, only http(s) links are kept, and Gemini Nano is told never to follow instructions inside it.
- **On-device AI only.** Gemini Nano runs locally. It receives a short fact summary, never the raw log or the link.

> [!NOTE]
> A failed download is retried with the **Retry** button in the viewer. Reloading the viewer tab can't retry, because the link is no longer stored anywhere; click **Download Logs** in Browsertrix again instead.
- **No data leaves your computer** other than the log download from Browsertrix and the optional soft-404 request to the crawled site.

---

## ⚠️ Limitations

- **Status codes depend on the log level.** At normal log level Browsertrix logs a page's status only when the page fails or is rate-limited, so a page that loaded with 403 or 404 looks like a success and *no blocking signal* is not proof that nothing was blocked. With `--logging debug`, every page's status (and most resources') is logged. Per-page statuses are also in `pages/pages.jsonl` inside the WACZ.
- **Memory needs debug logging.** RAM figures exist only if the crawl ran with `--logging debug`, and they cover the crawler process, not the browser. CPU and disk capacity are reported only when logged.
- **Not every parent is known.** Browsertrix doesn't always log where a queued URL came from. Timing-based guesses are labelled INFERRED and are not made for multi-worker crawls. *Page Load Blocked* events carry no page URL, so those pages can't be counted as skipped.
- **Huge logs need some memory and disk.** Logs above 500 MB load in large-log mode, which needs roughly 180 bytes of memory per log line (about 1.2 GB for a 1.7 GB log of 6.5 million lines) and, for logs downloaded from Browsertrix, free disk space for a temporary copy. Searching and sorting millions of rows take a few seconds.
- **Heuristics are labelled.** Soft-404 matches are suspects, and uncommon vendor MIME types may stay *Application / other*.

---

## 🛠 Development

No runtime dependencies and no build step: the folder you load **is** the source.

```bash
for f in *.js; do node --check "$f"; done
python -m json.tool manifest.json > /dev/null
```

Then test with a real Browsertrix log; the full checklist is in `Instructions_for_agents.txt`.

`example-browsertrix-crawl.log` is a real Browsertrix crawl log, **redacted** for publication: host names are reserved example domains (`example.org`, `example.net`, `media.example.org`) that never point to real sites, every URL path word is lorem ipsum (file extensions kept, the same original always gets the same replacement), and file names and browser IDs are neutral. Browsertrix's own messages, log levels, contexts, HTTP statuses, sizes, MIME types and timestamps are unchanged, so it exercises parsing, pages and resources, sizes, MIME groups, crawler-made items, memory telemetry, soft-404 comparison and provenance exactly like the original. Open it with **Open local log**.

<details>
<summary><b>Changing the news feeds or the time window</b></summary>

Edit `news-config.js`: each source has an `id`, a `name`, an RSS/Atom `url` and a `kind` (`release`, `community` or `shutdown`); `MAX_AGE_DAYS` sets the window (30 days), `MAX_PER_SOURCE` the number of items per source (10), and `CARD_TITLE_MAX` / `CARD_TEXT_MAX` the card length. After adding a source on a new website, turn news off and on once in Settings so Chrome can ask for access to it. A source that fails is shown as unavailable in Settings and never blocks the others. Tip cards live in `loading-cards-data.js`.

</details>

<details>
<summary><b>Project layout</b></summary>

```text
manifest.json                MV3 manifest
background.js                service worker: validates log links, registers content.js per address, opens the viewer
content.js                   catches Download Logs on the connected Browsertrix addresses
settings.html/.css/.js       the Settings page: connect and remove Browsertrix addresses
browsertrix-hosts.js         address rules shared by Settings and the service worker
viewer.html/.css/.js         the viewer: parsing, table, filters, diagnostics, soft-404, Gemini Nano
mime-groups.js               MIME extraction, extension fallback, type labels and MIME groups
raw-view.js                  the Raw log tab: virtualized code view and search UI
raw-search-core.js           pattern compiler and incremental search (shared)
raw-search-worker.js         Web Worker that runs full-log searches
guide.html/.css/.js          the user guide (opened from the viewer)
Instructions_for_agents.txt  design decisions and regression rules for developers and AI agents
line-store.js                holds a log as line-aligned chunks plus line offsets (never one huge string)
disk-lines.js                large-log mode: raw text on disk (line offsets only) and the private disk copy
loading-cards.js             the cards shown while a crawl log loads
loading-cards-data.js        the tip cards (edit freely: title ≤ 60, text ≤ 240 characters)
news-config.js               news sources, 30-day window and card limits (edit here to change feeds)
news.js                      fetches, filters, summarises and caches news cards
settings-loading.js          Settings: “While a crawl log loads”
example-browsertrix-crawl.log  a redacted example crawl log for testing
```

</details>

Before larger changes, read `Instructions_for_agents.txt`: it explains decisions that may otherwise look arbitrary, particularly around tokens, provenance, soft 404s, MIME grouping and AI analysis.

📜 See [CHANGELOG.md](CHANGELOG.md) for the version history.

---

## 🙏 Acknowledgements

- **[Browsertrix](https://github.com/webrecorder/browsertrix)** and **[Browsertrix Crawler](https://github.com/webrecorder/browsertrix-crawler)** (Webrecorder). The diagnostics are matched against the crawler's own log messages; no Browsertrix code is included.
- **[IANA Media Types registry](https://www.iana.org/assignments/media-types/media-types.xhtml)**, the basis for MIME names and top-level types.
