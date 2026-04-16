---
name: genesis-discover-mcp
description: Discover, evaluate, and configure MCP servers for agents. Searches GitHub, npm, and MCP registries to find the best MCP servers for a given domain or task.
when_to_use: When Genesis needs to find appropriate MCP servers for an agent, or user asks about available MCPs.
argument-hint: "[domain or task] — e.g., 'database management' or 'github operations'"
allowed-tools: "Read Write Edit Bash Grep Glob WebSearch WebFetch"
user-invocable: false
---

# MCP Discovery Protocol

Find and configure MCP servers for: **$ARGUMENTS**

## Step 1: Inventory Current MCPs

Check what's already configured:
```
Current MCP servers: !`cat ~/.claude/settings.json 2>/dev/null | grep -A2 '"mcpServers"' | head -20`
```

Also check project-level:
```
Project MCPs: !`cat .claude/settings.json 2>/dev/null | grep -A2 '"mcpServers"' | head -20`
```

## Step 2: Search for MCP Servers

Search multiple sources for MCP servers matching the domain:

### GitHub Search
Search for:
- `mcp-server-{domain}` repositories
- `@modelcontextprotocol/server-{domain}` packages
- `mcp {domain} server` in repo descriptions
- Check stars, recent activity, and maintenance status

### npm Registry
Search for:
- `@modelcontextprotocol/*` official packages
- `mcp-server-*` community packages
- Check weekly downloads and last publish date

### Known MCP Registries
Check these sources:
- **Anthropic official**: github.com/anthropics — official MCP servers
- **MCP Hub**: github.com/ravitemer/mcp-hub — centralized manager
- **Awesome MCP**: curated lists of MCP servers on GitHub
- **Smithery**: smithery.ai — MCP server registry
- **Glama**: glama.ai/mcp/servers — MCP directory
- **PulseMCP**: pulsemcp.com — MCP discovery

## Step 3: Evaluate Candidates

For each discovered MCP server, assess:

| Criteria | Weight | Check |
|----------|--------|-------|
| **Relevance** | High | Does it solve the exact need? |
| **Maintenance** | High | Last commit < 3 months? Active issues? |
| **Stars/Downloads** | Medium | Community trust signal |
| **Security** | High | Reputable author? Code audited? |
| **Transport** | Medium | stdio (local) vs SSE (remote)? |
| **Dependencies** | Low | Heavy runtime requirements? |
| **Auth** | Medium | What credentials needed? |

## Step 4: Configuration

For the selected MCP server, generate the configuration:

### stdio Transport (local process)
```json
{
  "mcpServers": {
    "{server-name}": {
      "command": "{runtime}",
      "args": ["{path-or-package}"],
      "env": {
        "API_KEY": "{placeholder}"
      }
    }
  }
}
```

### SSE Transport (remote HTTP)
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

## Step 5: Install & Configure

1. **Install**: Run the install command (npm, pip, etc.)
2. **Configure**: Add to appropriate settings.json (user or project)
3. **Permissions**: Add `mcp__{server}__*` to allow list
4. **Env vars**: Document required environment variables
5. **Test**: Verify server starts and tools are available

## Step 6: Wire to Agent

If configuring for a specific agent:
1. Add MCP tools to agent's `allowed-tools` in frontmatter
2. Add permission rules to settings.json
3. Create a skill that leverages the MCP tools
4. Document the MCP in the agent's description

## Output Format

```
MCP Discovery Results for: {domain}

Recommended:
  Server: {name}
  Source: {url}
  Transport: {stdio/sse}
  Tools: {list of key tools}
  Install: {command}
  Auth: {what's needed}
  
Configuration added to: {settings file}
Permissions: mcp__{name}__*

Alternative options:
  - {other-server-1}: {why it's an alternative}
  - {other-server-2}: {why it's an alternative}
```
