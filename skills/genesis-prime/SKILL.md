---
name: genesis-prime
description: Prime the current session with relevant knowledge from the Genesis knowledge base. Searches learnings, patterns, decisions, and gotchas matching the current task context and injects them into the conversation.
when_to_use: At session start, when resuming work, or before starting a new task. Automatically triggered by SessionStart hook.
argument-hint: "[context] — e.g., 'marketing campaign' or 'data analysis' or blank for auto-detect"
allowed-tools: "Read Grep Glob"
user-invocable: true
---

# Knowledge Priming Protocol

Context: **$ARGUMENTS**

## Step 1: Detect Context

If no explicit context given, infer from:
- Current working directory name
- Recent handoff (if exists)
- Files in the current directory

```
Latest handoff: !`ls -t ~/.claude/handoffs/*/*.yaml 2>/dev/null | head -1`
```

## Step 2: Search Knowledge Base

Search all JSONL files for entries matching the context:

```
Knowledge base stats:
  Learnings: !`wc -l < ~/.claude/knowledge/learnings.jsonl 2>/dev/null || echo 0`
  Decisions: !`wc -l < ~/.claude/knowledge/decisions.jsonl 2>/dev/null || echo 0`
  Patterns:  !`wc -l < ~/.claude/knowledge/patterns.jsonl 2>/dev/null || echo 0`
  Gotchas:   !`wc -l < ~/.claude/knowledge/gotchas.jsonl 2>/dev/null || echo 0`
  Debug:     !`wc -l < ~/.claude/knowledge/debug-insights.jsonl 2>/dev/null || echo 0`
```

For each file, search for entries matching:
1. **Tags** matching the context keywords
2. **Affected files** matching current directory patterns
3. **High confidence** entries (prioritize HIGH over MEDIUM over LOW)
4. **High usage** entries (frequently encountered = likely relevant)

## Step 3: Rank & Filter

Score each matching entry:
- +3 for tag match
- +2 for affected_files match
- +2 for HIGH confidence
- +1 for MEDIUM confidence
- +1 per usage_count (capped at +5)
- +1 per helpful_count (capped at +3)

Take top 10 entries by score. Skip entries marked `"compounded": true` (their knowledge is already in rules/skills).

## Step 4: Inject

Format the primed knowledge as a concise brief:

```markdown
## Primed Knowledge

### Relevant Patterns
- {pattern fact} (confidence: {level}, used {n} times)

### Watch Out (Gotchas)
- {gotcha fact}

### Previous Decisions
- {decision} — Why: {rationale}

### Debug Insights
- {insight}
```

Keep the total injection under 500 tokens — priming should inform, not overwhelm.

## Step 5: Track Helpfulness

After the session, if any primed knowledge was actually useful:
- Increment `helpful_count` on those entries
- This improves future priming accuracy

## Auto-Priming (via Hook)

When integrated as a SessionStart hook, the priming runs automatically:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|resume|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "bash -c 'kb_total=$(cat ~/.claude/knowledge/*.jsonl 2>/dev/null | wc -l); handoff=$(ls -t ~/.claude/handoffs/*/*.yaml 2>/dev/null | head -1); msg=\"\"; if [ \"$kb_total\" -gt 0 ]; then msg=\"Knowledge base: $kb_total entries available. Use /genesis-prime to load relevant context.\"; fi; if [ -n \"$handoff\" ]; then msg=\"$msg Previous handoff: $handoff\"; fi; if [ -n \"$msg\" ]; then echo \"{\\\"additionalContext\\\": \\\"$msg\\\"}\"; fi'"
      }]
    }]
  }
}
```
