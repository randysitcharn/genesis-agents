---
name: genesis-handoff
description: Create and resume session handoffs for continuity across sessions and compactions. Captures goal, progress, decisions, and next steps in structured YAML. Inspired by Continuous-Claude-v3's handoff protocol.
when_to_use: At session end, before context compaction, when resuming work, or when switching between tasks. Also triggered by PreCompact hook.
argument-hint: "[create|resume] [context] — e.g., 'create building auth module' or 'resume' or 'resume handoff-2024-04-16'"
allowed-tools: "Read Write Edit Bash Grep Glob"
---

# Handoff Protocol

Action: **$ARGUMENTS**

## CREATE Handoff

When creating a handoff (end of session, before compaction, or on request):

### Step 1: Extract Session State

Scan the current conversation for:
- **Goal**: What was the user trying to accomplish?
- **Now**: What is the current focus/state right now?
- **Done**: What tasks were completed this session?
- **In Progress**: What is partially done?
- **Blocked**: What couldn't be completed and why?
- **Decisions**: Key choices made and their rationale
- **Findings**: Important discoveries about the codebase
- **What Worked**: Approaches that succeeded
- **What Failed**: Approaches that didn't work (prevent repeating)
- **Next Steps**: Concrete actions for the next session
- **Files Modified**: All files created, edited, or deleted

### Step 2: Write YAML Handoff

```yaml
# Handoff — {brief description}
session: "{session-name or auto}"
date: "{ISO datetime}"
status: "complete|partial|blocked"
outcome: "SUCCEEDED|PARTIAL_PLUS|PARTIAL_MINUS|FAILED"

goal: |
  {What the user wanted to accomplish}

now: |
  {Current state — what was being worked on when session ended}

done_this_session:
  - "{task 1 — completed}"
  - "{task 2 — completed}"

in_progress:
  - task: "{what}"
    state: "{how far along}"
    next_action: "{what to do next}"

blocked:
  - issue: "{what's blocked}"
    reason: "{why}"
    workaround: "{if any}"

decisions:
  - decision: "{what was decided}"
    rationale: "{why}"
    alternatives_rejected: "{what else was considered}"

findings:
  - "{important discovery about the codebase or problem}"

worked:
  - "{approach that succeeded — repeat this}"

failed:
  - "{approach that didn't work — don't repeat this}"

next_steps:
  - "{concrete action 1}"
  - "{concrete action 2}"

files:
  created:
    - "{path}"
  modified:
    - "{path}"
  deleted:
    - "{path}"
```

### Step 3: Save Handoff

Write to: `~/.claude/handoffs/{project-slug}/{date}_{description}.yaml`

```
!`mkdir -p ~/.claude/handoffs 2>/dev/null`
```

### Step 4: Index

Append a one-line entry to `~/.claude/handoffs/INDEX.md`:
```
- [{date} {description}]({filename}) — {outcome} — {goal summary}
```

## RESUME Handoff

When resuming work from a previous session:

### Step 1: Find Latest Handoff

If no specific handoff given, find the most recent:
```bash
ls -t ~/.claude/handoffs/{project-slug}/*.yaml | head -1
```

If a specific reference is given, search by filename or date.

### Step 2: Read and Parse

Read the full YAML handoff. Extract:
- `goal` and `now` → primary context
- `in_progress` → what to continue
- `blocked` → what to address
- `next_steps` → action plan
- `failed` → what NOT to repeat
- `files` → what was touched

### Step 3: Verify Current State

For each item in the handoff, verify it's still accurate:
1. **Files exist?** Check all referenced files still exist
2. **Changes preserved?** Check git status for uncommitted work
3. **Blockers resolved?** Re-check any blocked items
4. **Dependencies met?** Verify any external dependencies

### Step 4: Generate Resume Brief

```markdown
## Resuming: {goal}

### Last Session ({date})
- Outcome: {outcome}
- Progress: {done count}/{total tasks}

### Current State
{now — verified against current filesystem}

### Continuing From
{in_progress items with current status}

### Action Plan
1. {next step 1 — verified still relevant}
2. {next step 2}

### Warnings
- {any discrepancies between handoff and current state}
- {any "failed" approaches to avoid}
```

### Step 5: Prime Knowledge

If a knowledge base exists, prime relevant learnings:
```bash
# Find learnings related to the handoff's affected files
grep -l "affected_files.*{file_pattern}" ~/.claude/knowledge/*.jsonl
```

## AUTO-HANDOFF (Hook Integration)

For automatic handoff creation on compaction or session end:

### PreCompact Hook
```json
{
  "hooks": {
    "PreCompact": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Create a minimal YAML handoff capturing: goal (1 line), now (1 line), done (bullet list), next_steps (bullet list), files modified. Write to ~/.claude/handoffs/{project}/auto-{timestamp}.yaml. Be extremely concise — under 50 lines.",
        "model": "claude-haiku-4-5"
      }]
    }]
  }
}
```

### Stop Hook
```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "If meaningful work was done this session, create a brief YAML handoff at ~/.claude/handoffs/. Skip if session was just Q&A with no file changes.",
        "model": "claude-haiku-4-5"
      }]
    }]
  }
}
```

### SessionStart Hook
```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "resume|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "latest=$(ls -t ~/.claude/handoffs/*/*.yaml 2>/dev/null | head -1); if [ -n \"$latest\" ]; then echo '{\"additionalContext\": \"Previous session handoff available at: '$latest'. Read it to resume context.\"}'; fi"
      }]
    }]
  }
}
```
