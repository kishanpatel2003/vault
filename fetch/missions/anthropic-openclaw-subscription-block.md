---
title: "Anthropic blocking subscription usage on OpenClaw"
type: research_brief
date: 2026-03-09
requested_by: kishan
tags: [anthropic, openclaw, claude, subscription, oauth, third-party-tools, policy, enforcement]
mission: null
sources_count: 8
summary: "Anthropic blocked Claude subscription OAuth tokens from working in third-party tools including OpenClaw in January 2026, then formalized this as official policy in February 2026; accounts are not being canceled, but the practice now explicitly violates Consumer Terms of Service."
---

## background

OpenClaw is an agentic platform (using the Pi harness) that allows users to run AI agents. Before January 2026, some users configured OpenClaw to route requests through Claude Pro or Max subscription credentials (OAuth tokens) rather than paid API keys — a practice also common across other third-party harnesses like OpenCode, Roo Code, Cline, and Cursor. This created a significant arbitrage: users on a $200/month Max subscription could consume token volumes that would cost $1,000+ via the pay-per-use API. The practice accelerated in late 2025 as developers popularized "Ralph Wiggum"-style autonomous overnight loops that burned tokens at scale without Anthropic's rate-limiting telemetry.

Research scope: Anthropic official documentation, The Register, The New Stack, openclaw.rocks official blog, Hacker News, Augmented Mind Substack, Reddit r/ClaudeAI and r/Anthropic, Gigazine.

## key findings

- **Technical block deployed January 9, 2026 (no advance notice):** Anthropic server-side blocked Claude subscription OAuth tokens from authenticating outside official Anthropic apps. Third-party tools began receiving the error: *"This credential is only authorized for use with Claude Code and cannot be used for other API requests."* Source: openclaw.rocks, https://openclaw.rocks/blog/anthropic-oauth-ban

- **Tools affected:** OpenCode, Roo Code, Cline, Cursor, and OpenClaw users routing via subscription OAuth. API-key users and OpenRouter integrations were not affected. Source: openclaw.rocks, https://openclaw.rocks/blog/anthropic-oauth-ban

- **Some accounts were incorrectly auto-banned:** Anthropic engineer Thariq Shihipar acknowledged that some accounts were "automatically banned for triggering abuse filters" in error, and those bans were reversed. Source: The Register, https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/

- **Official policy formalized February 19, 2026:** Anthropic updated its Claude Code legal and compliance documentation to explicitly state: *"Using OAuth tokens obtained through Claude Free, Pro, or Max accounts in any other product, tool, or service — including the Agent SDK — is not permitted and constitutes a violation of the Consumer Terms of Service."* Source: Anthropic official docs, https://code.claude.com/docs/en/legal-and-compliance

- **Policy has roots in ToS since at least February 2024:** Section 3.7 of the Consumer Terms of Service has prohibited automated access via non-Anthropic tools since at least February 2024. The February 2026 update added explicit OAuth-specific language rather than creating a new rule. Source: The Register, https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/

- **Anthropic is not canceling accounts for past usage:** Anthropic's official statement to The New Stack: *"Nothing changes around how customers have been using their account and Anthropic will not be canceling accounts."* The block is forward-looking enforcement, not retroactive punishment. Source: The New Stack, https://thenewstack.io/anthropic-agent-sdk-confusion/; OpenClaw blog, https://openclaw.rocks/blog/anthropic-oauth-ban

- **OpenClaw's official response — pivot to OpenAI Codex:** OpenClaw's founder confirmed on record that "Anthropic made it official, you cannot use a Claude subscription to power OpenClaw anymore," and redirected users to OpenAI Codex subscriptions as the new default. Source: Augmented Mind Substack, https://augmentedmind.substack.com/p/the-end-of-the-claude-subscription-hack

- **OpenCode removed Claude subscription support following legal request:** Third-party harness OpenCode (107k+ GitHub stars) pushed a commit on or around February 20, 2026 removing Claude Pro/Max account key support, citing "anthropic legal requests." Source: The Register (citing GitHub commit), https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/

- **OpenAI capitalized on Anthropic's move:** OpenAI's Thibault Sottiaux publicly endorsed the use of Codex subscriptions in third-party harnesses in direct response to Anthropic's ban. Source: The Register (citing X/Twitter post), https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/

- **Anthropic's stated rationale:** Anthropic engineer Shihipar cited two reasons: (1) economic — subscription arbitrage was unprofitable at scale; (2) technical — third-party tools don't send the telemetry Anthropic uses for rate limiting, debugging, and safety monitoring, creating blind spots for abuse detection. Source: The Register, https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/

## source tensions

- **Severity framing varies significantly.** Technical sources (The Register, openclaw.rocks) treat this as a definitive ToS enforcement with clear precedent in existing policy. Community sources (Reddit, HN) frame it as an unexpected, hostile crackdown. Both framings are accurate in their respective dimensions.

- **"Blocking people on OpenClaw" is partially accurate but imprecise.** Anthropic is not blocking OpenClaw users from using Claude — users with API keys are unaffected. What Anthropic blocked is using *subscription OAuth tokens* to route OpenClaw through. The distinction matters: API-key OpenClaw users experience no change.

- **Whether accounts are truly "safe" is contested.** Anthropic's official statement says accounts will not be canceled for past usage. However, Anthropic's docs explicitly state it "reserves the right to take measures to enforce these restrictions and may do so without prior notice." Community sources note ongoing uncertainty. The current enforcement posture appears to be block-not-ban, but this is not guaranteed.

- **"OpenClaw blocked" vs. "Claude subscription blocked in OpenClaw."** The correct frame is the latter. OpenClaw as a platform continues to operate; it now routes users to OpenAI Codex or API keys instead. Anthropic did not take direct action against OpenClaw's servers or infrastructure.

- Sources are uniformly consistent on the core factual sequence: technical block January 9, policy documentation February 19, no retroactive account cancellations as of filing date.

## related vault nodes

- `missions/general/research/harness-engineering.md` — topical overlap: harness engineering practices and how third-party harnesses interact with model providers; directly relevant context for why this dispute emerged.
- `missions/general/research/claude-code-auto-mode.md` — topical overlap: Anthropic's Claude Code product and its ongoing role as Anthropic's preferred harness for Claude model access.
