---
name: genesis-install-mcp
description: Install and configure MCP servers from the Genesis base catalog into settings.json. Handles installation, configuration, permissions, and credential prompts. The operational counterpart to genesis-discover-mcp.
when_to_use: After agent creation or update when MCPs need to be installed. When the user wants to set up MCP servers. When genesis-update-agents detects missing MCP configurations.
argument-hint: "[scope] — e.g., 'all', 'business', 'btp', 'support', 'secretary', 'filesystem', 'google-workspace'"
allowed-tools: "Read Write Edit Bash Grep Glob"
user-invocable: true
---

# MCP Installation Protocol

Install and configure MCP servers from the Genesis catalog.

## Parse Request

From `$ARGUMENTS`, determine:

| Pattern | Action |
|---------|--------|
| `all` | Install all MCPs from the catalog |
| `business` / `btp` / `support` | Install MCPs relevant to that department |
| `<agent-name>` | Install MCPs assigned to that agent |
| `<mcp-id>` | Install one specific MCP (e.g., `filesystem`, `google-workspace`) |
| `status` | Show which MCPs are installed vs pending |
| `dry-run [scope]` | Show what would be installed without doing it |

## Step 1: Read Current State

### Catalog
```
!`cat ~/.claude/agents/genesis/mcp-catalog.md 2>/dev/null | head -80`
```

### Already Configured
Read `~/.claude/settings.json` and extract the `mcpServers` section.
Build a list of already-installed MCP IDs.

## Step 2: Determine What to Install

Based on scope:
1. Read `mcp-catalog.md` relevance matrix
2. Collect MCPs needed for the scope
3. Subtract already-installed MCPs
4. Present the installation plan:

```
MCP INSTALLATION PLAN
═════════════════════

Already installed:
  ✓ sequential-thinking
  ✓ memory

To install:
  1. filesystem         — npx @modelcontextprotocol/server-filesystem
  2. google-workspace   — uvx workspace-mcp (requires OAuth2 setup)
  3. calculator         — uvx mcp-server-calculator

Requires credentials:
  ⚠ google-workspace   — Google OAuth2 (client ID + secret)
  ⚠ github             — GitHub Personal Access Token

Proceed? [Y/n]
```

WAIT for user approval before installing.

## Step 3: Install Each MCP

For each approved MCP, follow the catalog entry:

### 3a. Verify Runtime

Check that the required runtime is available:
```bash
# For npx-based MCPs
which npx 2>/dev/null && echo "npx: OK" || echo "npx: MISSING — install Node.js"

# For uvx/pip-based MCPs
which uvx 2>/dev/null && echo "uvx: OK" || echo "uvx: MISSING — install uv (pip install uv)"

# For Go-based MCPs
which go 2>/dev/null && echo "go: OK" || echo "go: MISSING"
```

If a runtime is missing, warn the user and skip that MCP.

### 3b. Test Installation

Before configuring, verify the MCP can be reached:
```bash
# For npx packages — check they exist
npx -y <package> --help 2>/dev/null && echo "OK" || echo "FAILED"

# For uvx packages
uvx <package> --help 2>/dev/null && echo "OK" || echo "FAILED"
```

### 3c. Add to settings.json

Read `~/.claude/settings.json`, add the MCP to `mcpServers`:

#### MCP Configurations Reference

**filesystem:**
```json
"filesystem": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "~/.claude/workspace"]
}
```

**document-loader:**
```json
"document-loader": {
  "command": "uvx",
  "args": ["awslabs.document-loader-mcp-server@latest"]
}
```

**pdf-reader:**
```json
"pdf-reader": {
  "command": "npx",
  "args": ["-y", "@sylphx/pdf-reader-mcp"]
}
```

**markdownify:**
```json
"markdownify": {
  "command": "npx",
  "args": ["-y", "markdownify-mcp"]
}
```

**pandoc:**
```json
"pandoc": {
  "command": "uvx",
  "args": ["mcp-pandoc"],
  "env": {}
}
```

**google-workspace:**
```json
"google-workspace": {
  "command": "uvx",
  "args": ["workspace-mcp", "--tool-tier", "core"],
  "env": {
    "GOOGLE_CLIENT_ID": "${GOOGLE_CLIENT_ID}",
    "GOOGLE_CLIENT_SECRET": "${GOOGLE_CLIENT_SECRET}"
  }
}
```

**ms365:**
```json
"ms365": {
  "command": "npx",
  "args": ["-y", "@softeria/ms-365-mcp-server"],
  "env": {}
}
```

**calculator:**
```json
"calculator": {
  "command": "uvx",
  "args": ["mcp-server-calculator"]
}
```

**sequential-thinking:**
```json
"sequential-thinking": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
}
```

**memory:**
```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"]
}
```

**ssh:**
```json
"ssh": {
  "command": "npx",
  "args": ["-y", "@fangjunjie/ssh-mcp-server"]
}
```

**git:**
```json
"git": {
  "command": "uvx",
  "args": ["mcp-server-git"]
}
```

**github:**
```json
"github": {
  "command": "npx",
  "args": ["-y", "@github/mcp-server"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
  }
}
```

## Step 4: Add Permissions

For each installed MCP, add permission to `~/.claude/settings.json` under `permissions.allow`:

```json
"permissions": {
  "allow": [
    "mcp__filesystem__*",
    "mcp__calculator__*",
    "mcp__git__*"
  ]
}
```

## Step 5: Handle Credentials

For MCPs that require auth, prompt the user:

```
MCP: google-workspace
  Requires: Google OAuth2 credentials
  
  Option A: Set environment variables
    export GOOGLE_CLIENT_ID=your-client-id
    export GOOGLE_CLIENT_SECRET=your-client-secret
    
  Option B: I'll guide you through Google Cloud Console setup
  
  Which option? [A/B/skip]
```

For MCPs with no auth → proceed silently.
For MCPs with optional auth → mention it but proceed without.
For MCPs requiring auth → MUST get credentials before marking as installed.

NEVER store raw credentials in settings.json — only `${ENV_VAR}` references.

## Step 6: Verify Installation

After configuring, verify each MCP:
1. Check settings.json has the entry
2. Check permissions are in allow list
3. Report status

## Step 7: Report

```
MCP INSTALLATION COMPLETE
═════════════════════════
  Installed:  X MCPs
  Skipped:    Y MCPs (already installed)
  Failed:     Z MCPs (runtime missing or package not found)
  Pending auth: W MCPs (credentials needed)

  Details:
    ✓ filesystem        — configured, no auth needed
    ✓ calculator        — configured, no auth needed
    ✓ git               — configured, no auth needed
    ⚠ google-workspace  — configured, awaiting OAuth2 credentials
    ✗ pandoc            — uvx not found, install uv first
    
  Next steps:
    - Set GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET for google-workspace
    - Install uv (curl -LsSf https://astral.sh/uv/install.sh | sh) for pandoc
```

## Current Request

$ARGUMENTS
