---
name: genesis-catalog
description: List, search, and describe skills and agents in the Genesis ecosystem. The skill registry — find what's available, what it does, and how to use it.
when_to_use: When the user asks what skills/agents exist, searches for a capability, or wants to know how to use a specific skill or agent.
argument-hint: "[action] [query] — e.g., 'list', 'search mcp', 'describe genesis-handoff', 'agents', 'categories'"
allowed-tools: "Read Grep Glob Bash"
user-invocable: true
---

# Skill & Agent Catalog

## Parse Request

From `$ARGUMENTS`, determine the action:

| Pattern | Action |
|---------|--------|
| (empty) / `list` / `all` | List all skills and agents |
| `search <query>` / `find <query>` | Search by keyword in names + descriptions |
| `describe <name>` / `info <name>` / `show <name>` | Show detailed info for one skill/agent |
| `agents` | List agents only |
| `skills` | List skills only |
| `categories` / `groups` | Show skills grouped by category |

## Data Sources

Skills directory:
```
!`ls ~/.claude/skills/ 2>/dev/null | sort`
```

Agents directory:
```
!`ls ~/.claude/agents/ 2>/dev/null | sort`
```

## Execution

### List All

For each skill in `~/.claude/skills/*/SKILL.md`, extract from frontmatter:
- `name` — skill identifier
- `description` — one-line summary
- `user-invocable` — whether the user can call it directly (default: true)
- `when_to_use` — trigger conditions

For each agent in `~/.claude/agents/*/agent.md`, extract:
- `name`, `description`, `model`, `tools`

Present as a clean table:

```
SKILLS (N total, M user-invocable)
──────────────────────────────────
/skill-name          — description (first 60 chars)
  └ trigger: when_to_use summary

AGENTS (N total)
────────────────
agent-name [model]  — description (first 60 chars)
  └ tools: tool list
```

### Search

1. Read `$1` as the search query
2. For each skill, search `name`, `description`, `when_to_use` for the query (case-insensitive)
3. For each agent, search `name`, `description` for the query
4. Return matches ranked by: exact name match > description match > when_to_use match
5. If no matches: suggest closest alternatives

### Describe

1. Read `$1` as the skill or agent name
2. Check both `~/.claude/skills/$1/SKILL.md` and `~/.claude/agents/$1/agent.md`
3. Display the full frontmatter as a structured card:

```
╭─ SKILL: genesis-handoff ─────────────────────────────╮
│ Description: Create and resume session handoffs...    │
│ Trigger: /genesis-handoff [create|resume] [context]   │
│ Invocable: yes                                        │
│ Tools: Read Write Edit Bash Grep Glob                 │
│ When to use: At session end, before compaction...     │
│                                                       │
│ Arguments:                                            │
│   [create|resume] [context]                           │
│   e.g., 'create building auth module'                 │
│         'resume handoff-2024-04-16'                   │
╰───────────────────────────────────────────────────────╯
```

### Categories

Group skills by prefix and function:

| Category | Skills |
|----------|--------|
| **Genesis Core** | genesis (orchestrator) |
| **Agent Lifecycle** | genesis-create-agent, genesis-create-skill, genesis-catalog |
| **MCP & Tools** | genesis-discover-mcp |
| **Automation** | genesis-schedule |
| **Learning Loop** | genesis-self-reflect, genesis-compound, genesis-prime |
| **Quality** | genesis-quality-gate |
| **Continuity** | genesis-handoff |
| **Intelligence** | genesis-tech-watch, genesis-status |
| **Other** | (non-genesis skills) |

## Current Request

$ARGUMENTS
