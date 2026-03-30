---
title: "Operational Lessons Learned"
type: permanent_canonical
created: 2026-03-29
last_updated: 2026-03-29
---

# Lessons Learned

durable operational lessons. each entry should be useful beyond the week it was learned.

---

## cron vs launchagent (2026-03-06)

- cron has no PATH, no keychain, no GUI session. useless for anything touching homebrew binaries or claude CLI oauth.
- fix PATH: set env vars at top of crontab (applies to all jobs).
- fix keychain: use LaunchAgents. they run in user session with full keychain access.
- claude CLI stores Pro plan auth in macOS keychain, not on disk. headless contexts can't reach it.
- claude CLI sandboxes file access to cwd. set cwd to workspace root.
- always check exit codes in scripts. "completed" doesn't mean "succeeded."

---

## gateway resilience (2026-03-06)

- gateway must be installed as LaunchAgent (`openclaw gateway install`) for auto-restart.
- can only install from GUI terminal on mac mini (not SSH/headless).
- health check script + cron monitors gateway every 5min, alerts via telegram.
