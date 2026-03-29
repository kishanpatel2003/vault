---
last_updated: 2026-03-28
last_entry: local-voice-conversation-stack.md
maintained_by: fetch
---

# VAULT MAP

## personal/
Context about Kishan — biographical, preferences, logs.
- kishan/bio.md — Static background, career history, technical skills
- kishan/preferences.md — Work style, values, risk tolerance, decision patterns

## tools/
Research briefs on open-source libraries, frameworks, and platforms.
- tools/python/ — Python library research.
  - tools/python/scrapling.md — Adaptive web scraping framework: anti-bot bypass, element tracking, spider framework. 17.9k GitHub stars, pre-1.0 (v0.4.1), single maintainer (as of Mar 2026).
- tools/research/ — General tool research not tied to a language.
  - tools/research/cloudflare-crawl-endpoint.md — Cloudflare Browser Rendering /crawl endpoint (open beta Mar 2026): async multi-page crawl API, HTML/Markdown/JSON output, pricing, limits, API usage, comparisons, integration patterns.
  - tools/research/local-voice-conversation-stack.md — Local-first two-way voice on macOS Apple Silicon (Mar 2026): whisper.cpp (Metal) for STT, Kokoro-82M via FastAPI for TTS, OpenClaw Telegram integration, install steps, MVP and long-term stack options.

## workflows/
Research briefs on how builders and teams use AI coding agents in practice.
- workflows/research/ — Practitioner workflow patterns.
  - workflows/research/coding-agent-builder-patterns.md — How real builders use coding agents daily (Q1 2026): iterative pair-programming dominates; CLAUDE.md + spec-first for context; multi-agent delegation via artifact handoffs; documented failure modes in autonomous setups.

## missions/
One subdirectory per active business venture. Also includes a `general/` directory for research not tied to a specific venture.
- general/ — General research not tied to a specific mission.
  - general/research/harness-engineering.md — What harness engineering is, how it differs from prompt engineering, Anthropic and OpenAI best practices, tooling patterns, evaluation frameworks, field direction (as of Mar 2026).
  - general/research/harness-engineering-eval.md — Self-assessment eval for harness-engineering.md (18/18).
  - general/research/claude-code-auto-mode.md — Claim verification of @rohanpaul_ai tweet on Anthropic's Claude Code auto mode announcement (Mar 2026). Largely accurate; one date error, one omitted caveat.
  - general/research/anthropic-openclaw-subscription-block.md — Anthropic's January 2026 technical block and February 2026 policy formalization banning Claude subscription OAuth tokens in third-party tools including OpenClaw. Accounts not canceled; API-key users unaffected (Mar 2026).
  - general/research/openai-vs-anthropic-agentic-remix-pipeline.md — Provider comparison for agentic audio DSP + LLM pipeline: GPT-5.4 vs Claude Opus 4.6 on capability benchmarks, auth/licensing, API pricing, and 100-track/week cost estimates. Flags subscription rate-limit gotcha and OpenAI's current permissive OAuth stance (Mar 2026).
- polymarket/ — Prediction market regulatory landscape, CFTC jurisdiction, and enforcement research.
  - polymarket/research/prediction-market-regulation.md — U.S. regulatory status: CFTC enforcement history, Polymarket re-entry approval, state gaming law vs. federal preemption circuit split (as of Mar 2026).
