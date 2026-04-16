---
name: genesis-discover-mcp
description: Discover, evaluate, and configure MCP servers for agents. Searches per-agent AND per-department. Presents a ranked list for user approval before any configuration. Searches GitHub, npm, Smithery, Glama, PulseMCP, and Awesome MCP lists.
when_to_use: When Genesis needs to find appropriate MCP servers for an agent, or user asks about available MCPs.
argument-hint: "[scope] — e.g., 'all', 'business', 'btp', 'support', 'secretary', 'engineer'"
allowed-tools: "Read Write Edit Bash Grep Glob WebSearch WebFetch Agent"
user-invocable: true
---

# MCP Discovery Protocol

Discover and propose MCP servers for: **$ARGUMENTS**

## Parse Scope

| Pattern | Action |
|---------|--------|
| `all` / (empty) | Discover for ALL agents + ALL departments |
| `business` / `btp` / `support` | Discover for one department (shared + per-agent) |
| `<agent-name>` | Discover for one specific agent |

## Step 1: Inventory

### Current MCPs
```
Configured MCPs: !`grep -A1 '"mcpServers"' ~/.claude/settings.json 2>/dev/null | head -30`
```

### Agents in Scope
Read `~/.claude/agents.md` to get the list of agents and their roles.
For each agent in scope, read its `agent.md` to understand:
- What it does (responsibilities)
- What tools it already has
- What domain it operates in
- What external systems it would benefit from

## Step 2: Search — Per Agent

For EACH agent in scope, launch a parallel research agent to search for relevant MCPs.

### Search Queries per Agent

For each agent, construct search queries based on its responsibilities:

| Agent | Search Keywords |
|-------|----------------|
| **secretary** | calendar, email, contacts, scheduling, meeting notes, reminders |
| **ceo** | analytics, dashboard, project management, OKR, reporting |
| **admin** | spreadsheet, invoice, PDF, accounting, document management |
| **hr** | recruitment, ATS, job board, onboarding, forms |
| **marketing** | SEO, social media, analytics, content, email marketing |
| **legal** | document analysis, PDF, contract, RGPD, compliance |
| **finance** | spreadsheet, accounting, banking, currency, tax |
| **technician** | calculator, PDF, filesystem, weather, standards database |
| **engineer** | CAD, calculator, PDF, spreadsheet, standards, geolocation |
| **site-manager** | project management, gantt, calendar, weather, maps |
| **estimator** | spreadsheet, calculator, market prices, currency |
| **backup** | filesystem, S3, cloud storage, docker, database, SSH |
| **monitoring** | prometheus, grafana, logs, docker, system stats, alerting |
| **doc-keeper** | markdown, wiki, notion, confluence, git, filesystem |

### Search Sources

For each set of keywords, search:
1. **GitHub**: `mcp-server-{keyword}`, `mcp {keyword} server`
2. **npm**: `@modelcontextprotocol/*`, `mcp-server-{keyword}`
3. **Smithery**: smithery.ai — search MCP registry
4. **Glama**: glama.ai/mcp/servers — MCP directory
5. **PulseMCP**: pulsemcp.com — MCP discovery
6. **Awesome MCP lists**: curated lists on GitHub

## Step 3: Search — Per Department

For each department in scope, search for SHARED MCPs that benefit multiple agents:

| Department | Shared MCP Keywords |
|-----------|-------------------|
| **Business** | Google Workspace (Drive, Calendar, Gmail, Sheets), Slack, Notion, CRM, Microsoft 365 |
| **BTP** | filesystem, PDF, calculator, weather API, geolocation, project management |
| **Support** | filesystem, SSH, Docker, cloud storage (S3/GCS), database, git |

## Step 4: Evaluate & Rank

For each discovered MCP, score on:

| Criteria | Weight | Scale |
|----------|--------|-------|
| **Relevance** | 30% | How well does it match the agent's needs? (1-5) |
| **Maintenance** | 20% | Last commit < 3 months? Active? (1-5) |
| **Stars/Downloads** | 15% | Community trust (1-5) |
| **Security** | 15% | Reputable author? Permissions scope? (1-5) |
| **Ease of setup** | 10% | stdio > SSE, npx > docker > manual build (1-5) |
| **Agent coverage** | 10% | How many agents in scope benefit? (1-5) |

**Score = weighted average → rank by score descending**

## Step 5: Present for Approval

NEVER auto-install. Present the ranked list to the user and WAIT for approval.

### Presentation Format

```
MCP DISCOVERY RESULTS — {scope}
════════════════════════════════

DEPARTMENT-WIDE MCPs (shared across multiple agents)
─────────────────────────────────────────────────────

 #1 ★★★★★ [score: 4.8] mcp-server-google-workspace
    Repo: github.com/...  |  Stars: XXX  |  Updated: YYYY-MM
    What: Google Calendar + Drive + Gmail + Sheets in one server
    Agents: secretary, ceo, admin, hr, marketing (5/7 business)
    Install: npx @example/mcp-server-google-workspace
    Auth: Google OAuth2
    → APPROVE? [Y/n]

 #2 ★★★★☆ [score: 4.2] mcp-server-slack
    Repo: github.com/...  |  Stars: XXX  |  Updated: YYYY-MM
    What: Send/read Slack messages, channels, threads
    Agents: secretary, ceo, marketing (3/7 business)
    Install: npx @example/mcp-server-slack
    Auth: Slack Bot Token
    → APPROVE? [Y/n]

PER-AGENT MCPs
──────────────

 secretary:
   #1 ★★★★☆ [score: 4.1] mcp-server-calendar
      What: Calendar management (create, read, update events)
      Install: npx ...
      Auth: ...
      → APPROVE? [Y/n]

 admin:
   #1 ★★★★☆ [score: 4.0] mcp-server-pdf
      What: Read, parse, and extract data from PDFs
      Install: npx ...
      Auth: none
      → APPROVE? [Y/n]

   #2 ★★★☆☆ [score: 3.5] mcp-server-spreadsheet
      What: Read/write Excel and CSV files
      Install: ...
      → APPROVE? [Y/n]

 ...

SUMMARY
───────
  Total MCPs found: N
  Department-wide: X
  Per-agent: Y
  Awaiting your approval to configure.
```

## Step 6: Configure Approved MCPs

ONLY after user explicitly approves each MCP:

### 6a. Install
```bash
# Example for npx-based MCP
npx @example/mcp-server-name --version
```

### 6b. Add to settings.json

For stdio transport:
```json
{
  "mcpServers": {
    "{server-name}": {
      "command": "{runtime}",
      "args": ["{path-or-package}"],
      "env": {
        "API_KEY": "{placeholder — ask user for value}"
      }
    }
  }
}
```

For SSE transport:
```json
{
  "mcpServers": {
    "{server-name}": {
      "type": "sse",
      "url": "{endpoint-url}",
      "headers": {
        "Authorization": "Bearer ${ENV_VAR}"
      }
    }
  }
}
```

### 6c. Add Permissions
Add `mcp__{server-name}__*` to the allow list in settings.json.

### 6d. Wire to Agents
For each approved MCP, update the relevant agents:
1. Add MCP tool references to agent's `tools` in frontmatter
2. Add a note in the agent's Communication section about available MCP tools
3. Update `agents.md` if the agent's capabilities changed

### 6e. Prompt for Credentials
If the MCP requires auth (API keys, OAuth), prompt the user:
```
MCP {name} requires authentication:
  - {credential-name}: {description}
  
Please provide the value, or set it as environment variable: {ENV_VAR_NAME}
```

NEVER store credentials in agent.md or SKILL.md files.
Store only env var references in settings.json.

## Step 7: Report

```
MCP CONFIGURATION COMPLETE
══════════════════════════
  Approved: X MCPs
  Rejected: Y MCPs
  Configured:
    - {mcp-1} → {agent-list} ✓
    - {mcp-2} → {agent-list} ✓
  Pending auth:
    - {mcp-3} → needs {credential} from user
```

## Current Request

$ARGUMENTS
