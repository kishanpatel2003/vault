---
title: "Agent Registry"
type: system
status: canonical
created: 2026-03-31
last_updated: 2026-03-31
tags: [agents, fleet, registry, metadata]
---

# Agent Registry

canonical source of truth for agent metadata. if a field here conflicts with another doc, fix the other doc.

## required fields

- agent
- id
- role
- default model
- workspace
- repo
- vault access
- vc strategy
- startup contract path
- notes

## agents

| agent | id | role | default model | workspace | repo | vault access | vc strategy | startup contract path | notes |
|-------|----|------|---------------|-----------|------|--------------|-------------|-----------------------|-------|
| diana | main | orchestrator + interface | haiku 4.5 | `~/.openclaw/workspace/` | `kishanpatel2003/openclaw-diana` | read canonical vault; curate/promote only | pr-first | `/Users/kpmacmini/.openclaw/workspace/AGENTS.md` | main interface to kishan; delegates research/coding |
| fetch | fetch | research synthesis | sonnet 4.6 | `~/.openclaw/workspace-fetch/` | `kishanpatel2003/openclaw-workspace-fetch` | write `~/vault/fetch/`; read broader vault as needed | auto-merge | `~/.openclaw/workspace-fetch/AGENTS.md` | research outputs are fetch-owned until promoted |
| selena | selena | general purpose coding (claude code) | opus 4.6 | `~/.openclaw/workspace-selena/` | `kishanpatel2003/openclaw-selena` | read vault via specs; no direct canonical writes by default | auto-merge | `~/.openclaw/workspace-selena/CLAUDE.md` | engineering executor |
| bryan | bryan | dedicated health performance advisor | sonnet 4.6 | `~/.openclaw/workspace/bryan/` | local workspace only (`~/.openclaw/workspace/bryan/.git`), no standalone remote repo documented | read `~/vault/health/`; no vault writes | local-only | `~/.openclaw/agents/bryan/agent/agent-spec.md` | identity/spec lives in `~/.openclaw/agents/bryan/agent/`; operational context lives in `~/.openclaw/workspace/bryan/` |

## retrieval rule

before asking kishan for any internal agent or fleet metadata, check in this order:
1. this registry
2. agent-local identity/spec files
3. agent workspace files
4. main workspace `AGENTS.md`
5. vault system docs (`fleet-architecture.md`, `memory-architecture.md`)
6. runtime/status/config surfaces

if one source is missing or empty, continue. do not escalate after a single miss.

if ambiguity remains after all checks, report:
- what was recovered
- what remains ambiguous
- exactly which local paths were checked
