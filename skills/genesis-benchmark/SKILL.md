---
name: genesis-benchmark
description: Competitive analysis for agents, skills, or the Genesis ecosystem itself. Searches similar repos/projects, deep-dives their features, builds comparison matrices, identifies gaps, prioritizes improvements, and integrates the best patterns. The "how we built Genesis" process, made repeatable.
when_to_use: When creating or improving an agent/skill and wanting to learn from existing projects. When the user asks to compare, benchmark, find inspiration, analyze competitors, or improve based on what others do.
argument-hint: "[target] [scope] — e.g., 'genesis full', 'PR review agent', 'MCP discovery approaches', 'knowledge management patterns'"
allowed-tools: "Read Write Edit Bash Grep Glob Agent WebSearch WebFetch"
user-invocable: true
---

# Competitive Benchmark Protocol

Systematic process to discover, analyze, compare, and integrate the best patterns from similar projects. This is how Genesis itself was built — now available for any agent or skill.

## Parse Request

From `$ARGUMENTS`, determine:
- **Target**: what to benchmark (an agent, a skill, a capability, or Genesis itself)
- **Scope**: `discover` (find repos only), `analyze` (deep-dive), `compare` (matrix), `gaps` (identify lacunes), `integrate` (propose improvements), or `full` (all phases)

Default scope is `full` if not specified.

## Phase 1: Discovery

**Goal**: Find 5-10 similar projects to learn from.

### Search Strategy

Launch parallel research agents for breadth:

1. **GitHub search** — repos matching the target domain
   - Keywords: `[target] agent`, `[target] claude`, `[target] framework`, `[target] automation`
   - Sort by: stars, recent activity
   - Filters: >10 stars, updated in last 6 months

2. **Ecosystem search** — Claude Code specific
   - `claude code [target] agent`
   - `claude code [target] skill`
   - `.claude/agents [target]`
   - MCP servers related to the domain

3. **Adjacent projects** — not direct competitors but relevant patterns
   - Other AI agent frameworks (LangChain, CrewAI, AutoGen patterns)
   - Non-AI tools that solve the same problem
   - Academic papers or blog posts on the topic

### Discovery Output

For each project found, capture:
```yaml
- name: project-name
  url: github.com/...
  stars: N
  last_updated: date
  nature: one-line description
  relevance: HIGH | MEDIUM | LOW
  why: what makes it interesting for our target
```

Select top 5-8 by relevance for deep analysis.

## Phase 2: Deep Analysis

**Goal**: Understand each project's architecture, features, and design decisions.

For each selected project, launch a research agent (parallel when possible):

### Analysis Checklist

For each project, extract:

1. **Architecture** — how is it structured? What are the core abstractions?
2. **Features** — complete feature list with detail level
3. **Design philosophy** — what trade-offs did they make? Why?
4. **Strengths** — what do they do exceptionally well?
5. **Weaknesses** — what's missing, broken, or poorly designed?
6. **Unique innovations** — what's novel that nobody else does?
7. **Tech stack** — dependencies, infrastructure requirements
8. **Community** — adoption, activity, documentation quality

### Analysis Output

Structured summary per project (keep concise — max 300 words each).

## Phase 3: Comparison Matrix

**Goal**: Feature-by-feature comparison across all projects including our target.

### Matrix Construction

1. **Identify dimensions** — extract all distinct capabilities across all analyzed projects
2. **Group by category** — cluster related features (e.g., "Agent Management", "Learning", "Security")
3. **Rate each project** per feature:
   - `oui` — fully implemented
   - `partiel` — partially implemented
   - `non` — not implemented
   - `oui (detail)` — implemented with noteworthy specifics
4. **Include our target** as the first column for direct comparison

### Matrix Format

One table per category. Each table has:
- Rows = features
- Columns = projects (our target first)
- Cells = implementation status

After each category table, add a **Verdict** paragraph:
- Who leads in this category and why
- What our target is missing
- What our target does uniquely well

## Phase 4: Gap Analysis

**Goal**: Identify what's missing and prioritize improvements.

### Classification

For each gap identified, evaluate:

1. **Relevance** — is this gap actually relevant for our use case?
   - HAUTE: core to the mission
   - MOYENNE: useful enhancement
   - BASSE: nice-to-have
   - NON PERTINENT: doesn't apply (remove from list)

2. **Effort** — how hard to implement?
   - FAIBLE: < 1 skill/file change
   - MOYEN: 1-3 new files
   - ÉLEVÉ: significant architecture change

3. **Inspiration source** — which project does it best and what's their approach?

### Prioritization

Assign priorities using this matrix:

| | High Relevance | Medium Relevance | Low Relevance |
|---|---|---|---|
| **Low Effort** | P0 — do immediately | P1 — do next | P2 — when convenient |
| **Medium Effort** | P1 — do next | P2 — when convenient | P3 — backlog |
| **High Effort** | P2 — plan carefully | P3 — backlog | Skip |

### Gap Output

Table with columns: Priority, Gap, Inspired By, Relevance, Effort, Impact

## Phase 5: Integration Proposals

**Goal**: Concrete proposals for what to build, inspired by the best patterns.

For each P0 and P1 gap:

1. **What to build** — specific files, skills, or changes
2. **Inspired by** — which project and which specific pattern
3. **Adapted how** — what we change to fit our context (not blind copying)
4. **Effort estimate** — files to create/modify
5. **Dependencies** — what needs to exist first

### Integration Output

Present as an ordered action plan:

```
INTEGRATION PLAN — [Target]
═══════════════════════════

Phase 1: [Name] (P0 items)
  1. Create skill X — inspired by [project]'s [pattern]
  2. Update agent Y — add [capability]
  ...

Phase 2: [Name] (P1 items)
  1. ...

Deferred (P2+):
  - Item — reason to defer
```

### User Approval Gate

ALWAYS present the integration plan to the user before executing. Wait for explicit approval or modifications. Never auto-integrate without consent.

## Execution Notes

- **Parallelism**: Phase 1 searches and Phase 2 analyses should use parallel agents for speed
- **Depth vs breadth**: For `full` scope, go deep. For `discover`, stop after Phase 1. For `gaps`, skip to Phase 4 if comparison already exists.
- **Save learnings**: After completing a benchmark, use `/genesis-self-reflect` to capture:
  - Which repos were most valuable
  - Which patterns were adopted
  - What was rejected and why
- **Knowledge base**: Store benchmark results in `~/.claude/knowledge/decisions.jsonl` as architectural decisions

## Current Request

$ARGUMENTS
