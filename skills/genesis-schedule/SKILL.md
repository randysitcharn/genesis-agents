---
name: genesis-schedule
description: Create, manage, and optimize scheduled tasks, cron jobs, loops, and automated workflows for agents. Part of the Genesis meta-agent ecosystem.
when_to_use: When Genesis needs to set up automation, recurring tasks, monitoring loops, or cron-based schedules for agents.
argument-hint: "[action] [details] — e.g., 'create daily security scan at 9am' or 'list all schedules' or 'loop monitor deploy every 5m'"
allowed-tools: "Read Write Edit Bash Grep Glob"
user-invocable: false
---

# Schedule Management Protocol

Manage schedules for: **$ARGUMENTS**

## Available Scheduling Methods

### 1. `/loop` — Session-Scoped Polling
Best for: active monitoring during a work session
```
/loop [interval] [prompt]
/loop 5m check CI status
/loop 30s poll for deploy completion  
/loop (no interval) — Claude self-paces 1m-1h
```

### 2. `CronCreate` — Durable Cron Jobs
Best for: recurring tasks that persist across sessions
```
CronCreate with:
  - cron expression (5-field: min hour dom month dow)
  - prompt to execute
  - up to 50 tasks, 7-day auto-expiry
```

Common cron expressions:
```
*/5 * * * *     → every 5 minutes
0 * * * *       → every hour
0 9 * * 1-5     → weekdays at 9am
0 9,17 * * *    → 9am and 5pm daily
0 0 * * 0       → Sunday midnight
30 14 1 * *     → 1st of month at 2:30pm
```

### 3. `/schedule` — Cloud Routines
Best for: persistent automation that survives session end
```
/schedule "prompt" "cron-expression"
Runs on Anthropic cloud, fresh git clone each run
```

### 4. Hooks — Event-Driven Automation
Best for: reactive automation triggered by specific events
```json
{
  "hooks": {
    "{Event}": [{
      "matcher": "{pattern}",
      "hooks": [{
        "type": "command",
        "command": "{script}"
      }]
    }]
  }
}
```

## Actions

### Create Schedule
Based on the request, determine:
1. **What** needs to run (the prompt/command)
2. **When** it should run (interval, cron, event)
3. **Where** it should run (session, cloud, hook)
4. **Who** runs it (which agent, or main session)
5. **How long** it persists (session, 7-day, permanent)

### List Schedules
Use `CronList` to show all active cron jobs.
Check hooks in settings.json for event-driven automation.
Check for active `/loop` in current session.

### Optimize Schedule
Review existing schedules for:
- **Overlap**: Multiple schedules doing similar work
- **Frequency**: Too often wastes tokens, too rare misses events
- **Cost**: Haiku for simple checks, Opus only when needed
- **Jitter**: Recurring crons get up to 10% jitter (max 15min)

### Delete Schedule
Use `CronDelete` with the task ID from `CronList`.

## Schedule Design for Agents

### Monitoring Pattern
```
/loop 5m — Agent: monitor-{system}
  → Check health endpoint
  → Alert if degraded
  → Log status to memory
```

### Maintenance Pattern
```
CronCreate: 0 3 * * 0 — weekly cleanup
  → Archive completed handoffs
  → Compact knowledge base
  → Remove stale temporary files
```

### Review Pattern
```
CronCreate: 0 9 * * 1-5 — daily morning review
  → Check pending tasks and deadlines
  → Review overnight activity
  → Summarize status for stakeholders
```

### Tech Watch Pattern
```
CronCreate: 0 10 * * 1 — weekly Monday tech watch
  → /genesis-tech-watch
  → Search for Claude Code updates
  → Check new MCP servers
  → Report findings
```

## Output Format

```
Schedule Created:
  Type: {loop/cron/routine/hook}
  Expression: {cron or interval}
  Prompt: "{what it does}"
  Agent: {which agent runs it}
  Persists: {session/7-day/permanent}
  Next run: {estimated time}
```
