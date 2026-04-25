---
date: 2026-03-30
type: lesson
tags: [cron, operations, scheduling, maintenance]
---

# lesson: treat one-off cron jobs as expiring obligations

## context
Repeated heartbeat checks showed a one-off cron for `run-harness-research.sh` lingering past its intended run time. The script itself still mattered, but the scheduled entry had become stale and easy to ignore.

## lesson
One-off cron jobs should be treated as expiring obligations. After their intended run window, either confirm they ran and remove them, or explicitly re-schedule them. Do not let stale one-off entries sit in crontab as ambiguous debris.

## why it matters
- stale one-off cron entries create false confidence that work is scheduled when it is not
- old entries add operational noise and make real automation state harder to read
- missed scheduled research or maintenance can quietly stall follow-up work

## apply when
- a cron entry has a single intended run time
- a scheduled script still matters after the original window passes
- heartbeat or maintenance reviews find old one-off entries still present
