---
date: 2026-03-04
status: decided
reversibility: type-2
review_date: 2026-04-04
---

# Decision: Benchmarking too slow; use Anthropic's faster model

## Context
Ran benchmarks on local ollama setup yesterday. Process kept hitting OOM (young-ha sandbox got SIGKILL'd). Turnaround was multiple minutes per iteration. That's a bottleneck for iteration speed on agent/system building.

## Alternatives Considered
- Scale up sandbox resources (expensive, doesn't solve the fundamental latency issue)
- Stick with ollama (free, reliable, but too slow)
- Switch to Anthropic API (fast, costs money, but unblocks iteration)

## Decision
Use Anthropic Claude (Haiku for speed, Sonnet for quality) for benchmarking and critical-path work. Keep local ollama for experimentation and learning only.

## Assumptions
- Anthropic API costs are acceptable at current iteration volume
- Faster turnaround (seconds vs. minutes) justifies cost vs. local compute
- Won't hit rate limits on critical tasks

## Downstream
- Affects tool selection for benchmark framework (will use API, not local)
- Cascades to cost-tracking if we track agent spend
- May shift strategy on when to use local vs. remote models

## Notes
This decision is reversible. If costs climb or we build better local infra, we can pivot back.
