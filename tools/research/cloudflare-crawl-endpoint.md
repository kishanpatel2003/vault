---
title: "Cloudflare Browser Rendering /crawl Endpoint"
type: research_brief
date: 2026-03-11
requested_by: kishan
tags: [cloudflare, web-scraping, browser-rendering, crawling, api, rag, agents]
mission: null
sources_count: 5
summary: "Cloudflare entered open beta on March 10, 2026 with a /crawl endpoint for its Browser Rendering REST API, enabling asynchronous multi-page site crawls returning HTML, Markdown, or AI-extracted JSON from a single API call, billed as standard browser hours."
---

## background

Cloudflare's Browser Rendering product runs headless Chrome on Cloudflare's global edge network. Before this announcement it offered a suite of single-page REST endpoints: `/content` (HTML), `/markdown`, `/screenshot`, `/pdf`, `/snapshot`, `/scrape`, `/json`, and `/links`. The `/crawl` endpoint, announced March 10, 2026 (open beta), adds multi-page asynchronous crawl orchestration to that suite — moving it from a "single URL" tool to a "whole site" tool. No separate product purchase is required; it runs under the existing Browser Rendering service on both Free and Paid Workers plans.

Research scope: Cloudflare official docs (`developers.cloudflare.com/browser-rendering/`), pricing and limits pages, REST API reference, and Hacker News community thread (HN #47329557) for real-world signal on gaps.

---

## key findings

### 1. What it does — capabilities, output formats, depth/scope

- **Asynchronous two-step model.** `POST /crawl` submits a job and returns a UUID job ID immediately. `GET /crawl/{id}` polls for status and paginated results. Jobs run for up to 7 days; results persist for 14 days after completion. Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Output formats: HTML, Markdown, or JSON.** The `formats` array parameter accepts any combination of `html`, `markdown`, and `json`. Default is HTML. Markdown format uses Cloudflare's existing HTML-to-Markdown conversion. JSON format routes each page through Workers AI (via the existing `/json` endpoint logic), requiring a `jsonOptions` object with `prompt` and/or `response_format` (JSON Schema). JSON extraction incurs Workers AI usage charges on top of browser hours. Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Crawl depth and page limits.** Both `limit` (max pages) and `depth` (max link depth) default to 100,000, with a maximum ceiling of 100,000 each. The practical default starting point for a call with no parameters set is 10 pages (`limit` defaults to 10 in the docs table). Source: [Cloudflare Docs — /crawl endpoint, optional parameters table](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Scope controls.** The `source` parameter governs URL discovery: `all` (sitemaps + page links, default), `sitemaps` only, or `links` only. Boolean flags `options.includeExternalLinks` (default false) and `options.includeSubdomains` (default false) gate cross-domain crawling. Wildcard patterns via `options.includePatterns` and `options.excludePatterns` filter which URLs are visited (`excludePatterns` takes strict priority over `includePatterns`). Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Render mode.** `render: true` (default) spins up a full headless Chrome session and executes JavaScript — necessary for SPAs and dynamic content. `render: false` does a fast static HTTP fetch with no JS execution, billed under Workers (not browser hours); this mode is free during beta and will be billed under standard Workers pricing afterward. Source: [Cloudflare Docs — /crawl endpoint, render parameter](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Caching.** An R2-backed cache is controlled by `maxAge` (default 86,400 seconds = 1 day, max 604,800 seconds = 7 days). Cache hit requires exact URL + parameter match. The `modifiedSince` Unix timestamp parameter enables incremental re-crawls (only visits pages modified since a given time). Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **robots.txt compliance.** The crawler respects `robots.txt` including `Crawl-delay`. Disallowed URLs are returned in the response with `"status": "disallowed"` rather than silently dropped. Source: [Cloudflare Docs — /crawl endpoint, crawler behavior](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

- **Response structure.** Each crawled page record includes: `url`, `status`, `markdown`/`html`/`json` (per requested formats), and a `metadata` object with HTTP status code, page title, and URL. Results are paginated via `cursor` if the response exceeds 10 MB. Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

---

### 2. Pricing — free tier, Workers plan limits, rate limits

**Browser hours model (shared across all Browser Rendering endpoints):**

| Plan | Browser hours included | Overage |
|------|----------------------|---------|
| Workers Free | 10 minutes/day | None — hard cap, 429 error |
| Workers Paid | 10 hours/month | $0.09/hour |

Source: [Cloudflare Docs — Browser Rendering Pricing](https://developers.cloudflare.com/browser-rendering/pricing/)

**Cost illustration (Workers Paid, render: true):** 50 browser-hours in a month = 40 billable hours × $0.09 = **$3.60**. At a crawl that uses, say, 2.7 seconds per page (inferred from the `browserSecondsUsed: 134.7` on a 50-page crawl in the docs example), 50 pages ≈ 0.037 browser hours ≈ $0.003 per 50-page crawl at overage rates. Source: [Cloudflare Docs — Pricing examples](https://developers.cloudflare.com/browser-rendering/pricing/); [/crawl example response](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

**JSON format additional cost:** Using `formats: ["json"]` routes each page through Workers AI. Workers AI has its own usage pricing (per token/inference). The docs do not quote a combined per-page cost for JSON mode — this is a gap. Source: [Cloudflare Docs — /crawl endpoint, formats parameter](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

**render: false pricing:** Currently free during beta. Post-beta will be billed under Workers pricing (currently $0.30 per million CPU milliseconds for Workers Paid). Source: [Cloudflare Docs — /crawl endpoint, render parameter](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

**Crawl-specific limits by plan:**

| Feature | Workers Free | Workers Paid |
|---------|-------------|-------------|
| Crawl jobs per day | 5 | Not stated (limited by browser hour budget) |
| Max pages per crawl job | 100 | 100,000 (via `limit` param) |
| REST API request rate | 6/min (1 every 10 sec) | 600/min (10/sec), fixed fill rate |
| Max job runtime | 7 days | 7 days |

Source: [Cloudflare Docs — Limits](https://developers.cloudflare.com/browser-rendering/limits/)

**Notable billing behavior:**
- Failed requests with `waitForTimeout` errors are not billed. Source: [Cloudflare Docs — Pricing FAQ](https://developers.cloudflare.com/browser-rendering/pricing/)
- The `X-Browser-Ms-Used` response header reports browser time in milliseconds per request for cost monitoring. Source: [Cloudflare Docs — REST API overview](https://developers.cloudflare.com/browser-rendering/rest-api/)
- The `cancelled_due_to_limits` job status fires when account browser time is exhausted mid-crawl. Source: [Cloudflare Docs — /crawl endpoint](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

---

### 3. API usage — how to call it, auth, key params

**Authentication:**
- Requires a Cloudflare API Token with `Browser Rendering - Edit` permission.
- Token passed as `Authorization: Bearer <apiToken>` header.
- Account ID embedded in the URL path.
Source: [Cloudflare Docs — REST API, Before you begin](https://developers.cloudflare.com/browser-rendering/rest-api/)

**Endpoint:**
```
POST https://api.cloudflare.com/client/v4/accounts/{account_id}/browser-rendering/crawl
GET  https://api.cloudflare.com/client/v4/accounts/{account_id}/browser-rendering/crawl/{job_id}
DELETE https://api.cloudflare.com/client/v4/accounts/{account_id}/browser-rendering/crawl/{job_id}
```

**Minimal launch call:**
```bash
curl -X POST 'https://api.cloudflare.com/client/v4/accounts/{account_id}/browser-rendering/crawl' \
  -H 'Authorization: Bearer <apiToken>' \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://example.com"}'
# Returns: {"success": true, "result": "<job-uuid>"}
```

**Key parameters (beyond url):**

| Parameter | Type | Default | Max | Notes |
|-----------|------|---------|-----|-------|
| `limit` | number | 10 | 100,000 | Max pages to crawl |
| `depth` | number | 100,000 | 100,000 | Max link depth from start URL |
| `formats` | string[] | `["html"]` | — | `html`, `markdown`, `json` |
| `render` | boolean | true | — | false = no JS, fast, free in beta |
| `source` | string | `"all"` | — | `all`, `sitemaps`, `links` |
| `maxAge` | number (seconds) | 86400 | 604800 | R2 cache TTL |
| `modifiedSince` | Unix timestamp | — | — | Incremental crawl filter |
| `options.includeExternalLinks` | boolean | false | — | Follow external domains |
| `options.includeSubdomains` | boolean | false | — | Follow subdomains |
| `options.includePatterns` | string[] | — | — | Wildcard URL allowlist |
| `options.excludePatterns` | string[] | — | — | Wildcard URL denylist (higher priority) |
| `jsonOptions` | object | — | — | Required when formats includes `json` |
| `authenticate` | object | — | — | HTTP Basic auth (username/password) |
| `cookies` | array | — | — | Cookie injection |
| `setExtraHTTPHeaders` | object | — | — | Custom headers (e.g., API keys) |
| `gotoOptions.waitUntil` | string | — | — | Puppeteer-style page load event |
| `rejectResourceTypes` | string[] | — | — | Block images, media, fonts, etc. |

Source: [Cloudflare Docs — /crawl endpoint, optional parameters](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/); [Browser Rendering API reference](https://developers.cloudflare.com/api/resources/browser_rendering/)

**Polling pattern (from official docs):** Poll with `?limit=1` to check status cheaply. Once status is not `running`, fetch full results without `limit`. Use `cursor` param for paginating results beyond 10 MB. Source: [Cloudflare Docs — /crawl endpoint, polling](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

---

### 4. Comparison to existing tools

**vs. Python scraping libraries (BeautifulSoup, Scrapy, Scrapling):**
- `/crawl` requires zero infrastructure: no server, no browser binary install, no session management. Scrapy and Scrapling require a running Python environment and orchestration code.
- `/crawl` handles JS rendering natively (headless Chrome). BeautifulSoup and basic HTTP libraries require Playwright/Selenium wrappers for JS-heavy pages.
- `/crawl` does not bypass Cloudflare bot protection. HN commenters (HN #47329557) note the crawler is identified as a bot and is subject to the same WAF/Bot Management rules it enforces on others — a significant gap for crawling Cloudflare-protected sites. Scrapling explicitly markets "undetectable" anti-bot bypass (that claim was assessed in [vault: tools/python/scrapling.md]).
- Cost at scale: self-hosted scraping has near-zero marginal per-page cost once infrastructure is running. At $0.09/hr for rendered pages, /crawl costs scale with usage.

**vs. browser automation frameworks (Puppeteer, Playwright):**
- `/crawl` abstracts multi-page orchestration entirely — no code to enumerate links, queue URLs, handle concurrency, or manage browser lifecycles. Puppeteer/Playwright require all of this to be written by the developer.
- Cloudflare's Workers Bindings method (separate from REST API) still exposes full Puppeteer and Playwright for complex workflows where /crawl's fixed behavior is insufficient.
- Playwright offers session reuse and stateful flows; /crawl is stateless per job.

**vs. existing Cloudflare Browser Rendering single-page REST endpoints:**
- `/crawl` is additive to the existing suite. Before it, developers needed to call `/markdown` or `/content` once per URL, manually discover and queue links, and handle pagination themselves. `/crawl` moves that orchestration server-side.
- The existing `/markdown` and `/content` endpoints are synchronous (request → response). `/crawl` is asynchronous (submit → poll). Different interaction model; relevant for tooling integration.

**vs. third-party crawl/scrape APIs (Firecrawl, Apify, Jina AI /r/ endpoint):**
- Cloudflare's /crawl runs on its own global edge network — low latency from any geography, no cold-start penalty. Firecrawl and Apify run on centralized cloud infrastructure.
- Cloudflare integrates natively with Workers, Vectorize, AI Gateway, and R2 — tighter stack integration for Cloudflare-native builders.
- Firecrawl and Apify have more mature anti-bot bypass features and proxy rotation (single-sourced based on product marketing; not independently verified here).
- Jina AI's Reader API (`r.jina.ai/{url}`) is synchronous and single-page; no native multi-page orchestration. Direct comparison to /crawl's async multi-page model favors /crawl for site-wide ingestion.

**Tensions noted:**
- The irony noted in HN #47329557 is real: Cloudflare sells bot protection products and the /crawl crawler is itself subject to those products on target sites, without a bypass mechanism. This is not a minor limitation — it means /crawl is unreliable against the large fraction of the web running Cloudflare's own WAF and Bot Management.
- The docs explicitly acknowledge: "Requests from Browser Rendering will always be identified as a bot." Source: [Cloudflare Docs — /crawl, user agent note](https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/)

---

### 5. Integration angles

**Vault ingestion pipeline:**
- `render: false` + `formats: ["markdown"]` is the right call for static documentation sites (e.g., crawling a library's docs to load into the vault). No browser hours consumed (free in beta). Fast. Returns clean Markdown. Set `includePatterns` to docs paths; `excludePatterns` for changelogs and archives.
- `render: true` + `formats: ["markdown"]` for SPA docs sites (e.g., React-based reference docs). Will consume browser hours.
- `modifiedSince` enables incremental re-ingestion — run periodically, only pick up updated pages. Pairs well with a cron-triggered Worker.

**Research pipeline:**
- Current web_fetch is synchronous and single-page. /crawl is async and multi-page. They are complementary, not substitutes: use web_fetch for targeted single-URL retrieval (current pattern), use /crawl for "ingest entire site as a corpus" tasks.
- The async model means /crawl cannot be dropped into a synchronous research pipeline without polling logic. Integration requires a job-submission → polling → results-retrieval pattern, which adds latency and complexity vs. simple web_fetch calls.
- For research tasks that currently involve multiple sequential web_fetch calls to crawl a documentation site, /crawl would be a meaningful throughput improvement — one API call instead of N sequential calls.

**Agent stack (RAG, knowledge base, content monitoring):**
- Official Cloudflare use cases match vault use cases: "building knowledge bases" and "RAG applications" are listed first. A crawl job → chunk → embed → Cloudflare Vectorize pipeline is the documented pattern.
- Content monitoring: `modifiedSince` + cron → only retrieve pages changed since last run → diff/summarize changes → notify. This is a low-cost incremental pattern.
- The `/json` format + `jsonOptions.prompt` + JSON Schema enables structured extraction at crawl time — returning typed objects rather than free-text Markdown. Useful for product catalog or pricing table ingestion where downstream pipeline expects structured data, but incurs Workers AI costs.

**What it does not replace:**
- Targeted, single-URL retrieval (web_fetch is simpler and synchronous).
- Sites protected by Cloudflare Bot Management (crawler is blocked by same rules).
- Workflows requiring stateful browser sessions (use Workers Bindings + Playwright instead).
- Anti-bot bypass scenarios (use Scrapling or similar; see tools/python/scrapling.md).

---

## source tensions

- **Free tier crawl job limit (5/day) vs. practical usability:** 5 crawl jobs/day on Free is very restrictive for iterative development. The docs do not state a corresponding Paid plan daily job limit — only that it is governed by the browser hour budget. This gap exists; the assumption is Workers Paid imposes no hard per-day job ceiling (only the browser hour cap), but this is not explicitly confirmed in the limits table for Paid users.

- **`limit` default of 10 vs. stated "default is 100,000":** The optional parameters table shows `depth` defaults to 100,000. For `limit`, the description says "default is 10, maximum is 100,000." These are different defaults — `limit` is conservatively capped at 10 by default; `depth` is not. Developers who expect full-site crawls without setting `limit` explicitly will get only 10 pages.

- **render: false post-beta pricing:** Docs state render: false crawls "will be billed under Workers pricing" after beta, but do not specify the rate or timing of the beta end. Workers pricing is currently $0.30/million CPU-ms; the actual cost per render: false crawl page is not estimable from available docs.

- **HN criticism vs. official positioning:** The docs present /crawl as a general-purpose crawler. HN community testing reports it fails on Cloudflare-protected sites. The docs acknowledge this (bot rules apply) but frame it as a configuration issue (add a WAF skip rule for your own sites). This means /crawl works well for self-owned sites or open sites, but poorly for third-party sites with bot protection — a relevant constraint for research use cases targeting arbitrary web destinations.

- **Workers AI cost for JSON format:** The docs say JSON mode "incurs usage on Workers AI" but provide no per-page cost estimate or example. Workers AI pricing is model-dependent. This is a genuine pricing gap for any pipeline planning to use JSON mode at scale.

---

## related vault nodes

- `tools/python/scrapling.md` — factual overlap on web scraping capabilities, anti-bot approaches, and output formats; both tools address the same scraping problem space via different mechanisms.

---

## sources

1. Cloudflare Docs — Browser Rendering overview: https://developers.cloudflare.com/browser-rendering/
2. Cloudflare Docs — /crawl endpoint (primary reference): https://developers.cloudflare.com/browser-rendering/rest-api/crawl-endpoint/
3. Cloudflare Docs — Browser Rendering Pricing: https://developers.cloudflare.com/browser-rendering/pricing/
4. Cloudflare Docs — Browser Rendering Limits: https://developers.cloudflare.com/browser-rendering/limits/
5. Cloudflare Docs — REST API overview: https://developers.cloudflare.com/browser-rendering/rest-api/
6. Hacker News thread #47329557 (community signal, secondary source): https://news.ycombinator.com/item?id=47329557
