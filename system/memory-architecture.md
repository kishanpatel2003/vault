---
title: "Memory Architecture Spec v1"
type: system
status: canonical
created: 2026-03-29
last_updated: 2026-03-29
---

# Memory Architecture

this is the controlling document for how memory works across all agents. no competing specs.

---

## 1. layers

### layer A — vault (canonical durable memory)

**location:** `~/vault/`
**access:** diana read-only, fetch read/write, selena read (via specs)
**purpose:** source of truth for anything worth remembering beyond a single session

**belongs here:**
- kishan profile, preferences, patterns, evolving context
- durable system/infrastructure lessons
- decision records
- research briefs and outputs
- mission/project knowledge that persists
- health baselines and longitudinal summaries (bryan-derived)
- agent operating knowledge worth reusing across sessions
- active threads that matter beyond today

**does NOT belong here:**
- raw daily logs
- heartbeat entries
- per-meal food logs
- individual progress pics
- one-off session scratch notes
- every small decision or preference tweak

**format:** markdown, obsidian-native (see section 10)

**human interface:** obsidian. kishan views, navigates, and audits the vault through obsidian. this is not optional compatibility — it's the primary way a human reads the brain.

### layer B — workspace memory (staging / operational)

**location:** `~/.openclaw/workspace/memory/`
**purpose:** short-term intake, daily operational log, heartbeat target

**belongs here:**
- daily raw session notes (`memory/YYYY-MM-DD.md`)
- heartbeat append entries
- temporary observations before promotion
- scratch context for current/recent sessions

**lifecycle:** raw intake. not canonical. reviewed periodically for promotion.

### layer C — agent-local operational state

**location:** agent workspace or workspace subdirectory (e.g., `workspace/bryan/`)
**purpose:** role-specific logs that are operational, not durable memory

**current examples:**
- `bryan/food-log/` — meal photos + macro estimates
- `bryan/progress-pics/` — physique check-in photos
- `bryan/weight-log.md` — daily weight readings

**lifecycle:** operational logs. summarized into vault periodically (weekly/monthly). raw logs are not canonical memory.

### layer D — retrieval (how agents access memory)

retrieval is explicit, layered, and deterministic:

1. **startup load** — deterministic files loaded every session (defined per agent)
2. **path/link lookup** — follow vault indexes and links for specific context
3. **semantic search** — `memory_search` for recall across workspace memory files
4. **vault browse** — navigate via VAULT_MAP.md → _index.md → specific notes

---

## 2. memory classes

| class | description | home | examples |
|-------|-------------|------|----------|
| permanent canonical | durable facts worth keeping indefinitely | vault | kishan bio, system lessons, decisions, research, health baselines |
| active state | live threads, current experiments, evolving context | vault (`system/active-threads.md` or mission-specific) | open projects, current agent constraints, in-progress experiments |
| raw intake | daily session capture, heartbeat noise | workspace `memory/` | daily logs, heartbeat entries, quick observations |
| operational logs | role-specific tracking artifacts | agent-local (e.g., `workspace/bryan/`) | food logs, weight logs, progress pics |

---

## 3. write contract

| artifact type | first written to | canonical? | who promotes | expiry |
|---------------|-----------------|------------|--------------|--------|
| daily session notes | `memory/YYYY-MM-DD.md` | no (staging) | diana during periodic review | kept ~30 days, then archivable |
| heartbeat entries | appended to daily log | no | never promoted raw; signal extracted during review | dies with daily log |
| decisions | `decisions/YYYY-MM-DD-slug.md` | yes (local canonical) | migrate to vault `system/decisions/` | permanent |
| research briefs | vault directly | yes | fetch writes directly | permanent |
| kishan profile/prefs | vault `personal/kishan/` | yes | diana updates when patterns change | permanent |
| system lessons | vault `system/` | yes | diana writes when lesson is durable | permanent |
| health baselines | vault `health/` (new) | yes | diana promotes from bryan logs | permanent |
| food/meal logs | `workspace/bryan/food-log/` | no (operational) | summarized weekly/monthly into vault health notes | kept indefinitely as raw data |
| progress pics | `workspace/bryan/progress-pics/` | no (operational) | referenced in vault health summaries | kept indefinitely as raw data |
| weight readings | `workspace/bryan/weight-log.md` | no (operational) | trend data promoted to vault health summaries | kept indefinitely as raw data |
| active threads | vault `system/active-threads.md` | yes | diana maintains | pruned when resolved |

---

## 4. retrieval contract

### diana (main agent)

**startup load (every session):**
1. `SOUL.md` — personality and operating rules
2. `IDENTITY.md` — who she is
3. `USER.md` — who kishan is
4. `memory/YYYY-MM-DD.md` — today + yesterday
5. `MEMORY.md` — long-term memory (until retired, then replaced by vault reads)

**on-demand:**
- `memory_search` for recall questions
- vault via VAULT_MAP.md → _index.md → specific notes
- `decisions/` — last 5 by date when relevant

**post-retirement of MEMORY.md:**
- replace step 5 with: vault `personal/kishan/preferences.md` + `system/active-threads.md`

### bryan (health agent)

**startup load (every session):**
1. `SOUL.md` — personality
2. `IDENTITY.md` — identity
3. `USER.md` — kishan context
4. `agent-spec.md` — photo handling modes, domain scope
5. `workspace/bryan/weight-log.md` — recent weight data (last 10 entries)
6. `workspace/bryan/food-log/log.md` — recent meal log (last 5 entries)
7. `workspace/bryan/progress-pics/log.md` — recent progress entries (last 3)

**on-demand:**
- vault `health/` for baselines, longitudinal summaries
- `memory_search` if recall question about past health context

### future agents

must define startup load in their agent spec before deployment. no agent ships without a retrieval contract.

---

## 5. promotion contract

### what gets promoted to vault

promotion happens during diana's periodic memory review (every few days or during heartbeat maintenance).

**promote when:**
- a lesson will be useful beyond this week
- a decision has downstream consequences
- a pattern about kishan is confirmed (not speculative)
- a project/mission fact is stable and reusable
- health data shows a meaningful trend (not single-day noise)

**do NOT promote:**
- raw heartbeat entries
- session-by-session play-by-play
- one-off troubleshooting steps that worked once
- speculative observations not yet confirmed
- every single preference tweak or micro-decision

### promotion targets

| source | destination | format |
|--------|------------|--------|
| daily log lesson | vault `system/lessons.md` or relevant note | append bullet |
| confirmed kishan pattern | vault `personal/kishan/preferences.md` or `bio.md` | update in place |
| decision record | vault `system/decisions/` (if significant) | copy or link |
| bryan weekly summary | vault `health/summaries/YYYY-WXX.md` | new note |
| project architecture insight | vault `missions/[project]/` | update relevant note |
| active thread change | vault `system/active-threads.md` | update in place |

---

## 6. integrity rules

### must detect:
- broken links in vault VAULT_MAP.md and _index.md files
- duplicate entries in maps/indexes
- vault notes not referenced in any index
- daily memory files with inconsistent naming
- uncommitted changes in vault or workspace repos
- agent startup files that reference nonexistent paths

### enforcement:
- phase 1: manual checks during memory review
- phase 2: selena builds validation script
- phase 3: scheduled integrity checks via cron

---

## 7. MEMORY.md lifecycle

**current state:** competing long-term store alongside vault
**target state:** retired

**migration path:**
1. audit current MEMORY.md content
2. promote durable facts to vault
3. move active threads to `system/active-threads.md`
4. reduce MEMORY.md to a pointer file or delete entirely
5. update AGENTS.md startup sequence to load vault notes instead

**timeline:** complete during phase 4-5 of refactor

---

## 8. vault structure (target)

```
vault/
├── VAULT_MAP.md
├── personal/
│   ├── _index.md
│   └── kishan/
│       ├── bio.md
│       └── preferences.md
├── system/
│   ├── _index.md
│   ├── memory-architecture.md    ← this file
│   ├── active-threads.md
│   ├── lessons.md
│   └── decisions/
│       └── [decision notes]
├── health/
│   ├── _index.md
│   ├── baselines.md
│   └── summaries/
│       └── [weekly/monthly summaries]
├── missions/
│   ├── _index.md
│   ├── general/
│   ├── polymarket/
│   └── [future missions]
├── tools/
│   ├── _index.md
│   └── [research notes]
└── workflows/
    ├── _index.md
    └── [research notes]
```

---

## 9. naming conventions

- vault notes: `kebab-case.md`
- daily logs: `YYYY-MM-DD.md`
- decision records: `YYYY-MM-DD-slug.md`
- health summaries: `YYYY-WXX.md` (weekly) or `YYYY-MM.md` (monthly)
- indexes: `_index.md` in every vault subdirectory
- vault map: `VAULT_MAP.md` at vault root

---

## 10. obsidian as human interface

the vault IS an obsidian vault. kishan opens it in obsidian to browse, search, audit, and review the entire digital brain.

### design rules for obsidian compatibility

1. **wikilinks preferred for internal links** — use `[[note-name]]` or `[[path/note-name|display text]]` over raw markdown links for cross-references within the vault. obsidian resolves these automatically.
2. **YAML frontmatter on every note** — obsidian uses frontmatter for metadata, search, and dataview queries. every vault note must have at minimum: `title`, `type`, `created`, `last_updated`.
3. **tags in frontmatter** — use `tags: [tag1, tag2]` in frontmatter for obsidian tag search. agents should add relevant tags when creating notes.
4. **flat-enough structure** — obsidian works best when folder nesting is shallow (2-3 levels max). current vault structure is good.
5. **no orphan notes** — every note should be reachable via VAULT_MAP.md, an _index.md, or a wikilink from another note. obsidian graph view makes orphans visible.
6. **graph-friendly linking** — when a note references another concept that has its own vault note, link it. this builds the knowledge graph organically.
7. **MOC pattern** — `_index.md` files function as Maps of Content (MOCs) in obsidian terminology. they are the entry points for each area.
8. **no binary blobs in vault** — photos, images, and binary files stay in agent-local operational storage (e.g., `workspace/bryan/`). vault is markdown and text only. summaries and references link out to where media lives.
9. **daily notes integration** — workspace `memory/YYYY-MM-DD.md` files are NOT in the vault, but obsidian can be configured to view them via a symlink or secondary vault if kishan wants a unified daily notes view.

### obsidian setup

- open `~/vault/` as an obsidian vault
- core plugins: graph view, search, backlinks, tags
- recommended community plugins: dataview (for querying frontmatter), templater (if kishan wants to create notes manually)
- `.obsidian/` directory is gitignored — obsidian config is local to each device, not version-controlled
