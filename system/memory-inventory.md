---
title: "Memory Inventory & Classification"
type: system
status: canonical
created: 2026-03-29
last_updated: 2026-03-29
---

# Memory Inventory & Classification

complete audit of all memory surfaces as of 2026-03-29.

---

## workspace memory files (`~/.openclaw/workspace/memory/`)

| file | size | classification | action |
|------|------|---------------|--------|
| `2026-01-31.md` | 2.0K | raw intake (old) | archive — pre-system, low value |
| `2026-02-12-2125.md` | 5.4K | raw intake (old, bad naming) | archive — rename if kept |
| `2026-02-18-0218.md` | 5.3K | raw intake (old, bad naming) | archive — rename if kept |
| `2026-03-05.md` | 2.6K | raw intake | keep as staging, scan for promotable facts |
| `2026-03-06.md` | 2.5K | raw intake | keep as staging, contains cron/launchagent lesson (already in MEMORY.md) |
| `2026-03-24.md` | 2.3K | raw intake | keep as staging |
| `2026-03-25.md` | 5.4K | raw intake | keep as staging |
| `2026-03-26.md` | 5.8K | raw intake | keep as staging |
| `2026-03-27.md` | 8.6K | raw intake | keep as staging |
| `2026-03-28.md` | 10.9K | raw intake (mostly heartbeat) | keep as staging, heavy heartbeat noise |
| `2026-03-29.md` | 8.3K | raw intake (mostly heartbeat) | active today |

**naming issue:** two files use `YYYY-MM-DD-HHMM.md` format instead of `YYYY-MM-DD.md`. standardize going forward.

**heartbeat noise:** daily logs from 3/28 and 3/29 are ~80% heartbeat entries repeating "no substantive new work." need heartbeat format reduction.

---

## MEMORY.md (`~/.openclaw/workspace/MEMORY.md`)

| section | classification | action |
|---------|---------------|--------|
| about bryan | active state + operational rules | promote bryan-relevant facts to vault `health/baselines.md`; operational rules stay in bryan agent-spec |
| about kishan | permanent canonical | already in vault `personal/kishan/bio.md` — deduplicate |
| patterns noticed | empty | remove section or populate during review |
| open threads | active state | migrate to vault `system/active-threads.md` |
| lessons learned (cron vs launchagent) | permanent canonical | promote to vault `system/lessons.md` |
| lessons learned (gateway resilience) | permanent canonical | promote to vault `system/lessons.md` |

**68 lines currently.** well under 500-line limit but contains competing-store content.

**recommendation:** migrate promotable content to vault, then either retire or reduce to a thin pointer file.

---

## decisions/ (`~/.openclaw/workspace/decisions/`)

| file | classification | action |
|------|---------------|--------|
| `2026-03-04-benchmarking-too-slow-use-anthropic-faster-model.md` | permanent canonical | good format. migrate to vault `system/decisions/` |

**only one decision record exists.** the system is underused — significant decisions from sessions have not been captured as records.

---

## bryan operational logs (`~/.openclaw/workspace/bryan/`)

| surface | classification | action |
|---------|---------------|--------|
| `food-log/` (1 photo + log.md) | operational log | keep in place, add to bryan startup retrieval |
| `progress-pics/` (1 photo + log.md) | operational log | keep in place, add to bryan startup retrieval |
| `weight-log.md` (1 entry) | operational log | keep in place, already in bryan startup retrieval plan |

**all started 2026-03-29.** too early for vault promotion. first weekly summary after ~7 days of data.

---

## bryan agent files (`~/.openclaw/agents/bryan/agent/`)

| file | classification | action |
|------|---------------|--------|
| `SOUL.md` | agent identity | keep — defines bryan personality |
| `IDENTITY.md` | agent identity | keep |
| `USER.md` | agent identity | keep |
| `agent-spec.md` | agent identity + operational rules | keep — contains photo handling modes, needs startup retrieval additions |
| `auth-profiles.json` | operational config | keep |
| `models.json` | operational config | keep |

**gap:** bryan has no startup retrieval contract. no `AGENTS.md` equivalent. no instruction to load operational logs at session start.

---

## vault notes (`~/vault/`)

| area | notes | classification | status |
|------|-------|---------------|--------|
| `personal/kishan/bio.md` | kishan profile | permanent canonical | good, current |
| `personal/kishan/preferences.md` | kishan prefs | permanent canonical | good, current |
| `tools/python/scrapling.md` | research | permanent canonical | good |
| `tools/research/cloudflare-crawl-endpoint.md` | research | permanent canonical | good |
| `tools/research/local-voice-conversation-stack.md` | research | permanent canonical | good |
| `workflows/research/coding-agent-builder-patterns.md` | research | permanent canonical | good |
| `missions/general/research/*` (5 notes) | research | permanent canonical | good |
| `missions/polymarket/research/*` (1 note) | research | permanent canonical | good |

**gaps:**
- no `system/` directory (until this refactor)
- no `health/` directory
- no decision records in vault
- no active-threads tracking
- no system lessons captured
- VAULT_MAP.md has a duplicate entry for local-voice-conversation-stack.md

---

## session transcripts

**classification:** ephemeral context, not memory
**action:** not part of memory architecture. used only for `memory_search` semantic recall. never promoted raw.

---

## summary of gaps

1. **bryan has no startup retrieval contract** — can't recall yesterday's context
2. **MEMORY.md competes with vault** — dual long-term stores
3. **decision capture is weak** — one record in months of operation
4. **vault has no system/ or health/ areas** — nowhere to put lessons, decisions, health summaries
5. **daily logs are heartbeat-heavy** — signal-to-noise ratio is poor
6. **VAULT_MAP.md has a duplicate entry** — integrity issue
7. **two daily files use wrong naming convention** — minor cleanup
8. **no active-threads tracking in vault** — open threads live only in MEMORY.md
