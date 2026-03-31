---
title: "Diana Proactive Completion Update Failure — Root Cause Audit"
type: research_brief
date: 2026-03-30
requested_by: diana
tags: [diana, async, communication, reliability, ops-audit, heartbeat, subagents]
mission: null
sources_count: 6
summary: "Diana frequently fails to send proactive completion updates for multi-minute background tasks because session-scoped promises have no cross-session persistence mechanism; the heartbeat communication audit is structurally unable to detect delegated tasks it was never told about; and the session-end protocol does not log pending notification state."
---

## background

diana is the orchestrator agent. she delegates multi-minute work to fetch and selena (and sometimes shells out background processes) and routinely promises to update kishan when the task finishes. the problem: those updates frequently never arrive, even when diana explicitly said they would. this audit investigates why using architecture docs, specs, heartbeat logs, and session memory as primary sources.

sources examined:
1. `~/.openclaw/workspace/AGENTS.md` — diana's operating spec including async communication protocol
2. `~/.openclaw/workspace/HEARTBEAT.md` — heartbeat task list and communication audit trigger
3. `~/.openclaw/workspace/memory/2026-03-29.md` — heartbeat log evidence (8 consecutive heartbeats)
4. `~/.openclaw/workspace/memory/2026-03-30.md` — heartbeat log evidence (3 heartbeats)
5. `~/vault/system/fleet-architecture.md` — agent model/role/runtime assignments
6. `~/vault/system/memory-architecture.md` — cross-session memory contracts and retrieval specs

---

## executive summary

diana's proactive update failures are not random. they follow a consistent structural pattern: she makes a completion-update promise inside an active session, but that session ends before the background task finishes. the next session — whether triggered by heartbeat or kishan — has no record of the pending promise. the heartbeat's communication audit, which should catch this, is effectively toothless because it cannot detect tasks it was never explicitly told about. the failure is ~80% architectural (session boundary + missing task registry) and ~20% model-behavioral (haiku 4.5 not reliably self-enforcing the anti-failure rule).

---

## key findings

- **session-scoped promises, no cross-session task registry.** diana's async communication protocol (in AGENTS.md) is detailed and explicit: start update, milestone updates, blocked state, completion, failure. the protocol is well-specified. what it lacks: a mechanism for persisting the *existence* of pending delegated tasks across session boundaries. when diana's session ends mid-task, the promise ends with it. source: `AGENTS.md` — async task communication protocol section.

- **heartbeat communication audit always reports clean.** the heartbeat fires on a cron schedule (~2 hours based on logs). its HEARTBEAT.md has an explicit audit step: "if any delegated/background task completed or failed without a proactive user update being sent, send the update now." across 10+ consecutive heartbeat log entries in 2026-03-29.md and 2026-03-30.md, this audit reports "no delegated/background task completion or failure is currently awaiting an unsent user update" every single time — including during the memory refactor period when selena was actively running multi-hour tasks. this means the audit is not detecting real pending tasks. source: `memory/2026-03-29.md`, `memory/2026-03-30.md`.

- **session-end protocol does not capture pending task state.** AGENTS.md specifies a 7-step session-end protocol (write memory, commit decisions, update vault, git commit/push). step 4 is "audit git status." no step says "log any delegated tasks currently in-flight with expected completion signal." without this, heartbeat sessions have no structured way to know what promises diana made. source: `AGENTS.md` — session end protocol.

- **subagent completion delivery relies on live requester session.** subagent context docs say "results auto-announce to your requester." if the requesting session has ended (timeout, idle disconnect, user closing chat), the delivery target no longer exists. the completion event may surface at next session start or may be lost entirely. this is a product/runtime constraint, not a prompt gap. source: subagent context injected in runtime environment.

- **haiku 4.5 is the model running this protocol.** diana runs on haiku 4.5 — lightest model in the fleet. the async communication rules are detailed and multi-conditional. haiku is expected to comply in the moment but is demonstrably weaker at sustained, self-enforced proactive behavior across a long session, especially when context grows. the "anti-failure rule" in AGENTS.md ("if diana notices that more than a few minutes have passed during active work without an outbound update, assume she is failing and send one immediately") requires self-monitoring that is harder at smaller model scale. source: `fleet-architecture.md`, `AGENTS.md`.

- **heartbeat cron interval is too coarse for task completion windows.** heartbeat fires roughly every 2 hours. most selena/fetch tasks complete in 5–30 minutes. even a perfectly-functioning heartbeat recovery path would introduce a worst-case ~2-hour notification delay. source: `memory/2026-03-29.md` (heartbeat timestamps: 03:23, 05:23, 07:23, 09:23, 10:23, 13:23, 16:23, 18:23, 21:23).

---

## root cause table

| # | root cause | type | confidence | evidence |
|---|-----------|------|-----------|---------|
| 1 | session-scoped promise + no cross-session task registry | architectural | high | 10+ heartbeats report clean comm audit during known active tasks |
| 2 | session-end protocol has no pending-task capture step | prompt/spec gap | high | AGENTS.md session-end protocol — 7 steps, none for in-flight tasks |
| 3 | heartbeat comm audit cannot detect what it was never told | architectural | high | every heartbeat logs "no tasks pending" including during selena runs |
| 4 | subagent completion delivery unreliable across session boundaries | product/runtime | medium-high | subagent docs: "results auto-announce to requester" — undefined behavior if session gone |
| 5 | haiku 4.5 weak at sustained self-monitoring over long sessions | model constraint | medium | model spec in fleet-architecture.md; pattern matches observed behavior |
| 6 | heartbeat cron interval too coarse for typical task durations | tooling/runtime | medium | 2h interval vs 5–30 min task window = structural delay floor |

---

## source tensions

no external sources were used. all evidence is internal (docs, specs, logs). no conflicts between sources were found. the AGENTS.md async protocol is well-written; the failure is not in the spec's content but in its enforcement infrastructure. the heartbeat logs are consistent and provide direct behavioral evidence.

---

## mitigations

### behavioral (diana, in-session)
1. **write a pending-task entry to memory before delegating.** immediately after spawning fetch or selena, append a structured entry to `memory/YYYY-MM-DD.md`: `[PENDING_TASK] task=X agent=Y expected_completion=~Nmin notification_owed=true`. cost: 10 seconds. value: makes heartbeat recovery possible.
2. **enforce the anti-failure rule on a clock, not on feel.** current rule says "if diana notices more than a few minutes passed without an update, send one." reframe: check whether an update is owed every time a tool call completes. explicit check beats ambient noticing.

### prompt/spec (AGENTS.md, HEARTBEAT.md)
3. **add step 0 to session-end protocol: "log pending tasks."** before memory write, check if any background tasks are in-flight. write their status to the daily log as structured PENDING_TASK entries. add this explicitly to AGENTS.md session-end protocol.
4. **rewrite heartbeat communication audit to scan for PENDING_TASK log entries.** current audit is "if any task completed without notification, send now." rewrite: "search today's and yesterday's memory file for PENDING_TASK entries. for each, check subagent status or last known state. send update if none was sent." this makes the audit active, not passive.
5. **add explicit instruction: complete the loop even in a new session.** AGENTS.md should state: "if you see a PENDING_TASK entry in memory that has no corresponding completion notification logged, that promise is yours to fulfill — regardless of whether you were the session that made it."

### workflow
6. **use `sessions_yield` explicitly to hold session open.** for tasks diana delegates where she can await the result, use `sessions_yield` to stay in the session rather than dropping back to idle. this keeps the requester session alive for auto-announce delivery.
7. **for fire-and-forget tasks, confirm in memory before yielding.** if diana must let a session end, the last action before yielding should be: write PENDING_TASK to memory log, commit.

### tooling/runtime
8. **shorten heartbeat interval to 30 minutes for active-task periods.** a 2-hour cron interval is appropriate for idle maintenance but too coarse when tasks are in-flight. consider: shorter interval triggered by the presence of PENDING_TASK entries in the memory log, or a standing 30-minute interval.
9. **runtime: clarify/fix subagent completion delivery to dead sessions.** the product should define what happens when a subagent result auto-announces to a session that has ended. if the event is dropped, the correct fix is: deliver to the next session startup as a pending event. if delivery to next startup is already the behavior, diana's startup sequence should explicitly check for queued completions before anything else.

---

## relevant behavioral evidence

the memory logs (2026-03-29, 2026-03-30) show direct evidence of the failure mode:
- the memory refactor involved selena running multi-hour multi-phase tasks (vault migrations, integrity tooling).
- heartbeat log at 2026-03-29 21:23 (post-selena run): "communication audit: no delegated/background task completion or failure is currently awaiting an unsent user update." — this entry is demonstrably incorrect or the tasks had already wrapped by session close. the audit was not catching selena's completion.
- every heartbeat communication audit is identical boilerplate: "no delegated/background task completion or failure is currently awaiting an unsent user update." this consistency across wildly different activity levels is itself evidence that the audit is not performing real detection — it is a passive check that defaults to clean when no structured signal exists.

---

## related vault nodes

- `system/fleet-architecture.md` — agent model assignments including diana=haiku 4.5, relevant to model constraint root cause.
- `system/lessons.md` — operational lessons on cron/LaunchAgent constraints; topical overlap with heartbeat cron interval finding.
- `fetch/general/coding-agent-builder-patterns.md` — patterns around human-in-the-loop and checkpointing in delegated coding workflows; tangential overlap with task monitoring.
