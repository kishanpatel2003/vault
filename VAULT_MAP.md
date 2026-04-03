---
last_updated: 2026-04-03
maintained_by: diana
---

# VAULT MAP

## personal/
Context about Kishan — biographical, preferences.
- personal/kishan/bio.md — static background, career history, technical skills
- personal/kishan/preferences.md — work style, values, risk tolerance, decision patterns

## system/
System-level knowledge: architecture, lessons, decisions, active threads.
- system/memory-architecture.md — canonical memory architecture spec: layers, retrieval, promotion, owner-first write contracts
- system/memory-inventory.md — full audit and classification of all memory surfaces (Mar 2026)
- system/active-threads.md — open threads requiring follow-up or monitoring
- system/lessons.md — durable operational lessons
- system/fleet-architecture.md — agent fleet architecture: agents, roles, models, governance rules
- system/decisions/ — decision records for significant choices

## health/
Bryan/shared canonical health memory. Baselines, longitudinal summaries, protocols.
- health/baselines.md — Bryan baseline data: starting weight 151.3 lbs, vegetarian diet rules, tracking start 2026-03-29
- health/summaries/ — weekly/monthly health summaries (promoted from bryan logs)
- health/protocols/ — durable health protocols and operating guidance

## fetch/
Fetch-owned research library. Not canonical shared memory until explicitly promoted.
- fetch/general/ — cross-cutting research
  - fetch/general/coding-agent-builder-patterns.md — how real builders use coding agents daily (Q1 2026)
  - fetch/general/diana-completion-update-failure-audit.md — root cause audit of diana's proactive completion update failures (Mar 2026)
- fetch/health/ — health-related research (distinct from canonical health/)
  - fetch/health/vesync-smart-scale-integration.md — VeSync/Etekcity scale integration feasibility (Mar 2026)
- fetch/tools/ — library, framework, and platform research
  - fetch/tools/scrapling.md — adaptive Python web scraping framework (17.9k stars, pre-1.0)
  - fetch/tools/eval_scrapling.md — self-assessment eval for scrapling.md (15/18)
  - fetch/tools/cloudflare-crawl-endpoint.md — Cloudflare Browser Rendering /crawl endpoint (open beta Mar 2026)
  - fetch/tools/local-voice-conversation-stack.md — local-first voice conversation on macOS Apple Silicon (Mar 2026)
  - fetch/tools/aristotle-memory-plugin.md — Aristotle OpenClaw memory protection plugin (Mar 2026)
  - fetch/tools/aristotle-utility-analysis.md — utility analysis addendum for Aristotle patterns (Mar 2026)
- fetch/missions/ — mission-related research briefs and evals
  - fetch/missions/harness-engineering.md — harness engineering practices and evaluation frameworks (Mar 2026)
  - fetch/missions/harness-engineering-eval.md — self-assessment eval (18/18)
  - fetch/missions/claude-code-auto-mode.md — claim verification of Claude Code auto mode announcement (Mar 2026)
  - fetch/missions/anthropic-openclaw-subscription-block.md — Anthropic subscription token block analysis (Jan-Feb 2026)
  - fetch/missions/openai-vs-anthropic-agentic-remix-pipeline.md — provider comparison for agentic audio DSP pipeline (Mar 2026)
  - fetch/missions/prediction-market-regulation.md — U.S. prediction market regulatory status: CFTC, Polymarket, state-level analysis (Mar 2026)

## missions/
Canonical mission state only. Raw research lives in fetch/missions/.
- missions/polymarket/ — prediction market strategy and regulatory landscape
- missions/general/ — general research not tied to a specific venture
  - missions/general/research/harness-engineering.md — harness engineering: definition, best practices (Anthropic Nov 2025–Mar 2026, OpenAI early 2026), tooling, eval frameworks, field direction. 13 sources. (Apr 2026)
  - missions/general/research/harness-engineering-eval.md — self-assessment eval, 18/18
