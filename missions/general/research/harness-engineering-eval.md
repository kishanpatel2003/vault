---
title: "Eval — harness-engineering.md (v2, Apr 2026)"
type: eval
date: 2026-04-03
brief: "fetch/missions/harness-engineering.md"
prior_eval: "fetch/missions/harness-engineering-eval.md (v1, Mar 6 2026, scored 18/18)"
evaluator: fetch (self-assessment against feedback.md patterns)
---

## scores

| dimension | score (0–3) | notes |
|---|---|---|
| thesis_clarity | 3 | Summary sentence captures topic, method, progression (Nov 2025 → Mar 2026), and key convergence finding. No editorializing. |
| source_tension | 3 | Five distinct tensions: self-reported productivity claims, model-dependent context management strategies, underspecified evaluator calibration, narrow infrastructure noise methodology, persistent term ambiguity. All are sourced and specific. |
| grounding | 3 | Every finding cites source name + full URL. Single-sourced claims flagged inline. No "Source: Docs" entries. |
| completeness | 3 | Covers all six requested areas. Adds four new Anthropic primary sources not in the March brief. |
| summary_accuracy | 3 | One-sentence summary accurately reflects the March 2026 expansion (GAN architecture, eval taxonomy, infrastructure noise) and the sustained convergence finding. |
| filing_quality | 3 | Replaces prior brief in-place at same vault path. _index.md and VAULT_MAP updated. Commit pending. |

**estimated total: 18/18**

---

## what's new in this revision (relative to March 6, 2026 brief)

**Added sources (4 new):**
1. Anthropic, "Harness design for long-running application development," Mar 24, 2026 — GAN-inspired three-agent architecture (planner/generator/evaluator), context resets vs. compaction distinction, evaluator skepticism tuning
2. Anthropic, "Demystifying evals for AI agents," Jan 9, 2026 — Full eval taxonomy (task/trial/grader/transcript/outcome/evaluation harness/agent harness), grader type tradeoffs, capability vs. regression eval lifecycle
3. Anthropic, "Effective context engineering for AI agents," Sep 29, 2025 — Context as finite attention budget, system prompt altitude principle, tool set curation as context decision
4. Anthropic, "Quantifying infrastructure noise in agentic coding evals," ~Apr 2026 — 6pp resource configuration effect, dual guaranteed/kill-threshold recommendation

**Key conceptual additions:**
- Distinction between evaluation harness and agent harness (Demystifying Evals taxonomy)
- Context resets as distinct strategy from compaction, and model-dependency of the choice (Sonnet 4.5 needs resets; Opus 4.5 does not)
- Evaluator calibration as a harness design problem, not just a prompt problem
- Infrastructure noise as a new eval discipline with its own engineering requirements
- Context engineering as the named intermediate discipline between prompt engineering and harness engineering

---

## self-critique against feedback.md patterns

### "contextualize metrics. raw numbers without baselines are weak claims."
**applied:** OpenAI's 3.5 PRs/day and 1M LOC flagged as single-sourced and self-reported. Infrastructure noise 6pp finding is given context: "larger than the leaderboard gap between top models" and compared to SWE-bench crossover (1.54pp — smaller because tasks are less resource-intensive). The contrast between the two benchmarks is the contextualization.

### "thin bullets waste space."
**applied:** No finding bullet is under 2 sentences. New bullets (e.g., evaluator skepticism tuning, context resets vs. compaction) carry 3–5 sentences including mechanism, evidence, and boundary conditions.

### "every claim needs a verifiable source path."
**applied:** Every bullet: source name + full URL. Infrastructure noise finding includes statistical significance (p < 0.01). Evaluator calibration limitation is flagged as qualitative-only.

### "add a discrete sources list"
**applied:** 13 numbered entries with full URLs.

---

## watch items for future briefs

- **Evaluator calibration methodology is underspecified.** Anthropic describes few-shot calibration qualitatively. A future brief should search for any quantitative guidance published on evaluator reliability, inter-rater agreement, or calibration set sizing.
- **No Anthropic public eval registry.** OpenAI's openai/evals GitHub still has no Anthropic equivalent. Worth monitoring for a public release.
- **Context reset vs. compaction model matrix.** Currently only Sonnet 4.5 and Opus 4.5 are documented. As new model versions ship, this matrix will need updating.
- **Brownfield harness applicability.** No source has published evidence on applying these harness patterns to legacy codebases. If this emerges, it is a significant finding.
