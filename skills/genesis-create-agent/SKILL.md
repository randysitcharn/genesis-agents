---
name: genesis-create-agent
description: Create a new Claude Code agent with proper configuration, tools, permissions, and supporting skills. Part of the Genesis meta-agent ecosystem.
when_to_use: When Genesis needs to create a new specialized agent, or user directly asks to create an agent.
argument-hint: "[agent purpose] — e.g., 'PR reviewer with security focus'"
allowed-tools: "Read Write Edit Bash Grep Glob Agent WebSearch WebFetch"
user-invocable: false
---

# Agent Creation Protocol

Create a new agent based on: **$ARGUMENTS**

## Step 1: Requirements Analysis

Determine from the request:
- **Mission**: What single responsibility does this agent serve?
- **Domain**: What area of expertise is needed?
- **Autonomy Level**: How much independence? (research-only vs full builder)
- **Model Choice**: Power (opus) vs Speed (haiku) vs Balance (sonnet)?
- **Isolation**: Does it need its own worktree?

## Step 2: Agent Design

Choose the appropriate template and customize:

### Research Agent (read-only, fast)
```yaml
model: claude-sonnet-4-6  # or haiku for simple lookups
tools: Read Grep Glob WebSearch WebFetch
permissions:
  defaultMode: plan
```

### Builder Agent (full access, careful)
```yaml
model: claude-opus-4-6
tools: Read Write Edit Bash Grep Glob
permissions:
  defaultMode: plan  # require plan approval
```

### Monitor Agent (lightweight, recurring)
```yaml
model: claude-haiku-4-5
tools: Read Bash Grep
```

### Reviewer Agent (read + analyze)
```yaml
model: claude-opus-4-6
tools: Read Grep Glob
memory: true  # learn from past reviews
```

## Step 3: Write agent.md

Create the file at `~/.claude/agents/{name}/agent.md` with:
- YAML frontmatter (name, description, model, tools, skills, permissions, memory)
- Markdown body with role definition, operating principles, and workflow

**Critical rules for agent.md:**
- Description must be specific enough for Claude to auto-delegate
- Tools list should be minimal (least privilege)
- Skills should be pre-created or created alongside
- Memory enabled only if the agent benefits from cross-session learning

## Step 4: Create Supporting Skills

For each agent, create 2-4 core skills:
1. A **primary skill** matching the agent's main workflow
2. A **verification skill** for quality checks
3. Optional **utility skills** for common sub-tasks

Use `/genesis-create-skill` for each.

## Step 5: Configure MCP (if needed)

If the agent needs external tools:
1. Identify required MCP servers
2. Use `/genesis-discover-mcp` to find and configure them
3. Add tool permissions to the agent's allowed-tools

## Step 6: Connect to Ecosystem

After creating the agent, run `/genesis-connect {name}` to:
1. Add it to the agent registry in `~/.claude/agents.md`
2. Define its delegation chain (who it reports to, who it can delegate to)
3. Generate its agent card (capabilities, inputs, outputs, limitations)
4. Update the communication topology
5. Create its workspace directory

This step is MANDATORY — an unconnected agent is invisible to the ecosystem.

## Step 7: Verify

1. Check file exists: `ls ~/.claude/agents/{name}/agent.md`
2. Check skills exist: `ls ~/.claude/skills/{name}-*/SKILL.md`
3. Validate YAML frontmatter syntax
4. Check agent is in `~/.claude/agents.md` registry
5. Test invoke: describe a matching task to see if Claude routes to the agent

## Output Format

After creation, report:
```
Agent: {name}
  Location: ~/.claude/agents/{name}/agent.md
  Model: {model}
  Tools: {tools}
  Skills: {list}
  MCPs: {list or "none"}
  Memory: {yes/no}
  Connected: {domain} — reports to {list}, delegates to {list}
  
Ready to use. Trigger by: {describe matching task}
```
