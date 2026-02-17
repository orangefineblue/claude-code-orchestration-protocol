# Cascade Pipeline Design — Detailed Reference

This file contains the full cascade pipeline patterns referenced by the orchestrate skill SKILL.md.

## CASCADE PATTERN [USE WHEN POSSIBLE]

Instead of the orchestrator waiting for each agent and launching the next, design agents to chain when safe.

## Pattern A — Orchestrator-Driven (default)

Orchestrator launches Agent 1 → waits for notification → reads summary → launches Agent 2 → ...
Use when: next agent's prompt depends on previous agent's output (e.g., QC needs to know what was produced)

## Pattern B — Self-Chaining

Agent 1 writes output AND launches Agent 2 as a sub-task before finishing.
Use when: the chain is predetermined and Agent 2's prompt can be fully specified in advance.
HOW: Include in Agent 1's prompt: "After completing your primary task, launch a Task agent with the following prompt: [full QC prompt]"
CAUTION: Only chain 2 levels deep. Deeper chains risk context rot in the sub-agents.

## Pattern C — Parallel Independent

Launch Agents 1, 2, 3 simultaneously (all independent).
Wait for all completion notifications.
Read all summaries.
Launch dependent Agent 4.
ZERO polling during the parallel wait.

## Choosing a Pattern

If all agent prompts can be written at plan time (no dependency on previous output content), use Pattern B or C. Otherwise, use Pattern A with zero-poll waiting.

## General Cascading Flow

```
PARALLEL: Research/Analysis agents (read sources, write findings to disk)
    ↓ all complete (orchestrator reads ONLY summaries)
SEQUENTIAL: Implementation agent (reads findings + sources, writes output)
    ↓ complete (orchestrator reads ONLY summary)
SEQUENTIAL: QC agent (reads baseline + output, writes validation report)
    ↓ pass/fail (orchestrator reads ONLY verdict)
CONDITIONAL: Fix agent (only if QC fails) → re-run QC
    ↓ pass
SEQUENTIAL: Review agent (reads validated output, writes final review)
```

## Rules

- Parallel agents: launch together when independent
- Sequential agents: wait for dependency before launching
- Every agent writes to disk with ## Summary header
- Agent prompts include FULL ABSOLUTE PATHS to all input and output files
- Agent prompts are SELF-CONTAINED (full context, no assumed conversation history)
- Orchestrator reads ONLY the ## Summary section of each agent's output
