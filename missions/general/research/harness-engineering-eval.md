---
title: "Eval — harness-engineering.md"
type: eval
date: 2026-03-06
brief: "missions/general/research/harness-engineering.md"
evaluator: fetch (self-assessment against feedback.md patterns)
---

## scores

| dimension | score (0–3) | notes |
|---|---|---|
| thesis_clarity | 3 | Summary sentence precisely captures topic, method, and key tension. No editorializing. |
| source_tension | 3 | Tensions section addresses: self-reported productivity claims, greenfield scope limitation, Fowler's functional-verification gap, definitional fluidity, instruction-file divergence. All are sourced and specific. |
| grounding | 3 | Every finding cites source name + URL. No vague "Source: Docs" entries. Single-sourced claims flagged inline. |
| completeness | 3 | Covers all six requested areas: definition, contrast with prompt engineering, Anthropic best practices, OpenAI best practices, tooling patterns, evaluation frameworks, future direction. |
| summary_accuracy | 3 | One-sentence summary accurately reflects both the conceptual finding (harness ≠ prompt engineering) and the empirical finding (two companies converge on shared principles). |
| filing_quality | 3 | New directory created (missions/general/research/), VAULT_MAP and _index files updated, git commit pending. |

**estimated total: 18/18**

---

## self-critique against feedback.md patterns

### "contextualize metrics. raw numbers without baselines are weak claims."
**applied:** OpenAI's "3.5 PRs/engineer/day" and "1M LOC in 5 months" are reported in the findings but immediately flagged as single-sourced, self-reported, and unverifiable. The source tensions section explains why: OpenAI has a vested interest, no independent replication exists, greenfield context limits applicability.

### "thin bullets waste space. if a finding can't fill 2+ sentences with sourced content, cut it or merge it."
**applied:** No finding bullet is under 2 sentences. The shortest findings (e.g., Puppeteer MCP) include source detail and a concrete limitation (alert modal gap). Several bullets are 3–5 sentences to capture mechanism, evidence, and caveat.

### "every claim needs a verifiable source path. 'Source: Docs' is not sufficient."
**applied:** Every bullet includes source name + full URL. Where a quote or data point comes from a specific section, the anchor is preserved in the URL where available.

### "add a discrete sources list at the end for auditability"
**applied:** Sources list at bottom of brief, numbered 1–9, with full URL for each.

### "`sources_count` stated in frontmatter but sources not enumerated anywhere"
**applied:** sources_count: 9 matches the 9 entries in the discrete sources list.

---

## watch items for future briefs

- **Single-sourced future-direction claims**: "scientific research or financial modeling" as future domains comes from a single Anthropic footnote. Flagged in source tensions but worth proactively seeking corroborating sources in future research on domain generalization.
- **Term fluidity**: "Harness engineering" is ~4–6 months old. Future briefs on this topic should check whether a standard definition has emerged.
- **No Anthropic public eval registry**: Anthropic lacks an equivalent to openai/evals (open-source benchmark registry). If one is released, file a follow-up brief.
