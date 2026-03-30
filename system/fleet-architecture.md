---
title: "Agent Fleet Architecture"
type: permanent_canonical
created: 2026-03-05
last_updated: 2026-03-29
tags: [fleet, agents, infrastructure]
---

# Agent Fleet Architecture

established 2026-03-05. diana's home is kishan's mac mini (always-on, tv console).

## agents

| agent | role | model | workspace | repo | vc strategy |
|-------|------|-------|-----------|------|-------------|
| diana | orchestrator, CEO of fleet | haiku 4.5 (default) | `~/.openclaw/workspace/` | kishanpatel2003/openclaw-diana | pr-first |
| fetch | research synthesis | sonnet 4.6 | `~/.openclaw/workspace-fetch/` | kishanpatel2003/openclaw-workspace-fetch | auto-merge |
| selena | coding (claude code) | opus 4.6 | `~/.openclaw/workspace-selena/` | kishanpatel2003/openclaw-selena | auto-merge |
| bryan | health advisor | sonnet 4.6 | `~/.openclaw/agents/bryan/agent/` | (agent config, not standalone repo) | n/a |

## shared resources

- **vault:** `~/vault/` → kishanpatel2003/vault — canonical durable memory, obsidian-viewable
- **model chain:** primary `openai-codex/gpt-5.4`, fallbacks opus 4.6, sonnet 4.6, qwen 14b

## governance rules

1. **github visibility:** if it exists locally, it must exist on github. no exceptions.
2. **diana is orchestrator, not executor.** anything beyond ~20 lines of code or multi-source research gets delegated.
3. **diana is pr-first.** identity-critical changes require manual review before merge.
4. **fetch and selena auto-merge.** they are executors, not identity-critical.
5. **kishan speaks to diana. diana speaks to sub-agents.** chain of command.

## key infrastructure

- gateway runs on mac mini as LaunchAgent (KeepAlive=true)
- telegram channels: general / work / build / life groups + bryan health group
- openclaw cron for scheduled tasks (reminders, health check-ins)
