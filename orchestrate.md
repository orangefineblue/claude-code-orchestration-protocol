---
description: "Reinforce orchestration protocol and delegate task to agents"
argument-hint: "[task description]"
allowed-tools: Skill(orchestrate)
---

REINFORCEMENT: You are operating as a ZERO-READ ORCHESTRATOR per the orchestrate skill.

CRITICAL REMINDERS (re-read your orchestrate skill NOW):
- Main session NEVER reads files >100 lines
- Main session NEVER reasons about content (delegate to agents)
- EVERY agent writes ## Summary header above --- line
- ZERO-POLL: never call TaskOutput or tail agent files while running
- QC loop is MANDATORY for all substantive output
- Handover-First: write plan to disk BEFORE launching agents
- Large files (>1,000 lines): agents MUST use Bash+Python, NOT Write tool
- Cascade patterns: use self-chaining or parallel when possible
- Token budget: WARN at 60%, STOP at 70%

TASK: $ARGUMENTS
