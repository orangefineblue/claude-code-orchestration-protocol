# Anti-Polling Rule — Detailed Reference

This file contains the full anti-polling protocol referenced by the orchestrate skill SKILL.md.

## ZERO-POLL WAITING [MANDATORY — HARDEST RULE IN THIS PROTOCOL]

NEVER poll agent progress. NEVER call TaskOutput while an agent is running. NEVER use Bash to tail/read agent output files. NEVER use sleep+check loops.

## Why This Rule Exists

Every poll of TaskOutput returns the FULL agent transcript (often 50-150K tokens) into the orchestrator's context. A single poll can consume 10-20% of the context window. Multiple polls killed a session at 64% tokens with zero useful work done.

## What To Do Instead

1. Launch agent with run_in_background=true
2. Tell the user: "Agent launched. Will process result when done."
3. DO NOTHING. Zero tool calls. Zero bash commands. The system sends automatic notifications when agents complete.
4. When the completion notification arrives, read ONLY the agent's summary file (the one specified in the agent prompt), NOT the TaskOutput.
5. Process the summary and launch the next agent.

## If The User Asks For Status

Say "Agent is still running. The system will notify me when it completes. No action needed from either of us."

## If Agent Seems Stuck (>15 minutes)

Use ONE minimal bash command: `wc -l <output_file_path> 2>/dev/null || echo "NOT YET"`. This costs ~20 tokens vs 100K+ for TaskOutput. Never use TaskOutput for status checks.

## Hard Ban List

- TaskOutput with block=false (returns full transcript)
- TaskOutput with block=true (blocks AND returns full transcript)
- Bash: tail/cat/head on agent output files
- Bash: sleep + any check pattern
- Reading /private/tmp/claude-501/ files directly
