---
name: genesis-tech-watch
description: Technology watch for Claude Code — discovers new features, MCP servers, community skills, agent patterns, and best practices. Keeps the agent ecosystem current.
when_to_use: When Genesis needs to update its knowledge of Claude Code capabilities, find new tools, or report on ecosystem changes.
argument-hint: "[focus area] — e.g., 'new mcp servers' or 'claude code changelog' or 'community skills'"
allowed-tools: "Read Write Edit Bash Grep Glob WebSearch WebFetch"
---

# Tech Watch Protocol

Focus: **$ARGUMENTS**

## Watch Sources

### 1. Official Anthropic
Search and check:
- **Claude Code changelog**: Latest releases, new features, breaking changes
- **Anthropic blog**: announcements about Claude Code, Agent SDK, MCP
- **GitHub anthropics/**: claude-code repo releases, agent-sdk repos, MCP repos
- **Documentation**: docs.anthropic.com for updated guides

### 2. MCP Ecosystem
Search for:
- **New MCP servers**: GitHub trending with "mcp-server" topic
- **Smithery.ai**: New listings on MCP registry
- **npm @modelcontextprotocol/***: New official packages
- **Community MCPs**: High-star repos providing MCP servers

### 3. Community Skills & Agents
Search for:
- **GitHub "claude-code skill"**: New community skills
- **GitHub "claude-code agent"**: New agent patterns
- **Curated lists**: awesome-claude-code, buildwithclaude
- **Popular patterns**: What workflows are people automating?

### 4. Agent SDK & Tooling
Search for:
- **claude-agent-sdk updates**: New capabilities, API changes
- **Agent orchestration patterns**: Multi-agent, team coordination
- **Hook patterns**: New hook events, community hook recipes
- **Plugin ecosystem**: New Claude Code plugins

### 5. Competitor Intelligence
Quick scan of:
- **OpenAI Codex CLI**: New features we can match
- **Gemini CLI**: Capabilities worth emulating
- **Cursor/Windsurf**: IDE agent innovations
- **Other agent frameworks**: swarms, agency-swarm, crewAI

## Analysis Framework

For each discovery, evaluate:

| Question | Why It Matters |
|----------|----------------|
| Does this enable a new agent type? | Expand what we can build |
| Does this improve existing agents? | Upgrade current capabilities |
| Does this change best practices? | Update our patterns |
| Is this a breaking change? | Fix before it breaks us |
| Does this reduce cost/tokens? | Optimize operations |

## Reporting

### Full Report
```markdown
# Tech Watch Report — {date}

## New Features
- {feature}: {impact on our ecosystem}

## New MCP Servers
- {server}: {what it enables, stars, maturity}

## Community Highlights
- {skill/agent/pattern}: {what we can learn}

## Breaking Changes
- {change}: {action needed}

## Recommendations
1. {Most impactful action to take}
2. {Second priority}
3. {Third priority}
```

### Quick Update
For focused checks, just report:
- What changed
- Whether it affects us
- Recommended action (if any)

## Memory Update

After each tech watch:
1. Save significant findings to Genesis memory
2. Update any stale agent/skill configurations
3. Flag deprecated patterns for removal
4. Note new capabilities for future agent designs

## Automation

To schedule regular tech watch:
```
CronCreate: 0 10 * * 1  — "Run /genesis-tech-watch for weekly update"
```

This ensures the ecosystem stays current without manual intervention.
