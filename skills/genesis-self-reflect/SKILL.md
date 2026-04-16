---
name: genesis-self-reflect
description: Extract learnings from the current session — decisions, corrections, patterns, gotchas, and debugging insights. Stores them in a structured knowledge base for future priming. Inspired by phantom's evolution pipeline and CC-v3's compound learnings.
when_to_use: After completing a task, at session end, or when explicitly asked to reflect. Also useful after failed attempts to capture what went wrong.
argument-hint: "[focus] — e.g., 'this session' or 'last PR' or 'debugging findings'"
allowed-tools: "Read Write Edit Bash Grep Glob"
---

# Self-Reflection Protocol

Reflect on: **$ARGUMENTS**

## Phase 1: Signal Extraction

Scan the current conversation for learning signals. Look for these patterns:

### Corrections (highest value)
- User said "no", "not that", "don't", "wrong", "actually..."
- Failed attempts that were retried differently
- Approaches abandoned after feedback

### Realizations
- "I see", "realized", "the issue was", "turns out"
- Unexpected behavior discovered during work
- Assumptions that proved wrong

### Decisions
- Architecture choices and their rationale
- Tool/library selections with reasoning
- Trade-offs made (and why)

### Patterns Discovered
- Code conventions found in the codebase
- Build/test/deploy workflows that worked
- File organization patterns

### Gotchas
- Surprising behavior of tools or APIs
- Edge cases that caused failures
- Configuration pitfalls

### Debugging Insights
- Root causes found after investigation
- Diagnostic approaches that worked
- Red herrings that wasted time

## Phase 2: Classification & Scoring

For each learning, assign:

```jsonl
{
  "id": "{uuid-short}",
  "type": "CORRECTION|REALIZATION|DECISION|PATTERN|GOTCHA|DEBUG",
  "fact": "{the learning in imperative mood}",
  "why": "{rationale or context}",
  "confidence": "HIGH|MEDIUM|LOW",
  "affected_files": ["path/to/file"],
  "tags": ["domain", "tool", "pattern"],
  "source": "{session|pr|user-feedback}",
  "created": "{ISO date}",
  "usage_count": 0,
  "helpful_count": 0
}
```

### Quality Filters (reject if ANY apply)
1. Is this already obvious from reading the code? → SKIP
2. Is this specific to a one-time task with no reuse? → SKIP
3. Is this already documented in CLAUDE.md? → SKIP
4. Does this contain secrets, tokens, or PII? → SKIP
5. Is this a duplicate of an existing learning? → SKIP (increment usage_count instead)

## Phase 3: Deduplication

Before adding, check existing knowledge base:
```
!`cat ~/.claude/knowledge/patterns.jsonl 2>/dev/null | wc -l` existing patterns
!`cat ~/.claude/knowledge/gotchas.jsonl 2>/dev/null | wc -l` existing gotchas
!`cat ~/.claude/knowledge/decisions.jsonl 2>/dev/null | wc -l` existing decisions
```

For each new learning:
1. Grep existing files for similar facts (fuzzy match on key terms)
2. If duplicate found: increment `usage_count` on existing entry
3. If related but new angle: add as new entry with association link
4. If truly new: append to appropriate JSONL file

## Phase 4: Storage

Append validated learnings to the appropriate JSONL file:

| Type | File |
|------|------|
| CORRECTION, REALIZATION | `~/.claude/knowledge/learnings.jsonl` |
| DECISION | `~/.claude/knowledge/decisions.jsonl` |
| PATTERN | `~/.claude/knowledge/patterns.jsonl` |
| GOTCHA | `~/.claude/knowledge/gotchas.jsonl` |
| DEBUG | `~/.claude/knowledge/debug-insights.jsonl` |

## Phase 5: Report

```markdown
## Self-Reflection Report

### Learnings Extracted: {count}
| Type | Fact | Confidence |
|------|------|------------|
| {type} | {fact} | {confidence} |

### Duplicates Found: {count}
- {fact} (usage_count now: {n})

### Skipped: {count}
- {reason}

### Recommendation
{If any learning has usage_count >= 3, flag for compounding via /genesis-compound}
```

## Phase 6: Memory Update

If significant learnings were found, update Genesis memory:
- Add a summary line to `~/.claude/projects/{project}/memory/MEMORY.md`
- Save detailed findings in a dated memory file

## Automation

To auto-reflect after every session:
```json
{
  "hooks": {
    "Stop": [{
      "matcher": ".*",
      "hooks": [{
        "type": "prompt",
        "prompt": "Extract 1-3 key learnings from this session as JSONL. Focus on corrections, gotchas, and patterns. Write to ~/.claude/knowledge/ files. Be concise.",
        "model": "claude-haiku-4-5"
      }]
    }]
  }
}
```
