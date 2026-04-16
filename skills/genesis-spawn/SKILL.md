---
name: genesis-spawn
description: Bootstrap an agent end-to-end — install its MCPs, create its workspace, then launch it with first-run onboarding actions. The single command that brings an agent to life.
when_to_use: When the user wants to activate an agent for the first time, or re-initialize an agent. Also used to bootstrap an entire department at once.
argument-hint: "[scope] — e.g., 'secretary', 'business', 'btp', 'support', 'all'"
allowed-tools: "Read Write Edit Bash Grep Glob Agent"
user-invocable: true
---

# Agent Spawn Protocol

Bring an agent to life: install MCPs → create workspace → launch with onboarding.

## Parse Request

| Pattern | Action |
|---------|--------|
| `<agent-name>` | Spawn one agent |
| `business` / `btp` / `support` | Spawn all agents in a department |
| `all` | Spawn all domain agents (not genesis infrastructure) |
| `status` | Show which agents are spawned vs pending |

## Agent Groups

```
business: secretary, ceo, admin, hr, marketing, legal, finance
btp: technician, engineer, site-manager, estimator
support: backup, monitoring, doc-keeper
```

## Spawn Sequence

For each agent to spawn, execute these steps IN ORDER:

### Step 1: Pre-flight Check

```bash
# Verify agent exists
ls ~/.claude/agents/{name}/agent.md

# Check if already spawned (has .spawned marker)
ls ~/.claude/agents/{name}/.spawned 2>/dev/null
```

If `.spawned` exists, ask: "Agent {name} was already spawned. Re-initialize? [Y/n]"

### Step 2: Install MCPs

Run `/genesis-install-mcp {name}` to:
1. Read mcp-catalog.md for this agent's MCPs
2. Install missing MCPs into settings.json
3. Add permissions
4. Handle credentials

If MCPs require auth that isn't configured yet, warn but continue — the agent can work without them.

### Step 3: Create Workspace

```bash
# Determine domain
# business → ~/.claude/workspace/business/
# btp → ~/.claude/workspace/btp/
# support → ~/.claude/workspace/support/

mkdir -p ~/.claude/workspace/{domain}/{name}
```

### Step 4: Launch Agent with Onboarding

Spawn the agent using the Agent tool with an onboarding prompt built from the agent's First Launch section.

The onboarding prompt template:

```
You are {name}, being activated for the first time.

Read your agent definition at ~/.claude/agents/{name}/agent.md to understand your role.
Read ~/.claude/agents.md to understand your place in the ecosystem.

Execute your First Launch checklist:
{first_launch_actions from agent.md}

After completing onboarding, deposit a summary at:
~/.claude/workspace/{domain}/{name}/ONBOARDING.md

Include:
- What you set up
- What's ready to use
- What needs user input (credentials, preferences, data)
- Your recommended first real task
```

### Step 5: Mark as Spawned

```bash
# Create spawn marker with timestamp
echo "spawned: $(date -Iseconds)" > ~/.claude/agents/{name}/.spawned
echo "mcps: {list of installed MCPs}" >> ~/.claude/agents/{name}/.spawned
echo "workspace: ~/.claude/workspace/{domain}/{name}" >> ~/.claude/agents/{name}/.spawned
```

### Step 6: Report

```
SPAWN COMPLETE: {name}
═════════════════════
  MCPs installed: X (Y already existed)
  Workspace: ~/.claude/workspace/{domain}/{name}/
  Onboarding: completed
  
  Ready for: {first recommended task}
```

## Department Spawn Order

When spawning an entire department, respect the delegation hierarchy — spawn leads first:

**Business:** ceo → finance → legal → admin → hr → marketing → secretary
**BTP:** engineer → site-manager → estimator → technician
**Support:** doc-keeper → backup → monitoring

Leads first because their onboarding may create structures that subordinates depend on.

## First Launch Actions per Agent

Each agent has domain-specific onboarding. Here's what each should do:

### Business

**ceo:**
- Create `~/.claude/workspace/business/strategy/` directory
- Generate a blank OKR template
- Generate a delegation matrix summary from agents.md
- Create a weekly review template

**secretary:**
- Create `~/.claude/workspace/business/secretary/templates/` directory
- Generate email templates (follow-up, meeting invite, thank you)
- Generate a meeting notes template
- Create a task tracking template

**admin:**
- Create `~/.claude/workspace/business/admin/templates/` directory
- Generate an invoice template
- Generate an expense report template
- Create a compliance calendar template
- Set up folder structure: invoices/, expenses/, compliance/, documents/

**hr:**
- Create `~/.claude/workspace/business/hr/templates/` directory
- Generate a job description template
- Generate an onboarding checklist template
- Generate a performance review template
- Set up folder structure: recruitment/, onboarding/, reviews/, policies/

**marketing:**
- Create `~/.claude/workspace/business/marketing/templates/` directory
- Generate a campaign brief template
- Generate a content calendar template
- Generate a social media post template
- Set up folder structure: campaigns/, content/, analytics/, brand/

**legal:**
- Create `~/.claude/workspace/business/legal/templates/` directory
- Generate a contract review checklist
- Generate a RGPD compliance checklist
- Generate a legal risk assessment template
- Set up folder structure: contracts/, policies/, compliance/, watch/

**finance:**
- Create `~/.claude/workspace/business/finance/templates/` directory
- Generate a budget template
- Generate a cash flow forecast template
- Generate a KPI dashboard template
- Set up folder structure: budgets/, forecasts/, reports/, invoices/

### BTP

**engineer:**
- Create `~/.claude/workspace/btp/engineer/templates/` directory
- Generate a note de calcul template
- Generate a CCTP section template
- Generate a design review checklist
- Set up folder structure: projets/, notes-de-calcul/, cctp/, plans/

**site-manager:**
- Create `~/.claude/workspace/btp/site-manager/templates/` directory
- Generate a CR de chantier template
- Generate a weekly planning template
- Generate a trade coordination matrix template
- Set up folder structure: chantiers/, cr/, planning/, coordination/

**estimator:**
- Create `~/.claude/workspace/btp/estimator/templates/` directory
- Generate a DQE template
- Generate a comparative analysis template
- Generate a budget tracking template
- Set up folder structure: devis/, dqe/, comparatifs/, suivi-budget/

**technician:**
- Create `~/.claude/workspace/btp/technician/templates/` directory
- Generate a field checklist template
- Generate an intervention report template
- Generate a technical sheet template
- Set up folder structure: fiches-techniques/, interventions/, checklists/, calculs/

### Support

**doc-keeper:**
- Create `~/.claude/workspace/shared/doc-keeper/templates/` directory
- Generate a .gitattributes with LFS rules for all binary types
- Generate a VERSIONS.md template
- Generate a project repository structure template
- Set up folder structure: templates/, archives/
- Initialize Git LFS if not already done: `git lfs install`

**backup:**
- Create `~/.claude/workspace/support/backup/` directory
- Generate a backup strategy template (3-2-1 rule)
- Generate a disaster recovery plan template
- Generate a test restore checklist
- Inventory current backup state (what exists, what's missing)

**monitoring:**
- Create `~/.claude/workspace/support/monitoring/` directory
- Generate an alert rules template
- Generate a dashboard layout template
- Generate a health check checklist
- Inventory current monitoring state

## Current Request

$ARGUMENTS
