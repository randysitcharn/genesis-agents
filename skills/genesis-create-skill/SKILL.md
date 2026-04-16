---
name: genesis-create-skill
description: Generate a new Claude Code skill with proper SKILL.md, frontmatter, templates, and supporting files. Part of the Genesis meta-agent ecosystem.
when_to_use: When Genesis needs to create a skill for an agent, or user directly asks to create a skill.
argument-hint: "[skill purpose] [for agent X] — e.g., 'code review checklist for review-agent'"
allowed-tools: "Read Write Edit Bash Grep Glob"
user-invocable: false
---

# Skill Generation Protocol

Create a new skill based on: **$ARGUMENTS**

## Step 1: Skill Design

Determine from the request:
- **Name**: kebab-case, descriptive (e.g., `review-strategy`, `build-campaign`, `analyze-market`)
- **Scope**: User-level (`~/.claude/skills/`) or project-level (`.claude/skills/`)
- **Invocation**: User-invocable (slash command) or agent-only (background knowledge)
- **Context**: Run inline or in fork (subagent)?
- **Tools needed**: What tools should be pre-approved?

## Step 2: Frontmatter Configuration

```yaml
---
name: {skill-name}
description: {precise description — this drives auto-invocation}
when_to_use: {additional trigger context}
argument-hint: "{hint for autocomplete}"
allowed-tools: "{space-separated tool list}"
# Optional:
model: {model override if needed}
context: fork  # if should run in subagent
agent: Explore  # if context: fork, which agent type
paths: "{glob}"  # path-specific activation
disable-model-invocation: true  # manual-only
user-invocable: false  # agent-only
shell: bash
---
```

## Step 3: Skill Body Design Patterns

### Workflow Skill (step-by-step process)
```markdown
# {Title}

Execute the following workflow for: $ARGUMENTS

## Step 1: {Action}
{Instructions with specific commands or tool usage}

## Step 2: {Action}
{Instructions}

## Output
{Expected output format}
```

### Research Skill (gather + analyze)
```markdown
# {Title}

Research: $ARGUMENTS

1. Search for relevant files: use Grep/Glob
2. Read and analyze found content
3. Cross-reference with: !`git log --oneline -10`
4. Synthesize findings

## Report Format
- **Finding**: {what}
- **Evidence**: {where, file:line}
- **Recommendation**: {action}
```

### Template Skill (fill-in-the-blank)
```markdown
# {Title}

Generate from template for: $ARGUMENTS

## Template
{Use $0, $1, $ARGUMENTS for dynamic parts}
{Use !`command` for live data injection}
```

### Guard Skill (validation/check)
```markdown
# {Title}

Verify: $ARGUMENTS

## Checks
- [ ] {Check 1}: {how to verify}
- [ ] {Check 2}: {how to verify}

## On Failure
{What to do if check fails}
```

## Step 4: Supporting Files

Create alongside SKILL.md if needed:
```
{skill-name}/
├── SKILL.md           # Required
├── template.md        # For template-based skills
├── examples/          # Example inputs/outputs
│   └── example-1.md
└── scripts/           # Helper scripts
    └── validate.sh
```

## Step 5: Write Files

1. Create skill directory: `~/.claude/skills/{name}/` or `.claude/skills/{name}/`
2. Write SKILL.md with frontmatter + body
3. Write supporting files if designed
4. Verify: `cat ~/.claude/skills/{name}/SKILL.md`

## Step 6: Verify

1. Check YAML frontmatter parses correctly
2. Confirm `$ARGUMENTS` substitutions are present
3. Verify `allowed-tools` matches what the skill actually uses
4. Test: invoke with sample arguments

## Output Format

After creation, report:
```
Skill: {name}
  Location: {path}/SKILL.md
  Invoke: /{name} [args]  (or "agent-only")
  Tools: {allowed-tools}
  Context: {inline or fork}
  
Test with: /{name} {example args}
```
