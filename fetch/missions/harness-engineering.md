---
title: "Harness Engineering"
type: research_brief
date: 2026-03-06
requested_by: kishan
tags: [harness-engineering, llm-agents, prompt-engineering, evaluation, anthropic, openai, context-engineering, agent-sdk]
mission: general
sources_count: 9
summary: "Harness engineering is the discipline of designing the software infrastructure surrounding an LLM — tools, memory, context management, feedback loops, and architectural constraints — as distinct from prompt engineering, which addresses single-turn instruction quality; both Anthropic and OpenAI published canonical treatments in late 2025, converging on shared principles while differing in emphasis."
---

## background

The term "harness engineering" crystallized as a field in the second half of 2025. Mitchell Hashimoto used "engineer the harness" as a discrete step in his AI adoption framework (Step 5 of 6) in late 2025, distinguishing the work of building scaffolding around agents from the work of writing prompts. OpenAI then published a piece titled "Harness engineering: leveraging Codex in an agent-first world" in early 2026, documenting a 5-month experiment building a million-line codebase with zero manually-written code. Anthropic published "Effective harnesses for long-running agents" on November 26, 2025. Martin Fowler's analysis of the OpenAI piece on martinfowler.com brought the concept further into practitioner discourse.

This research synthesizes primary sources from Anthropic and OpenAI — official engineering blogs, SDK documentation, the openai/evals GitHub repository, and the OpenAI platform evals API guide — plus practitioner analysis from Fowler, Hashimoto, Parallel.ai, and Analytics Vidhya. Scope covers: definition and conceptual framing, contrast with prompt engineering, best practices from each company, tooling patterns, evaluation frameworks, and the field's direction.

---

## key findings

### Definition

- An agent harness is the software infrastructure surrounding an LLM that manages everything except the model itself: context lifecycle, tool execution, memory, feedback loops, and architectural constraints. Source: Parallel.ai, "What is an agent harness?", https://parallel.ai/articles/what-is-an-agent-harness
- One definition from an AI architect quoted by Parallel.ai: "the complete architectural system surrounding an LLM that manages the lifecycle of context: from intent capture through specification, compilation, execution, verification, and persistence." Source: Parallel.ai, https://parallel.ai/articles/what-is-an-agent-harness (citing Anthony Alcaraz / LinkedIn)
- The Anthropic Claude Agent SDK documentation explicitly refers to itself as "a powerful, general-purpose agent harness adept at coding, as well as other tasks that require the model to use tools to gather context, plan, and execute." Source: Anthropic engineering blog, "Effective harnesses for long-running agents," Nov 26, 2025, https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Analytics Vidhya (Dec 2025) frames the field as a tri-layer architecture: agent framework (library/SDK), agent runtime (execution environment), and agent harness (deployment wrapper). Source: Analytics Vidhya, "Agent Frameworks vs Runtime vs Harnesses," Dec 8, 2025, https://www.analyticsvidhya.com/blog/2025/12/agent-frameworks-vs-runtimes-vs-harnesses/

### How Harness Engineering Differs from Prompt Engineering

- Prompt engineering focuses on the content and structure of a single instruction passed to a model. Harness engineering is the surrounding system: it determines what context reaches the model, how tools are surfaced, how memory persists across sessions, and how outputs are verified. Parallel.ai states: "An AI harness includes prompt engineering as one of its duties (deciding what to feed the model), but goes much further — it manages tools, memory, and the whole loop of interactions." Source: Parallel.ai, https://parallel.ai/articles/what-is-an-agent-harness
- The OpenAI harness engineering piece explicitly reframes the engineer's job: "what changes when a software engineering team's primary job is no longer to write code, but to design environments, specify intent, and build feedback loops." Source: OpenAI, "Harness engineering: leveraging Codex in an agent-first world," https://openai.com/index/harness-engineering/
- Martin Fowler summarizes: harness engineering trades "generate anything" flexibility for reliability — "constraining the solution space: specific architectural patterns, enforced boundaries, standardized structures." Source: Martin Fowler, "Harness Engineering," martinfowler.com, https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
- Mitchell Hashimoto's practitioner framing: "If you give an agent a way to verify its work, it more often than not fixes its own mistakes and prevents regressions." Verification infrastructure is harness work; writing the instruction to verify is prompt work. Source: Mitchell Hashimoto, "My AI Adoption Journey," https://mitchellh.com/writing/my-ai-adoption-journey#step-5-engineer-the-harness

### Anthropic Best Practices

- **Two-agent harness structure for long-running tasks.** Anthropic recommends a split between an *initializer agent* (first session only: sets up init.sh, claude-progress.txt, initial git commit) and a *coding agent* (all subsequent sessions: incremental progress + clean state at end). The two agents differ only in their initial user prompt; system prompt and tools are identical. Source: Anthropic, "Effective harnesses for long-running agents," Nov 26, 2025, https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Feature list as a structured artifact.** The initializer agent writes a JSON file of 200+ granular end-to-end feature descriptions, all initially marked `"passes": false`. Coding agents may only toggle the `passes` field; deleting or editing features is "unacceptable." JSON is preferred over Markdown because models are less likely to overwrite JSON files inappropriately. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Session startup sequence ("getting your bearings").** Each coding agent begins by running `pwd`, reading `claude-progress.txt` and the feature list, checking `git log --oneline -20`, running `init.sh` to start the dev server, and verifying basic functionality end-to-end before starting new work. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Incremental progress + git as state persistence.** Agents commit to git with descriptive messages after each feature. Progress files serve as readable handoffs between sessions; git history enables rollback of bad states. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Browser automation for verification.** Absent explicit prompting, Claude marks features complete without proper end-to-end testing. Providing Puppeteer MCP access and requiring human-style verification dramatically improved pass accuracy. Known limitation: Claude cannot see browser-native alert modals through Puppeteer. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Context engineering via file system.** The Claude Agent SDK documentation treats folder and file structure as a form of context engineering: Claude can use `grep`, `tail`, and similar bash tools to selectively load relevant context. Semantic search is available but Anthropic recommends starting with agentic file search and adding semantic search only when speed is required. Source: Anthropic, "Building agents with the Claude Agent SDK," Sep 29, 2025, https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Subagents for parallelization and context isolation.** The Claude Agent SDK supports subagents by default. Subagents receive isolated context windows and return only relevant excerpts to the orchestrator, reducing noise and enabling concurrent task execution. Source: Anthropic, "Building agents with the Claude Agent SDK," https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Compaction for long context.** The SDK's compact feature automatically summarizes prior messages when the context window approaches its limit. Described as "context management" rather than a workaround; designed to be invisible to the agent's functional behavior. Source: Anthropic, "Building agents with the Claude Agent SDK," https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Four canonical failure modes.** Anthropic documents: (1) agent declares victory too early, (2) agent leaves environment in broken/undocumented state, (3) agent marks features done without testing, (4) agent wastes context figuring out how to run the app. Each has a corresponding harness countermeasure (feature list, progress file + git, browser automation, init.sh script). Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Multi-context window prompting guidance.** Anthropic's Claude 4 prompting guide includes a dedicated section on multi-context window workflows, recommending a different initial prompt for the first context window. Source: Anthropic, "Prompting best practices," https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

### OpenAI Best Practices

- **"No manually-written code" as a forcing function.** OpenAI's harness engineering team used an absolute constraint (zero human-typed code) to discover what scaffolding agents actually require. "When something failed, the fix was almost never 'try harder.' … human engineers always stepped into the task and asked: 'what capability is missing, and how do we make it both legible and enforceable for the agent?'" Source: OpenAI, "Harness engineering: leveraging Codex in an agent-first world," https://openai.com/index/harness-engineering/
- **AGENTS.md as table of contents, not encyclopedia.** Monolithic instruction files fail: they crowd out task context, become stale, are hard to verify, and cause agents to pattern-match locally. OpenAI's solution: AGENTS.md is ~100 lines pointing to a structured `docs/` directory. Each area of the docs is indexed, versioned, and has a defined owner. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Knowledge base as repository-local, versioned artifact.** "From the agent's point of view, anything it can't access in-context while running effectively doesn't exist." Design decisions in Slack threads or Google Docs are illegible to the agent; they must be captured in markdown/schemas/plans committed to the repo. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Architectural constraints enforced deterministically.** OpenAI requires agents to "parse data shapes at the boundary" but does not specify how. Custom linters and CI jobs validate the knowledge base structure and cross-links. A recurring "doc-gardening" agent scans for stale documentation and opens fix-up PRs. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Observability as agent-legible context.** Logs (via LogQL), metrics (via PromQL), and traces are exposed to Codex via an ephemeral per-worktree observability stack. This makes prompts like "ensure service startup completes in under 800ms" tractable. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Per-worktree isolated app instances.** Each Codex task runs against a fully isolated app instance (including its own logs/metrics), torn down after task completion. Browser automation via Chrome DevTools Protocol enables UI-level verification. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Agent-to-agent code review.** Codex is instructed to review its own changes, request additional agent reviews, respond to feedback, and loop until all agent reviewers are satisfied. Human code review is optional. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Execution plans as first-class artifacts.** Complex tasks get formal "execution plans" with progress logs and decision logs checked into the repo. Smaller tasks use lightweight ephemeral plans. Known technical debt is versioned alongside active work. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Productivity claim: 1/10th time, 3.5 PRs/engineer/day.** OpenAI reports the team built ~1M LOC across 1,500 PRs in 5 months with 3–7 engineers, estimated at 1/10th the time of manual coding. Throughput increased as team size grew from 3 to 7. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/ — single-sourced, self-reported.

### Tooling Patterns

- **Claude Agent SDK** (Anthropic): General-purpose harness with compaction, subagent support, tool use, context management. Renamed from Claude Code SDK in September 2025. Source: Anthropic, "Building agents with the Claude Agent SDK," https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Codex CLI** (OpenAI): Agent runtime for code-generation tasks; integrates with GitHub workflow (gh CLI, pull requests, CI). Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **openai/evals** (OpenAI, open-source): Framework for evaluating LLMs and LLM systems; open-source registry of benchmarks; supports basic, model-graded, and custom eval templates. Accepts YAML eval definitions and JSON test data. Source: GitHub, https://github.com/openai/evals
- **OpenAI Evals API**: Programmatic eval configuration via `data_source_config` (test data schema) and `testing_criteria` (graders: string_check, model-graded, etc.). Runs are asynchronous; results delivered via webhook or polling. Source: OpenAI platform docs, https://platform.openai.com/docs/guides/evals
- **Puppeteer MCP**: Model Context Protocol server for browser automation; used by Anthropic's Claude to perform end-to-end UI verification. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **CLAUDE.md / AGENTS.md**: Instruction files read at agent session start. Claude Code uses `CLAUDE.md`; OpenAI Codex uses `AGENTS.md`. Both serve as entry-point context, but OpenAI's guidance limits them to ~100 lines (table of contents only). Source: Anthropic Claude Code docs, https://code.claude.com/docs; OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **MCP (Model Context Protocol)**: Open standard for connecting agents to external data sources (Google Drive, Jira, Slack, databases, custom tooling). Used in Claude Agent SDK. Source: Anthropic, "Building agents with the Claude Agent SDK," https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Git + progress files**: Common across both companies. Used for state persistence, session handoffs, and rollback. OpenAI additionally uses execution plans as committed documents.

### Evaluation Frameworks

- **OpenAI Evals (open-source)**: Registry of ~100+ benchmark evals covering reasoning, instruction-following, coding, factuality, and function-calling. Evaluators write YAML + JSON; no custom code accepted for public contributions. Supports string checks and model-graded evals. Source: GitHub, openai/evals, https://github.com/openai/evals
- **OpenAI Evals API (platform)**: Three-step flow: (1) define eval with data schema and graders, (2) upload test data as JSONL, (3) run eval with a prompt template against a model. Graders include `string_check` (exact match) and model-graded rubrics. Results available via report URL or webhook. Source: OpenAI platform docs, https://developers.openai.com/api/docs/guides/evals
- **Anthropic's eval guidance**: "If your agent's performance varies as you add features, build a representative test set for programmatic evaluations (or evals) based on customer usage." No public Anthropic-maintained eval registry as of this research. Source: Anthropic, "Building agents with the Claude Agent SDK," https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- **Feature list JSON as inline eval**: Anthropic's harness pattern treats the feature list file as a live evaluation surface — features are marked `passes: true/false` only after verified end-to-end testing. This embeds a lightweight regression harness directly into the agent loop. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Model-graded evals**: Both companies support using an LLM as a judge to evaluate outputs against a rubric. OpenAI's evals framework supports this natively; Anthropic recommends it for complex output assessment. Single-sourced on Anthropic side. Source: OpenAI evals framework README, https://github.com/openai/evals; Anthropic SDK docs
- **Deterministic structural tests**: OpenAI's harness uses custom linters and CI jobs to enforce architectural constraints. These are distinct from model evals — purely rule-based, no LLM involved. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/

### Where the Field Is Heading

- **Multi-agent specialization**: Anthropic explicitly frames it as an open question whether a single general-purpose coding agent outperforms a team of specialized agents (testing agent, QA agent, code cleanup agent). Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- **Harnesses as the new service templates**: Fowler speculates that teams will pick from harness templates for common application topologies, much as they use service templates today. Open questions about how harnesses evolve and how teams incorporate upstream updates without forking. Source: Martin Fowler, "Harness Engineering," https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
- **Convergence on constrained tech stacks**: OpenAI's team chose "boring" technology (well-established frameworks with high training-set representation) because it is easier for agents to reason about. Fowler predicts developers will increasingly choose stacks with good harnesses available rather than preferred developer ergonomics. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/; Martin Fowler, ibid.
- **Generalization beyond software development**: Anthropic identifies "scientific research or financial modeling" as future domains for harness patterns. No concrete examples yet — noted as future work. Source: Anthropic, "Effective harnesses for long-running agents," https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents — single-sourced.
- **Human attention as the scarce resource**: Both companies converge on the same design constraint: the bottleneck is human time and attention, not agent capability. OpenAI: "our one truly scarce resource: human time and attention." Harness design is increasingly about minimizing the surface area requiring human intervention. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/
- **Agent-to-agent review replacing human review**: OpenAI reports pushing "almost all review effort towards being handled agent-to-agent." Humans may review PRs but are not required to. Source: OpenAI, "Harness engineering," https://openai.com/index/harness-engineering/ — single-sourced.

---

## source tensions

**Productivity claims are single-sourced and self-reported.** OpenAI's "1/10th the time" and "3.5 PRs/engineer/day" metrics come from the OpenAI team writing about an OpenAI tool (Codex). No independent replication or third-party audit. Fowler notes this explicitly: "OpenAI do have a vested interest in us believing in AI-maintainable code." Additionally, the codebase was greenfield — no pre-existing human code to contend with. Applicability to brownfield codebases is entirely unverified.

**Anthropic's harness optimized for web app development only.** The November 2025 blog post explicitly states the demo is "optimized for full-stack web app development" and flags generalization as future work. The feature list, Puppeteer verification, and init.sh patterns may not transfer cleanly to, e.g., data pipelines or scientific computing.

**Fowler identifies a gap in the OpenAI piece: no functional verification.** OpenAI's harness addresses code maintainability, context legibility, and architectural consistency. Fowler notes it "is missing verification of functionality and behaviour" — the harness does not confirm the product does what users need, only that the code structure is sound.

**Term definitions remain fluid.** "Harness engineering" appears differently across sources: Parallel.ai treats it as an architectural system; Analytics Vidhya treats it as one layer in a tri-layer stack (framework/runtime/harness); OpenAI uses it as a practice label for their experiment. There is no settled standard definition. The term is approximately 4–6 months old as of this research.

**AGENTS.md vs CLAUDE.md divergence.** OpenAI and Anthropic use different instruction file conventions (`AGENTS.md` vs `CLAUDE.md`), with different size recommendations (OpenAI caps at ~100 lines; Anthropic's guidance is less prescriptive). Cross-company harnesses working with both models would need to reconcile these.

**Sources were uniformly consistent on one point**: the model (GPT-4, Claude Opus, etc.) is increasingly not the determining factor in agent quality. Multiple independent sources — Fowler, Parallel.ai, Hashimoto, the Medium piece on "2026 is agent harnesses" — converge on the claim that "the harness determines whether agents succeed or fail."

---

## related vault nodes

- `tools/python/scrapling.md` — topical overlap with tool use in agentic systems; scrapling addresses one class of tool (web scraping) that might be integrated into an agent harness.

---

## sources

1. Anthropic, "Effective harnesses for long-running agents," Nov 26, 2025 — https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
2. Anthropic, "Building agents with the Claude Agent SDK," Sep 29, 2025 — https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
3. Anthropic, "Prompting best practices (Claude 4)," 2025 — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
4. OpenAI, "Harness engineering: leveraging Codex in an agent-first world," early 2026 — https://openai.com/index/harness-engineering/
5. OpenAI, "Working with evals," platform docs — https://platform.openai.com/docs/guides/evals
6. OpenAI, openai/evals GitHub repository — https://github.com/openai/evals
7. Martin Fowler, "Harness Engineering," martinfowler.com — https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
8. Parallel.ai, "What is an agent harness?" — https://parallel.ai/articles/what-is-an-agent-harness
9. Mitchell Hashimoto, "My AI Adoption Journey," Step 5: Engineer the Harness — https://mitchellh.com/writing/my-ai-adoption-journey#step-5-engineer-the-harness
