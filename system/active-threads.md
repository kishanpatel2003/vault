---
title: "Active Threads"
type: active_state
created: 2026-03-29
last_updated: 2026-03-29
---

# Active Threads

open items requiring follow-up, monitoring, or action. pruned when resolved.

---

## gateway install

kishan needs to run `openclaw gateway install` from Terminal on mac mini (GUI session required). cannot be done over SSH/headless.

**status:** waiting on kishan

---

## telegram bot token rotation

token was pasted in plaintext during initial setup. needs revoke + new token from @BotFather.

**status:** pending verification — confirm whether rotation was completed after initial setup

---

## fetch harness research

pinned task. script works, just needs to be re-scheduled.

**status:** pending re-scheduling

---

## openclaw auth/control layer for external repos

selena's review of `Yuqi2002/agentic-job-hunting` concluded that direct oauth retrofit is not a real path. if kishan wants external repos to use hosted models safely, route model calls through openclaw as the auth/control layer instead.

**status:** open — decide whether to build the proxy path or drop the retrofit attempt
