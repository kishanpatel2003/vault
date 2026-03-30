# tools/research/ — Index

General tool research briefs not tied to a specific programming language or mission.

| file | date | summary | tags |
|------|------|---------|------|
| cloudflare-crawl-endpoint.md | 2026-03-11 | Cloudflare Browser Rendering /crawl endpoint (open beta, Mar 2026): async multi-page crawl API returning HTML/Markdown/AI JSON. Free tier: 5 jobs/day, 100 pages. Paid: 10 hrs/month + $0.09/hr overage. | cloudflare, web-scraping, browser-rendering, crawling, api, rag |
| local-voice-conversation-stack.md | 2026-03-28 | Local-first two-way voice on macOS Apple Silicon: whisper.cpp (Metal) for STT (~0.3–1.2s), Kokoro-82M via FastAPI for TTS (~90–200ms), both integrating with OpenClaw's existing config primitives. 12 sources. | voice, stt, tts, whisper, kokoro, openclaw, apple-silicon, local-first, telegram |
| aristotle-memory-plugin.md | 2026-03-30 | Aristotle is a single-maintainer OpenClaw plugin (TypeScript, ~3k lines) for proactive memory protection: before_tool_call hook enforces 10 Guard rules, context shield monitors window pressure, nightly QC cron + Telegram reports. 6 days old, no releases, no community. Watchlist recommendation. | openclaw, memory, plugin, fleet, agents, compaction, guard, qc |
| aristotle-utility-analysis.md | 2026-03-30 | Utility-focused follow-up: strips repo maturity framing. Classifies Aristotle as a memory policy engine / middleware pattern. Recommends cloning 3 concepts internally now: per-agent path enforcement plugin, vault integrity cron, diana context pressure rule. Bryan scope enforcement identified as highest-priority first action. | openclaw, memory, plugin, fleet, architecture, policy, middleware, bryan, vault |
