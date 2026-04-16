---
name: genesis-status
description: Show the complete Genesis ecosystem status — agents, skills, knowledge base stats, recent handoffs, and active schedules. The dashboard for the meta-agent ecosystem.
when_to_use: When the user asks for ecosystem status, what's available, or a Genesis overview.
argument-hint: "[section] — e.g., 'agents' or 'knowledge' or blank for full status"
allowed-tools: "Read Bash Grep Glob"
user-invocable: true
---

# Genesis Ecosystem Status

Section: **$ARGUMENTS**

## Agents
```
!`echo ""; for d in ~/.claude/agents/*/; do name=$(basename "$d"); desc=$(head -5 "$d/agent.md" 2>/dev/null | grep "description:" | sed 's/description: //'); model=$(head -5 "$d/agent.md" 2>/dev/null | grep "model:" | sed 's/model: //'); echo "  $name | $model | $desc"; done`
```

## Skills
```
!`echo ""; for d in ~/.claude/skills/*/; do name=$(basename "$d"); desc=$(head -5 "$d/SKILL.md" 2>/dev/null | grep "description:" | head -1 | sed 's/description: //' | cut -c1-80); echo "  /$name — $desc"; done`
```

## Knowledge Base
```
!`echo ""; for f in ~/.claude/knowledge/*.jsonl; do name=$(basename "$f" .jsonl); count=$(wc -l < "$f" 2>/dev/null || echo 0); echo "  $name: $count entries"; done`
```

## Recent Handoffs
```
!`echo ""; ls -t ~/.claude/handoffs/*/*.yaml 2>/dev/null | head -5 | while read f; do name=$(basename "$f" .yaml); echo "  $name"; done; if [ ! "$(ls ~/.claude/handoffs/*/*.yaml 2>/dev/null)" ]; then echo "  (none yet)"; fi`
```

## Summary

Present a formatted dashboard:

```markdown
## Genesis Ecosystem Dashboard

### Agents ({count})
| Name | Model | Description |
|------|-------|-------------|
| {name} | {model} | {description} |

### Skills ({count})
| Name | Type | Description |
|------|------|-------------|
| /{name} | {genesis/other} | {description} |

### Knowledge Base
| Category | Entries | Last Updated |
|----------|---------|-------------|
| {category} | {count} | {date} |

### Recent Handoffs
| Date | Description | Outcome |
|------|-------------|---------|
| {date} | {desc} | {outcome} |

### Health
- Knowledge base: {empty/growing/mature}
- Last reflection: {date or "never"}
- Last compound: {date or "never"}
- Handoff coverage: {yes/no recent handoff}
```

## Recommendations

Based on ecosystem state, suggest:
- If KB is empty: "Run `/genesis-self-reflect` after your next task"
- If no handoffs: "Use `/genesis-handoff create` before ending session"
- If no recent tech-watch: "Run `/genesis-tech-watch` to stay current"
- If learnings have usage_count >= 3: "Run `/genesis-compound` to promote learnings"
