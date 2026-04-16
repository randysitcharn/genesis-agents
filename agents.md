# Agent Communication Manifest

This file defines how agents discover, communicate, and coordinate with each other.
It is the single source of truth for the agent ecosystem's topology and rules.

## Agent Registry

Each agent declares its identity, capabilities, and communication interface.

### Genesis (Infrastructure)

| Agent | Model | Role | Accepts Requests From | Can Delegate To |
|-------|-------|------|-----------------------|-----------------|
| genesis | Opus | Meta-agent, agent factory | any | any |
| researcher | Sonnet | Deep research, read-only | genesis, ceo, engineer | none |
| reviewer | Opus | Adversarial quality review | genesis, ceo | none |
| builder | Opus | Implementation with gates | genesis | researcher, reviewer |
| monitor | Haiku | Fast health checks | genesis, ceo | none |

### Business

| Agent | Model | Role | Accepts Requests From | Can Delegate To |
|-------|-------|------|-----------------------|-----------------|
| ceo | Opus | Strategic decisions, delegation | user, genesis | secretary, admin, hr, marketing, legal, finance, researcher |
| secretary | Sonnet | Agenda, emails, meeting notes | ceo, admin, hr | none |
| admin | Sonnet | Accounting, invoices, compliance | ceo, finance | secretary |
| hr | Sonnet | Recruitment, onboarding, labor law | ceo | secretary |
| marketing | Sonnet | Campaigns, content, SEO | ceo | researcher |
| legal | Opus | Contracts, RGPD, risk | ceo, admin, hr | researcher |
| finance | Opus | Treasury, budgets, KPIs | ceo | admin |

### BTP

| Agent | Model | Role | Accepts Requests From | Can Delegate To |
|-------|-------|------|-----------------------|-----------------|
| engineer | Opus | Conception, dimensionnement | user, genesis, ceo | technician, estimator, researcher |
| site-manager | Sonnet | Planning chantier, coordination | engineer, ceo | technician, estimator |
| technician | Sonnet | Fiches techniques, calculs, normes | engineer, site-manager | none |
| estimator | Sonnet | Chiffrage, DQE, comparatifs | engineer, site-manager, ceo | none |

### Support

| Agent | Model | Role | Accepts Requests From | Can Delegate To |
|-------|-------|------|-----------------------|-----------------|
| backup | Sonnet | Sauvegarde, restauration, DR | user, genesis, ceo | none |
| monitoring | Haiku | Surveillance, alertes, logs | user, genesis, ceo | none |
| doc-keeper | Sonnet | Documentation, archivage, wiki | any | none |

## Communication Rules

### Rule 1: Delegation Chain
Agents can only delegate to agents listed in their "Can Delegate To" column.
An agent that receives a request it cannot handle MUST escalate up, not sideways.

```
user/genesis
    └→ ceo (strategic decisions)
        ├→ secretary (operational tasks)
        ├→ admin (administrative tasks)
        │   └→ secretary (support)
        ├→ hr (people tasks)
        │   └→ secretary (scheduling)
        ├→ marketing (growth tasks)
        │   └→ researcher (market research)
        ├→ legal (legal tasks)
        │   └→ researcher (legal research)
        ├→ finance (financial tasks)
        │   └→ admin (invoicing support)
        └→ researcher (any research need)

user/genesis
    └→ engineer (technical decisions)
        ├→ technician (field execution)
        ├→ estimator (cost estimation)
        ├→ site-manager (chantier coordination)
        │   ├→ technician
        │   └→ estimator
        └→ researcher (technical research)
```

### Rule 2: Request Format
Every inter-agent request MUST follow this structure:

```yaml
request:
  from: agent-name
  to: agent-name
  type: task | question | review | report
  priority: critical | high | normal | low
  subject: one-line summary
  context: relevant background (keep minimal)
  deliverable: what exactly is expected back
  deadline: when (if applicable)
  format: expected output format (if specific)
```

### Rule 3: Response Format
Every response MUST include:

```yaml
response:
  from: agent-name
  to: agent-name
  status: completed | partial | blocked | escalated
  deliverable: the actual output
  notes: caveats, assumptions, or follow-up needed
  escalation: (if blocked) who should handle this and why
```

### Rule 4: Shared Workspace
Agents deposit deliverables in a shared workspace for cross-agent visibility:

```
~/.claude/workspace/
  business/          ← business agent outputs
  btp/               ← BTP agent outputs
  support/           ← support agent outputs
  shared/            ← cross-domain deliverables
```

Naming convention: `YYYY-MM-DD_from-agent_subject.md`

### Rule 5: Escalation Protocol
When an agent cannot complete a request:
1. If the blocker is in their delegation chain → delegate down
2. If not → escalate to the agent that sent the request
3. If the requester was user/genesis → ask the user directly
4. Never silently drop a request — always respond with status

### Rule 6: Context Minimization
When delegating, pass ONLY the context the receiving agent needs.
Do NOT forward the entire conversation. Extract the relevant facts.
This preserves token budget and keeps agents focused.

### Rule 7: doc-keeper Access
`doc-keeper` accepts requests from ANY agent (the only exception to Rule 1).
Any agent can ask doc-keeper to document, archive, or retrieve information.

### Rule 8: Conflict Resolution
When two agents produce conflicting outputs:
1. The requesting agent (parent) decides
2. If the parent cannot decide → escalate to ceo (business) or engineer (BTP)
3. If still unresolved → escalate to user
4. Document the conflict and resolution in the workspace

## Agent Cards

Each agent exposes a card summarizing its capabilities. Genesis reads these
when deciding who to delegate to.

### Card Format
```yaml
agent: name
domain: business | btp | support | genesis
capabilities:
  - short description of capability 1
  - short description of capability 2
inputs: what this agent needs to work (context, documents, data)
outputs: what this agent produces (reports, documents, scripts)
limitations: what this agent cannot or should not do
```

## How Genesis Uses This File

1. **After creating an agent** → Genesis runs `genesis-connect` to:
   - Add the agent to the registry table
   - Define its delegation chain (who it reports to, who it can delegate to)
   - Generate its agent card
   - Update the communication topology

2. **Before delegating** → Genesis reads this file to find the right agent

3. **On conflict** → Genesis reads the escalation protocol

4. **On status check** → Genesis reads the registry to report the ecosystem state
