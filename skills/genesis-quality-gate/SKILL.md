---
name: genesis-quality-gate
description: Configurable quality gates for agent work — plan review, deliverable review, verification, and validation with iteration limits and escalation. Works across any domain (business, research, design, code, content, strategy). Inspired by metaswarm's gates and claude-pipeline's iteration loops.
when_to_use: When Genesis creates agents that need quality enforcement, or when setting up quality gates for a workflow. Also used directly to review agent output.
argument-hint: "[gate-type] [target] — e.g., 'plan-review ./plan.md' or 'review deliverable' or 'verify requirements met'"
allowed-tools: "Read Write Edit Bash Grep Glob Agent"
---

# Quality Gate Protocol

Gate request: **$ARGUMENTS**

## Gate Types

### 1. Plan Review Gate
**When**: Before execution begins
**Reviewers**: 3 parallel perspectives (Feasibility, Completeness, Scope)
**Pass criteria**: ALL three must PASS
**Max iterations**: 3, then escalate to user

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ Feasibility │     │ Completeness │     │   Scope &   │
│  Reviewer   │     │  Reviewer    │     │  Alignment  │
└──────┬──────┘     └──────┬───────┘     └──────┬──────┘
       │                   │                     │
       └───────────┬───────┘─────────────────────┘
                   ▼
          ALL PASS? ──no──→ Feedback → Revise → Retry (max 3)
              │
             yes
              ▼
          PROCEED
```

**Feasibility Review**:
- [ ] Required resources and tools are available
- [ ] Proposed approach is viable
- [ ] Dependencies are identified and met
- [ ] Scope is realistic for the context

**Completeness Review**:
- [ ] All requirements addressed
- [ ] Edge cases and exceptions considered
- [ ] Risks identified with mitigation
- [ ] Success criteria defined

**Scope Review**:
- [ ] No scope creep beyond original request
- [ ] Changes are focused and minimal
- [ ] No unnecessary additions
- [ ] Aligns with the user's stated goal

### 2. Deliverable Review Gate
**When**: After execution, before delivery
**Mode**: Adversarial — reviewer is ALWAYS a fresh agent (never the author)
**Pass criteria**: No CRITICAL issues, max 2 IMPORTANT issues
**Max iterations**: 5, then escalate to user

**Review Dimensions** (adapt to domain):

| Dimension | Check | Severity |
|-----------|-------|----------|
| **Correctness** | Does it meet the requirements? | CRITICAL |
| **Accuracy** | Are facts, data, and claims verified? | CRITICAL |
| **Completeness** | Are all requested elements present? | CRITICAL |
| **Coherence** | Do all parts align internally? | CRITICAL |
| **Clarity** | Is it understandable by target audience? | IMPORTANT |
| **Structure** | Is it well-organized and navigable? | IMPORTANT |
| **Consistency** | Uniform tone, format, conventions? | SUGGESTION |
| **Polish** | Minor quality improvements? | SUGGESTION |

**Domain-Specific Checks**:

| Domain | Additional Checks |
|--------|-------------------|
| **Business** | Financial accuracy, market validity, competitive awareness |
| **Research** | Methodology rigor, source quality, bias detection |
| **Marketing** | Audience alignment, messaging clarity, brand consistency |
| **Data** | Statistical validity, visualization clarity, conclusions supported |
| **Content** | Readability, factual accuracy, tone appropriateness |
| **Code** | Security, performance, test coverage, maintainability |
| **Strategy** | Risk assessment, stakeholder impact, timeline realism |
| **Design** | User needs, accessibility, visual hierarchy |

**Review Output Format**:
```markdown
## Review — {scope}

**Domain**: {domain}
**Verdict**: PASS | FAIL

### Issues Found
| # | Severity | Location | Issue | Suggested Fix |
|---|----------|----------|-------|---------------|
| 1 | CRITICAL | {where} | {what} | {how to fix} |

### What Looks Good
- {positive observations}

### Iteration {n}/{max}
```

### 3. Verification Gate
**When**: After review passes
**Mode**: Independent verification (check results, don't trust self-reports)
**Pass criteria**: All checks pass, no regressions
**Max iterations**: 10, then escalate

**Protocol**:
1. Re-read the original requirements
2. Check each requirement against the deliverable
3. Verify no existing work was broken or regressed
4. Run any applicable automated checks
5. Confirm the output is ready for the end user

**CRITICAL RULE**: Never trust an agent's claim that "everything is fine." Always verify independently.

### 4. Spec Compliance Gate
**When**: After all other gates pass
**Mode**: Verify deliverable matches original request
**Focus**: Outcomes, not process

**Checklist**:
- [ ] Original request fully satisfied
- [ ] No extra deliverables added (scope creep)
- [ ] No requirements missing
- [ ] Output matches expected format and quality

### 5. Retrospective Gate
**When**: After task completion
**Mode**: Self-reflection for continuous improvement
**Outputs to**: Knowledge base via `/genesis-self-reflect`

**Questions**:
1. What went well? (capture as PATTERN)
2. What went wrong? (capture as GOTCHA)
3. What was surprising? (capture as REALIZATION)
4. What would we do differently? (capture as DECISION)

## Gate Configuration

When creating agents via Genesis, attach gates to their workflow:

```yaml
quality_gates:
  plan_review:
    enabled: true
    max_iterations: 3
    reviewers: [feasibility, completeness, scope]
  deliverable_review:
    enabled: true
    max_iterations: 5
    severity_threshold: IMPORTANT
    fresh_reviewer: true           # always use fresh agent
  verification:
    enabled: true
    max_iterations: 10
  spec_compliance:
    enabled: true
  retrospective:
    enabled: true
    auto_reflect: true
```

## Escalation Protocol

When max iterations reached without passing:

```
Iteration {n}/{max} FAILED

Escalation:
1. Summarize all iteration feedback
2. Identify the persistent blocking issue
3. Present to user with:
   - What was attempted
   - Why it keeps failing
   - Recommended manual intervention
4. DO NOT silently skip the gate
5. DO NOT increase max_iterations automatically
```
