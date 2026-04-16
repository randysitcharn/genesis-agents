# genesis
- **Genesis** (`~/.claude/agents/genesis/agent.md`) - Meta-agent that creates agents, skills, discovers MCPs, schedules tasks, and does tech watch. Trigger: `/genesis`
- Sub-skills: `/genesis-create-agent`, `/genesis-create-skill`, `/genesis-discover-mcp`, `/genesis-schedule`, `/genesis-tech-watch`, `/genesis-self-reflect`, `/genesis-compound`, `/genesis-quality-gate`, `/genesis-handoff`, `/genesis-prime`, `/genesis-status`, `/genesis-catalog`, `/genesis-benchmark`, `/genesis-connect`
- Communication manifest: `~/.claude/agents.md` — agent registry, delegation chains, request/response formats, escalation protocol
- Pre-built agents: `researcher` (Sonnet, any domain), `reviewer` (Opus, adversarial quality), `builder` (Opus, execution), `monitor` (Haiku, fast checks)
- Knowledge base: `~/.claude/knowledge/*.jsonl` — learnings, decisions, patterns, gotchas, debug-insights
- Hooks: SessionStart (KB priming + handoff reminder), Stop (reflect reminder), PreCompact (handoff reminder)
When the user types `/genesis`, invoke the Skill tool with `skill: "genesis"` before doing anything else.

# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.
