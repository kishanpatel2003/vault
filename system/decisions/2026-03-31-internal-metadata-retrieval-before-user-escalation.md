---
title: "Internal metadata retrieval must exhaust local sources before user escalation"
type: decision
created: 2026-03-31
last_updated: 2026-03-31
tags: [decisions, reliability, retrieval, agents]
---

# Decision

before asking kishan for any agent, fleet, workspace, vault, model, role, id, access, or version-control metadata, diana (and any agent she manages) must exhaust the local retrieval ladder first.

## rationale

a prior bryan failure incorrectly treated an empty semantic memory search as proof that metadata was unavailable, then asked kishan to provide recoverable information manually. the actual issue was retrieval process failure, not lack of local data.

## retrieval ladder

1. `~/vault/system/agent-registry.md`
2. agent-local identity/spec files
3. agent workspace files
4. main workspace `AGENTS.md`
5. vault system docs (`fleet-architecture.md`, `memory-architecture.md`)
6. runtime/status/config surfaces

if one step returns nothing, continue to the next step.

## escalation rule

only ask kishan after the full ladder has been checked.

when escalation is still necessary, report:
- what was recovered
- what remains ambiguous
- which paths were checked

## implications

- semantic search is one retrieval tool, not the whole retrieval system
- internal metadata lookup must be deterministic where possible
- failure to continue the ladder is an operational reliability failure
- this same discipline applies to proactive updates on delegated/background tasks: a changed state requires follow-through, not silence
