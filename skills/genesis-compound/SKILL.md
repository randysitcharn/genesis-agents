---
name: genesis-compound
description: Promote frequently-occurring learnings into permanent rules, skills, or agent updates. The "compound interest" mechanism — ephemeral knowledge becomes durable capabilities. Inspired by CC-v3's compound-learnings and phantom's evolution pipeline.
when_to_use: When the knowledge base has accumulated enough learnings (10+), or when a learning has usage_count >= 3, or on weekly tech maintenance.
argument-hint: "[scope] — e.g., 'all' or 'patterns only' or 'for project X'"
allowed-tools: "Read Write Edit Bash Grep Glob"
---

# Compound Learnings Protocol

Scope: **$ARGUMENTS**

## Phase 1: Knowledge Audit

Read all knowledge base files and build a frequency map:

```
Learnings: !`cat ~/.claude/knowledge/learnings.jsonl 2>/dev/null | wc -l`
Decisions: !`cat ~/.claude/knowledge/decisions.jsonl 2>/dev/null | wc -l`
Patterns: !`cat ~/.claude/knowledge/patterns.jsonl 2>/dev/null | wc -l`
Gotchas: !`cat ~/.claude/knowledge/gotchas.jsonl 2>/dev/null | wc -l`
Debug: !`cat ~/.claude/knowledge/debug-insights.jsonl 2>/dev/null | wc -l`
```

For each entry, check:
- `usage_count` — how often this learning was encountered
- `helpful_count` — how often it was useful when primed
- `tags` — cluster related learnings
- `affected_files` — find file-scoped patterns

## Phase 2: Pattern Detection

Group learnings by similarity and frequency:

### Frequency Threshold
- **usage_count >= 3** → Candidate for permanent rule
- **usage_count >= 5** → Strong candidate for skill creation
- **Same tag cluster with 3+ entries** → Candidate for agent specialization

### Clustering
Group by:
1. **Same affected files** → Path-scoped rule (`.claude/rules/`)
2. **Same tags** → Domain skill or agent update
3. **Same type** → Systematic improvement

## Phase 3: Promotion Decisions

For each cluster that meets threshold, decide the target:

| Signal | Promote To | Location |
|--------|-----------|----------|
| Recurring pattern in specific files/areas | **Path-scoped rule** | `.claude/rules/{topic}.md` |
| Recurring workflow or process | **New skill** | `~/.claude/skills/{name}/SKILL.md` |
| Recurring correction about agent behavior | **Agent update** | Update agent.md description/instructions |
| Recurring gotcha about a tool/API | **CLAUDE.md entry** | Append to project CLAUDE.md |
| Recurring debugging pattern | **Debug skill** | `~/.claude/skills/debug-{domain}/SKILL.md` |

## Phase 4: Invariant Checks (Safety)

Before writing ANY promotion, verify:

### Mandatory Checks (hard fail → abort)
- [ ] **I1**: No secrets, tokens, or credentials in output
- [ ] **I2**: No existing rule/skill contradicted without explicit override
- [ ] **I3**: Promoted content is actionable (not vague)
- [ ] **I4**: No single file grows by more than 50 lines

### Warning Checks (soft fail → log warning)
- [ ] **I5**: Near-duplicate of existing rule (>80% overlap)
- [ ] **I6**: Learning source is LOW confidence
- [ ] **I7**: Affected files no longer exist

If any hard check fails, log to `~/.claude/knowledge/compound-failures.jsonl` and skip.

## Phase 5: Execute Promotions

For each approved promotion:

### Create Path-Scoped Rule
```markdown
---
paths:
  - "{affected_file_glob}"
---

# {Rule Title}

{Imperative instruction derived from learnings}

**Why:** {Aggregated rationale from source learnings}
```
Write to `.claude/rules/{topic}.md`

### Create New Skill
Use `/genesis-create-skill` with the accumulated knowledge as input.

### Update Agent
Read the target agent.md, append the learning to its instructions section.

### Update CLAUDE.md
Append a concise line under the appropriate section.

## Phase 6: Mark Compounded

After successful promotion:
1. Add `"compounded": true` and `"compounded_to": "{target path}"` to each source learning
2. Do NOT delete the source — it serves as provenance
3. Log the promotion to `~/.claude/knowledge/compound-log.jsonl`:

```jsonl
{
  "date": "{ISO date}",
  "source_ids": ["id1", "id2", "id3"],
  "target_type": "rule|skill|agent|claudemd",
  "target_path": "{path}",
  "summary": "{what was promoted}"
}
```

## Phase 7: Report

```markdown
## Compound Report — {date}

### Knowledge Base Stats
- Total entries: {n}
- Eligible for compounding (usage >= 3): {n}
- Already compounded: {n}

### Promotions Made
| Source IDs | Target | Path | Summary |
|-----------|--------|------|---------|
| {ids} | {type} | {path} | {summary} |

### Skipped (invariant failures)
| IDs | Reason |
|-----|--------|
| {ids} | {reason} |

### Recommendations
- {Next steps or maintenance suggestions}
```

## Automation

Schedule weekly compounding:
```
CronCreate: 0 10 * * 0 — "/genesis-compound all"
```
