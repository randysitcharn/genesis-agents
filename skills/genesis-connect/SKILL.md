---
name: genesis-connect
description: Wire up agent communication after creation — adds the agent to the registry, defines its delegation chain, generates its agent card, and updates the topology in agents.md. Runs automatically at the end of genesis-create-agent.
when_to_use: After creating a new agent, when reorganizing agent topology, or when an agent's role/capabilities change. Also used to visualize the current communication topology.
argument-hint: "[agent-name] [action] — e.g., 'secretary' (auto-connect), 'show topology', 'rewire engineer'"
allowed-tools: "Read Write Edit Bash Grep Glob"
user-invocable: true
---

# Agent Connection Protocol

Wire up an agent into the communication ecosystem after creation.

## Parse Request

From `$ARGUMENTS`, determine:

| Pattern | Action |
|---------|--------|
| `<agent-name>` | Connect this agent (add to registry, define chain, generate card) |
| `show` / `topology` / `map` | Display the current communication topology |
| `rewire <agent-name>` | Reconfigure an existing agent's connections |
| `validate` | Check agents.md for inconsistencies |
| (empty) | Show topology |

## Manifest Location

The communication manifest is at `~/.claude/agents.md`. This is the single source of truth.

## Connect a New Agent

When connecting agent `$1`:

### Step 1: Read the Agent

Read `~/.claude/agents/$1/agent.md` to extract:
- name, description, model, tools
- What domain it belongs to (infer from description and existing topology)

### Step 2: Read Current Manifest

Read `~/.claude/agents.md` to understand:
- Existing agents and their domains
- Current delegation chains
- Who could logically delegate to this new agent
- Who this new agent could delegate to

### Step 3: Determine Connections

Based on the agent's role and domain, determine:

1. **Domain**: business | btp | support | genesis
2. **Reports to**: which agents can send requests to this one
   - Always include `genesis` and `user` for top-level agents
   - Include the domain lead (ceo for business, engineer for BTP)
   - Include direct supervisors based on the delegation tree
3. **Delegates to**: which agents this one can request work from
   - Only agents whose capabilities serve this agent's mission
   - Prefer existing leaf nodes over creating new delegation paths
4. **Peer access**: can `doc-keeper` be requested by this agent? (usually yes)

### Step 4: Generate Agent Card

Create a YAML card block:

```yaml
agent: $1
domain: <determined domain>
capabilities:
  - <extracted from agent description and responsibilities>
inputs: <what this agent needs>
outputs: <what this agent produces>
limitations: <what it cannot do>
```

### Step 5: Update agents.md

1. Add the agent to the appropriate registry table
2. Update "Can Delegate To" for agents that should be able to reach it
3. Update "Accepts Requests From" for the new agent
4. Add the agent card to the Agent Cards section
5. If the delegation tree diagram needs updating, update it

### Step 6: Notify

Report what was connected:
```
Connected: [agent-name]
  Domain: [domain]
  Reports to: [list]
  Delegates to: [list]
  Card: generated
  Manifest: updated
```

## Show Topology

Read `~/.claude/agents.md` and display:

1. The delegation tree (ASCII art)
2. Agent count per domain
3. Orphan detection (agents with no connections)
4. Hub detection (agents with too many connections)

## Rewire Agent

1. Read current connections from agents.md
2. Show current state
3. Propose changes based on the agent's current role
4. Apply changes after confirmation

## Validate

Check agents.md for:
- Agents in registry but missing from `~/.claude/agents/`
- Agents on disk but missing from registry
- Circular delegation chains
- Orphans (no "Accepts Requests From")
- Dead-ends (referenced in "Can Delegate To" but don't exist)
- Inconsistent permissions (delegating to an agent that doesn't accept from you)

Report issues as:
```
VALIDATION REPORT
  OK: X agents properly connected
  WARN: Y inconsistencies found
    - [agent] is in registry but not on disk
    - [agent] delegates to [other] but [other] doesn't accept from [agent]
  FIX: suggested corrections
```

## Workspace Setup

Ensure the shared workspace exists:
```bash
mkdir -p ~/.claude/workspace/{business,btp,support,shared}
```

## Integration with genesis-create-agent

This skill should be invoked automatically at the end of `genesis-create-agent`.
The creation skill builds the agent; this skill wires it into the ecosystem.

## Current Request

$ARGUMENTS
