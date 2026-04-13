---
date: 2026-04-13
type: lesson
tags: [openclaw, auth, oauth, architecture]
---

# lesson: use openclaw as the auth/control layer, not direct oauth retrofits

## context
selena reviewed `Yuqi2002/agentic-job-hunting` and evaluated whether direct oauth retrofits were a viable way to add hosted model access.

## lesson
for external repos that need hosted model access, direct oauth retrofits are a dead-end path. the more durable pattern is to route model calls through openclaw, which centralizes auth, policy, and control.

## why it matters
- avoids repo-specific auth hacks that are brittle and hard to maintain
- keeps model access and controls in one place
- gives a cleaner integration path for future agentized projects

## apply when
- a repo needs access to hosted models
- the alternative is wiring ad hoc oauth into an existing codebase
- control, auditing, or policy enforcement matters
