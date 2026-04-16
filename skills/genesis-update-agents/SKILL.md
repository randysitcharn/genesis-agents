---
name: genesis-update-agents
description: Batch-update existing agents to align with the latest genesis-create-agent standards. Audits all agents, detects what's missing or outdated, and applies fixes — communication wiring, missing sections, frontmatter alignment, workspace setup.
when_to_use: After evolving genesis-create-agent or agents.md, when agents are out of sync with current standards. Also useful after adding new ecosystem features (connect, quality gates, etc.) that existing agents don't know about.
argument-hint: "[scope] — e.g., 'all', 'business', 'btp', 'support', 'secretary', or 'dry-run' to preview without changes"
allowed-tools: "Read Write Edit Bash Grep Glob"
user-invocable: true
---

# Agent Batch Update Protocol

Update existing agents to match current `genesis-create-agent` standards.

## Parse Request

From `$ARGUMENTS`, determine:

| Pattern | Action |
|---------|--------|
| `all` / (empty) | Update all agents (except genesis itself) |
| `business` | Update business agents only |
| `btp` | Update BTP agents only |
| `support` | Update support agents only |
| `<agent-name>` | Update one specific agent |
| `dry-run [scope]` | Audit and report without making changes |

## Current Standards Checklist

Read `~/.claude/skills/genesis-create-agent/SKILL.md` to extract the latest standards.
Read `~/.claude/agents.md` to get the communication manifest.

For each agent, verify:

### 1. Frontmatter Completeness
- [ ] `name` — present and matches directory name
- [ ] `description` — specific enough for auto-delegation
- [ ] `model` — valid model (claude-opus-4-6, claude-sonnet-4-6, claude-haiku-4-5)
- [ ] `tools` — minimal set (least privilege)
- [ ] `memory` — present (true/false)

### 2. Body Structure
- [ ] Has a clear **role definition** (who you are)
- [ ] Has **core responsibilities** (numbered list)
- [ ] Has **output standards** (what deliverables look like)
- [ ] Has **principles** (behavioral rules)

### 3. Communication Awareness
- [ ] Knows its place in the delegation chain (who it reports to, who it can delegate to)
- [ ] Knows the request/response format from agents.md
- [ ] Knows about the shared workspace and where to deposit deliverables
- [ ] Knows the escalation protocol (what to do when blocked)

### 4. Ecosystem Integration
- [ ] Listed in `~/.claude/agents.md` registry
- [ ] Has an agent card in agents.md
- [ ] Workspace directory exists (`~/.claude/workspace/{domain}/`)

## Execution

### Step 1: Audit

For each agent in scope, read its `agent.md` and check against the standards checklist.
Produce an audit report:

```
AUDIT: agent-name
  Frontmatter:  OK | MISSING: [fields]
  Body:         OK | MISSING: [sections]
  Communication: OK | MISSING: [items]
  Integration:  OK | MISSING: [items]
  Verdict:      UP-TO-DATE | NEEDS UPDATE
```

### Step 2: Plan Changes

For each agent that needs update, plan the specific edits:

- **Add communication section** if missing — inject after Principles:
  ```markdown
  ## Communication

  You operate within the Genesis agent ecosystem.

  **Reports to**: [from agents.md]
  **Can delegate to**: [from agents.md]
  **Shared workspace**: `~/.claude/workspace/{domain}/`

  ### Receiving Requests
  You may receive structured requests from authorized agents. Always check
  that the requesting agent is in your "Reports to" list.

  ### Making Requests
  When delegating, use the standard request format:
  - Specify: subject, context (minimal), deliverable expected, deadline
  - Only delegate to agents in your "Can delegate to" list

  ### Escalation
  If you cannot complete a request:
  1. If the blocker can be handled by a delegate → delegate down
  2. Otherwise → escalate back to the requesting agent with status: blocked
  3. Never silently drop a request

  ### Deliverables
  Deposit outputs in `~/.claude/workspace/{domain}/` using naming convention:
  `YYYY-MM-DD_{your-name}_{subject}.md`
  ```

- **Add to agents.md** if missing — run genesis-connect logic
- **Create workspace dir** if missing

### Step 3: Apply (unless dry-run)

Apply all planned edits. For each agent:
1. Edit `agent.md` to add missing sections
2. Update `agents.md` if not registered
3. Create workspace directory if missing

### Step 4: Report

```
UPDATE COMPLETE
═══════════════
  Audited:  N agents
  Updated:  M agents
  Skipped:  K agents (already up-to-date)

  Changes applied:
    - agent-1: +communication section, +agents.md registry
    - agent-2: +communication section, +workspace dir
    - agent-3: already up-to-date (skipped)
    ...
```

## Current Request

$ARGUMENTS
