---
title: "Aristotle — Memory Protection Plugin for OpenClaw"
type: research_brief
date: 2026-03-30
requested_by: kishan
tags: [openclaw, memory, plugin, fleet, agents, compaction, guard, qc]
mission: fleet / agents / leverage
sources_count: 5
summary: "Aristotle is a single-maintainer OpenClaw plugin (~3,000 lines TypeScript) that adds proactive memory protection via a before_tool_call hook: it blocks/redirects memory-corrupting agent actions, monitors context window pressure, and runs nightly integrity checks. Released March 2026, no prior releases, 0 open issues, 1 contributor."
---

## background

Aristotle is an OpenClaw plugin created to address a specific failure mode: agents losing context mid-session due to compaction (OpenClaw's automatic compression of older conversation turns when working memory fills up). The plugin targets the OpenClaw-specific before_tool_call hook to intercept file writes, shell commands, and sub-agent spawns before execution. It is not a general agent framework, orchestrator, or coding tool — it is narrowly scoped to memory integrity within OpenClaw sessions.

The GitHub org `aristotle-agent` joined March 24, 2026. The repo contains one project (aristotle). No releases have been tagged. No open issues exist. The discussion board requires login to view. Based on GitHub join date and README framing ("I made this because I got tired of waking up to a blank agent"), this is a solo creator project less than two weeks old at time of research.

Source: GitHub org profile — https://github.com/aristotle-agent; README — https://github.com/aristotle-agent/aristotle

## key findings

- **Architecture: four-component plugin.** Aristotle implements (1) Guard — a before_tool_call hook enforcing 10 memory protection rules via code-level redirects rather than blocks; (2) Context Shield — monitors working memory % via JSONL transcript parsing every 50 tool calls and triggers proactive compression at 65% or session reset at 70%; (3) QC Nightly — 11 automated integrity checks run as cron at 11:15 PM, generates a deterministic TypeScript-formatted Telegram report at 11:20 PM; (4) QC Mid-Session — two lightweight checks every 50 actions for write-failure and backup-gap detection. Source: README — https://github.com/aristotle-agent/aristotle

- **Deployment model: cron + plugin hook.** Setup wizard auto-detects Telegram chat ID and workspace path from existing OpenClaw config. Installs four cron jobs (3:30 AM pre-reset checkpoint, 10:45 PM continuity update, 11:15 PM QC, 11:20 PM report). Claims "four questions, sensible defaults." Requires `openclaw plugins install --link` (not standard install path) because the hook system requires install provenance; Guard silently fails without this. Source: README install section — https://github.com/aristotle-agent/aristotle

- **Scope is explicitly narrow.** README states: "Doesn't manage tasks or projects. That's the operations layer, coming separately. Aristotle protects memory. Period." Self-reported combined reliability on memory protection: 90–92%. The remaining 8–10% caught by nightly QC or operating file rules. Doesn't replace OpenClaw's built-in compression; adds a proactive layer on top. Source: README limitations section — https://github.com/aristotle-agent/aristotle

- **Maturity: pre-release, single maintainer, no community.** GitHub org created March 24, 2026. No version releases tagged. 0 open issues. Discussions board inaccessible without login. Commits page did not render in fetch (login required). The org shows 1 repository, 1 contributor. README is polished and persuasive but written in a direct-to-consumer sales voice targeting non-technical OpenClaw users ("you now know what Terminal is"), not developers. Source: https://github.com/aristotle-agent, https://github.com/aristotle-agent/aristotle/issues, https://github.com/aristotle-agent/aristotle/releases

- **Token efficiency claim: unverified, single-sourced.** Creator claims token use dropped "from an embarrassing 262k average in my first week to now under 12k" after deploying Aristotle. This is self-reported, no methodology provided, no baseline controlled for learning curve or prompt changes. The 96% reduction figure is presented without evidence. Source: README — single-sourced, unverified. https://github.com/aristotle-agent/aristotle

- **Lock-in surface: moderate.** Aristotle creates four cron jobs, deploys QC agent protocol files, configures file permissions on 7 protected files, and modifies memory directory structure. Uninstall is described as clean ("your agent works exactly like it did before"), but no uninstall command is documented in the README CLI reference. The plugin also changes AGENTS.md behavior (marks it read-only) which would conflict with teams that modify AGENTS.md directly. Source: README Guard rules table — https://github.com/aristotle-agent/aristotle

## source tensions

**Tension 1 — overlap with existing vault architecture.** Kishan's fleet already has a documented memory architecture (vault, SOUL.md, AGENTS.md, IDENTITY.md) with defined write rules, git-backed vault, and session bootstrapping patterns. Aristotle's Guard rules (e.g., "Read-only. Notes go to memory/qc/", "Overwrite MEMORY.md → Append, don't replace") assume a specific memory file structure (`MEMORY.md`, `memory/qc/`) that does not match the current fleet's layout. Installing Aristotle without adapting its rules would create conflicts between Guard's enforced redirects and the fleet's actual file conventions. Source: Aristotle README Guard rules + vault/system/memory-architecture.md.

**Tension 2 — problem diagnosis vs. actual problem.** The README anchors on compaction-induced amnesia as the core pain. The current fleet's architecture document (vault/system/memory-architecture.md) and memory-inventory.md indicate the fleet has already built a multi-layer memory system (vault, per-agent SOUL/IDENTITY files, session bootstrapping). Whether compaction amnesia is currently a live problem for diana, fetch, selena, or bryan is not documented in the vault. Aristotle may solve a problem that has already been mitigated by fleet design. Source: vault/system/memory-architecture.md (internal vault node).

**Tension 3 — no external validation.** All performance claims come from the creator. No independent reviews, no community discussion threads visible, no issue history. The 90–92% reliability figure has no methodology. The token reduction claim has no methodology. This is week-1 software with week-1 marketing copy. Source: GitHub — zero external engagement signals as of 2026-03-30.

## stack comparison

| capability | aristotle | current fleet |
|---|---|---|
| memory protection (compaction guard) | ✅ core feature | ⚠️ partial — AGENTS.md rules, vault conventions, but no code enforcement |
| session continuity / context handoff | ✅ cron-based continuity files | ✅ vault + SOUL/IDENTITY bootstrapping |
| nightly integrity checks | ✅ 11-check QC cron | ❌ not present |
| telegram QC reports | ✅ deterministic nightly report | ❌ not present |
| memory write guardrails (code-enforced) | ✅ before_tool_call hook | ❌ convention-only (AGENTS.md rules) |
| research synthesis | ❌ out of scope | ✅ fetch |
| coding / task execution | ❌ out of scope | ✅ selena |
| orchestration | ❌ out of scope | ✅ diana |
| health context | ❌ out of scope | ✅ bryan |
| vault / retrieval | ❌ out of scope | ✅ vault system |

## recommendation

**WATCHLIST. Do not install now.**

Reasoning:
1. **Too new to trust.** The repo is 6 days old at time of research (GitHub org: March 24, 2026). No releases, no issue history, no external adoption signals. Installing week-1 software that modifies cron, file permissions, and hook enforcement on a production fleet is high risk with no upside evidence.

2. **File structure mismatch.** Aristotle's Guard rules assume `MEMORY.md`, `memory/qc/`, and a specific directory layout. Kishan's fleet uses a different architecture (vault-centric, per-agent workspace files, SOUL/IDENTITY/AGENTS.md patterns). Guard would fire incorrect redirects or silently fail on valid fleet operations without explicit reconfiguration.

3. **Problem may not be live.** The vault already documents a deliberate memory architecture with session bootstrapping. Whether compaction amnesia is an active pain point for any fleet agent is not documented. Before adopting a solution, confirm the problem exists.

4. **One genuine gap it addresses.** Code-enforced write guardrails (before_tool_call Hook → Guard redirects) and nightly integrity checks are not present in the current fleet. These are real capabilities, not redundant. If compaction or memory corruption becomes a documented fleet problem, Aristotle's architecture is a reasonable starting point.

**Watchlist criteria: revisit if (a) compaction-induced context loss becomes a documented fleet issue, (b) Aristotle reaches a tagged release with >1 contributor or community engagement, or (c) repo accumulates >2 months of commit history without abandonment.**

**If adopted, integration path would be:**
- Sandbox test only in a non-production agent (new test workspace)
- Audit Guard rules against fleet file conventions before enabling enforce mode
- Replace Aristotle's assumed file layout with fleet-actual paths (vault/, workspace/ structures)
- Verify cron jobs do not conflict with existing OpenClaw LaunchAgent cron schedule
- Do not install on diana (orchestrator) without manual review of every Guard rule

---

## sources

1. Aristotle GitHub README — https://github.com/aristotle-agent/aristotle (primary, self-authored by creator)
2. aristotle-agent GitHub org profile — https://github.com/aristotle-agent (join date, repo count, contributor count)
3. Aristotle Issues page — https://github.com/aristotle-agent/aristotle/issues (0 open issues, no history)
4. Aristotle Releases page — https://github.com/aristotle-agent/aristotle/releases (no releases)
5. vault/system/fleet-architecture.md + vault/system/memory-architecture.md (internal — current fleet state for comparison)
