---
title: "Scrapling — Adaptive Web Scraping Framework for Python"
type: research_brief
date: 2026-03-05
requested_by: diana
tags: [python, web-scraping, anti-bot, adaptive-scraping, open-source, tooling]
mission: null
sources_count: 7
summary: "Scrapling is a Python 3.10+ adaptive web scraping framework (17.9k GitHub stars, ~298k PyPI downloads) that combines a fast lxml-based parser with built-in anti-bot bypass, adaptive element tracking, and a Scrapy-like spider framework, maintained by a single developer with rapid release cadence but pre-1.0 API stability."
---

## background

Scrapling is an open-source Python library created by Karim Shoair (GitHub: D4Vinci), first publicly announced in October 2024. It is positioned as a unified web scraping toolkit that integrates parsing, HTTP fetching, headless browser automation, anti-bot bypass, adaptive element tracking, and a full spider/crawling framework into a single package — addressing use cases that previously required combining Scrapy, BeautifulSoup, Playwright, and separate stealth libraries.

As of March 2026, the project is at version 0.4.1 (pre-1.0), licensed under BSD-3-Clause, and hosted at:
- **Repo:** https://github.com/D4Vinci/Scrapling
- **Docs:** https://scrapling.readthedocs.io
- **PyPI:** https://pypi.org/project/scrapling/

Research scope: capabilities, tool comparisons, anti-bot mechanisms, adaptive tracking, spider framework, community traction, maturity, docs quality, limitations.

---

## key findings

- **What it does.** Scrapling provides three stacked layers: (1) a parser (`Selector`) based on lxml/Parsel for CSS, XPath, text, and regex selection; (2) four fetcher classes (`Fetcher` for plain HTTP+TLS impersonation, `StealthyFetcher` for anti-bot headless browsing via patchright, `DynamicFetcher` for full Playwright Chromium/Chrome automation, and async variants of each); (3) a spider framework introduced in v0.3 (Feb 15, 2026) for concurrent, multi-session crawls with pause/resume. Source: GitHub README, https://github.com/D4Vinci/Scrapling

- **Adaptive element tracking.** The library's differentiating concept is that scraped elements can be saved with `auto_save=True` on first scrape; on subsequent runs with `adaptive=True`, the parser uses a similarity algorithm to re-locate elements even if the page structure has changed. This is claimed to be ~5× faster than the closest alternative (AutoScraper). Source: GitHub README benchmarks, https://github.com/D4Vinci/Scrapling

- **Anti-bot bypass.** `StealthyFetcher` (formerly backed by Camoufox, switched to patchright in a recent release) can bypass Cloudflare Turnstile and Cloudflare Interstitial challenges. `Fetcher` supports TLS fingerprint impersonation (Chrome, Firefox) and HTTP/3. The project claims "undetectable" status. Source: GitHub README; Releases page, https://github.com/D4Vinci/Scrapling/releases

- **Spider framework.** Added in v0.3 (Feb 15, 2026): Scrapy-like API with `start_urls`, async `parse` callbacks, `Request`/`Response` objects, configurable concurrency, per-domain throttling, download delays, multi-session routing (mix HTTP and stealthy browser in one spider by session ID), checkpoint-based pause/resume, streaming mode, lifecycle hooks, and built-in JSON/JSONL export. Built on `anyio` with optional `uvloop` support. Source: GitHub Releases, https://github.com/D4Vinci/Scrapling/releases

- **Performance benchmarks (self-reported).** Scrapling's parser benchmarked at 2.02ms for a 5,000-element extraction test, near-parity with Parsel/Scrapy (2.04ms), and ~784× faster than BS4+lxml (1,584ms). Benchmarks run as averages of 100+ runs; methodology at `benchmarks.py` in repo. Source: GitHub README, https://github.com/D4Vinci/Scrapling

- **Community traction.** GitHub: **17,900 stars**, **1,200 forks** as of early March 2026. PyPI: version 0.4.1, approximately **298,000 total downloads**. Listed on TrendShift (repo #14244). Active Discord server and @Scrapling_dev on X. Reddit r/webscraping post for v0.2.99 (Apr 2025) received 158 upvotes, 58 comments. Multiple commercial proxy/scraping service sponsors (Evomi, SerpAPI, Decodo, HasData, ProxyEmpire) visible in README. Source: GitHub search result snippet; PePy.tech (https://pepy.tech/projects/scrapling); Reddit (https://www.reddit.com/r/webscraping/comments/1jugawo)

- **Maintenance activity.** Two major releases in February 2026 alone (v0.3 on Feb 15, v0.4.1 on Feb 27). Active PR merges from external contributors (RinZ27, robin-ede, salmanmkc visible in v0.4.1 release notes). GitHub issue count: 3 open issues as of March 2026 — notably low, consistent with rapid triage. Source: GitHub Issues, https://github.com/D4Vinci/Scrapling/issues

- **Documentation quality.** Dedicated ReadTheDocs site launched April 2025, following the Diataxis documentation framework (tutorials, how-tos, reference, explanation). README available in 7 languages (Arabic, Spanish, French, German, Chinese, Japanese, Russian). Auto-generated API reference from source. Migration guide (BeautifulSoup → Scrapling) included. Source: https://scrapling.readthedocs.io/en/latest/

- **Installation.** Modular: `pip install scrapling` (parser only) → `pip install "scrapling[fetchers]"` + `scrapling install` (downloads browsers) → optional `[ai]`, `[shell]`, `[all]` extras. Docker image auto-built per release. Requires Python ≥ 3.10. Source: PyPI, https://pypi.org/project/scrapling/

- **MCP server.** Built-in Model Context Protocol server for AI-assisted scraping (Claude, Cursor, VS Code Copilot). Targets token efficiency by extracting specific HTML content before passing to the LLM. Source: Docs, https://scrapling.readthedocs.io/en/latest/ai/mcp-server/

---

## comparison to existing tools

| Dimension | Scrapling | BeautifulSoup | Scrapy | Playwright | Selenium |
|---|---|---|---|---|---|
| **Primary role** | Unified fetch + parse + crawl | HTML parsing only | Crawl framework | Browser automation | Browser automation |
| **Anti-bot bypass** | Built-in (StealthyFetcher, TLS impersonation) | None | Via middlewares (manual) | None natively | None natively |
| **Adaptive element tracking** | Yes (core feature) | No | No | No | No |
| **JavaScript rendering** | Yes (DynamicFetcher / StealthyFetcher) | No | Via Splash/Scrapy-Playwright | Yes (native) | Yes (native) |
| **Spider/crawl framework** | Yes (added Feb 2026) | No | Yes (mature, ~10 years) | No | No |
| **Ecosystem/plugins** | Small (pre-1.0) | Large (BS4 extensions) | Large (Scrapy Cloud, middleware ecosystem) | Large (Playwright network, recorder) | Large (Grid, hub services) |
| **Performance (parsing)** | ~2ms (self-benchmarked) | ~1,584ms (BS4+lxml per Scrapling) | ~2ms (Parsel, same engine) | N/A | N/A |
| **API stability** | Pre-1.0, breaking changes noted | Stable | Stable | Stable | Stable |
| **Primary maintainer(s)** | 1 (D4Vinci) | Community | Zyte + community | Microsoft | Selenium HQ |

Key differentiator vs. Scrapy: Scrapling trades Scrapy's mature ecosystem and production track record for integrated stealth/anti-bot and adaptive tracking in fewer lines of code. The spider API is explicitly modeled after Scrapy but is very new (6 weeks old as of research date). Source: GitHub README; ScrapingBee comparison (https://www.scrapingbee.com/blog/best-python-web-scraping-libraries/)

---

## limitations, known issues, and red flags

- **Single maintainer / bus factor.** The entire project is driven by one developer (Karim Shoair). No evidence of organizational backing. Source: GitHub contributor history (single-sourced inference from PR author list).

- **Pre-1.0 API instability.** The project is at v0.4.x. The v0.3 release notes explicitly note "breaking changes" and instruct users to review carefully before upgrading. The StealthyFetcher backend changed from Camoufox to patchright between releases with API-level impact. Source: GitHub Releases, https://github.com/D4Vinci/Scrapling/releases

- **Spider framework is brand new.** The crawl/spider layer was introduced February 15, 2026 — only ~18 days before this research. It cannot yet be considered battle-tested for production crawling workloads. Source: GitHub Releases.

- **Self-reported benchmarks.** All performance claims originate from `benchmarks.py` in the Scrapling repo itself. No independent third-party benchmark reproducing these results was found. Source: GitHub README.

- **Python 3.14 incompatibility.** Open issue #104 as of March 2026: `CamoufoxConfig` fails on Python 3.14. (Note: the library has since migrated to patchright, which may resolve this, but the issue remains open.) Source: GitHub Issues, https://github.com/D4Vinci/Scrapling/issues/104

- **Anti-bot bypass arms race.** Cloudflare actively updates its bot detection. The library's claims of bypassing Cloudflare Turnstile/Interstitial may degrade with future Cloudflare updates. No durability guarantees. Source: General domain knowledge; release notes show ongoing bypass refinements.

- **Heavy optional dependency footprint.** Full installation (fetchers + browsers) requires downloading Chromium, patchright binaries, fingerprint libraries. Adds significant disk and setup complexity. Source: PyPI, README.

- **Legal disclaimer.** The library's README includes a prominent caution: "provided for educational and research purposes only… Always respect the terms of service of websites and robots.txt files." Anti-bot bypass use may violate ToS of targeted sites. Source: GitHub README.

---

## source tensions

- **Download count.** PePy.tech reported ~298k total downloads for version 0.4.1. This is a lifetime cumulative figure; per-period download rates were not available at the time of research, making traction assessment relative to peer libraries difficult. Only one source (PePy) for this data point.

- **"Battle tested" claim vs. project age.** The README states "used daily by hundreds of Web Scrapers over the past year." The project was first publicly announced in October 2024, making "over the past year" accurate in duration but short in absolute time. The spider framework, a major component, is 18 days old. These claims are internally consistent but represent a relatively short track record.

- **Performance benchmarks.** The README's benchmark table shows Scrapling and Parsel/Scrapy at near-identical speeds (2.02ms vs. 2.04ms), which is expected given both use lxml under the hood. The benchmark's framing of "Scrapling vs. BS4" comparisons (784× faster) conflates the parser with BS4's use of a slower html5lib parser — a favorable but not misleading comparison. No independent verification found.

- **Anti-bot effectiveness.** The project describes bypass as "out of the box" and "easily bypass all types of Cloudflare's Turnstile." The recent switch from Camoufox to patchright (described as improving stability and speed) also coincided with an open Python 3.14 issue in the prior implementation. Sources agree stealth fetching works but maintenance of effectiveness requires ongoing updates.

---

## related vault nodes

- *(No existing vault nodes on web scraping tooling, Python libraries, or anti-bot research found in current VAULT_MAP.)*
