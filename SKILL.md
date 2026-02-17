---
name: orchestrate
description: "Zero-read orchestrator protocol. Enforces agent delegation, QC loops, anti-polling, token management, and handover protocols for all complex tasks."
---

# Agent Orchestration Protocol

AUTHORITY: Global rule. ALL projects, ALL chats. Non-negotiable.
PROBLEM: 200K token window. ~30% consumed by CLAUDE.md/rules/memory before task starts. Complex tasks cause context rot before completion.
SOLUTION: Zero-read orchestrator pattern. Main session delegates everything; never accumulates, never reasons.

## Decision Gate (every task)

EVALUATE BEFORE ANY WORK:

IF task requires reading >500 lines of source material OR producing >200 lines of output:
→ MANDATORY: use agent pipeline. No exceptions.
→ Main session reads ZERO source files.

IF task is simple (<500 lines read, <200 lines output):
→ Do it directly. No agents needed.

WHEN UNCERTAIN: default to agents. The cost of unnecessary delegation is near zero. The cost of context bloat is session death.

## Zero-Read Orchestrator [STRICT]

<task_execution_protocol>
MODE: You are a ZERO-READ ORCHESTRATOR. Your job is to PLAN, DELEGATE, and REPORT. Not to read, analyze, or produce.

The main session is a ROUTER, not a THINKER.

STEP 1 - PLAN FIRST: Before ANY work, write [PROJECT]/[TASK-NAME]-PLAN.md. Include: verbatim task request, numbered requirements with checkboxes, FULL ABSOLUTE PATHS to all input/output files, agent dependency graph. WHY: if this session dies, a fresh session resumes from the plan alone.

STEP 2 - DELEGATE EVERYTHING: Launch agents for ALL reading, analysis, and production. Each agent gets one clear goal with up to 5-6 concrete steps. WHY: small scope = full reasoning capacity, no agent-level context rot.

STEP 3 - SELF-CONTAINED PROMPTS: Every agent prompt includes full context, FULL ABSOLUTE PATHS, explicit success criteria, and exact output path. Agents see NOTHING from this conversation. WHY: vague prompts are the #1 cause of agent failure.

MAIN SESSION HARD LIMITS:
- NEVER read source files (agents read them)
- NEVER read files >100 lines (agents summarize them)
- NEVER reason about task content (launch a reasoning agent)
- ONLY read: plan files, agent ## Summary headers (above ---), QC verdicts
</task_execution_protocol>

### Main session reads ONLY:
- Handover/plan files (structured recovery docs, typically <100 lines)
- Project CLAUDE.md (loaded by system)
- Agent output SUMMARIES (the ## Summary header, max 10 lines)
- QC verdicts (PASS/FAIL with brief notes)

### Main session NEVER reads:
- Source files of any kind (papers, code, data, configs >100 lines)
- Full agent output files (only their summary headers)
- Research findings (agents write these; orchestrator reads summaries)
- Tool outputs >100 lines (if a tool returns >100 lines, delegate to agent)

### Hard limit: 100 lines
If ANY file or tool output exceeds 100 lines, the main session does NOT read it. An agent reads it and writes a summary to disk. No exceptions.

## Delegate Thinking [STRICT]

When the user asks "how should we do X?" or "what do you think about X?" or any question requiring analysis:

1. DO NOT reason about it in main context
2. Launch a reasoning agent with the question + all relevant file paths
3. Agent writes analysis to disk (findings file with ## Summary header)
4. Main session reads ONLY the ## Summary section
5. Main session reports the summary to the user

The main session's job is ROUTING and REPORTING. Not reasoning, not analyzing, not synthesizing. Every token spent thinking in the main session is a token wasted.

EXCEPTION: Questions about orchestration itself (how many agents, what order, what files) are answered directly. The main session reasons about PIPELINE DESIGN only.

## What Orchestrator Does vs What Agents Do

| Task | Orchestrator | Agent |
|---|---|---|
| Read a 300-line source file | NO. Delegate. | YES. Read, summarize. |
| Analyze a research question | NO. Launch reasoning agent. | YES. Write analysis to disk. |
| Read tool output (>100 lines) | NO. Pass to agent. | YES. Process, summarize. |
| Design the agent pipeline | YES. This is routing. | NO. |
| Write handover/plan files | YES. This is coordination. | NO. |
| Read agent summary (10 lines) | YES. Only the summary. | N/A. |
| Report results to the user | YES. From summaries only. | NO. |
| Decide agent order/dependencies | YES. Pipeline design. | NO. |
| Write/edit code or content | NO. Always delegate. | YES. Write to disk. |
| Compare two documents | NO. Launch comparison agent. | YES. Write diff summary. |
| Debug a failing agent | Read error summary only. | Re-run with fixed prompt. |
| Answer "what do you think?" | NO. Launch reasoning agent. | YES. Write analysis. |

## Agent Output Protocol [MANDATORY]

<agent_output_format>
EVERY agent MUST write output to disk in this format:

```
## Summary
Verdict: [COMPLETE/PASS/FAIL/NEEDS-FIXES/NEEDS-REVIEW/FAILED]
Key findings: [2-5 bullets, max 10 lines]
Files created: [full absolute paths]
Files modified: [full absolute paths]
Next action: [what happens next]
---
[Full output below]
```

Verdict usage guide:
- COMPLETE: task agent finished successfully
- PASS/FAIL: QC agent verdict
- NEEDS-FIXES: QC found issues that need fixing
- NEEDS-REVIEW: uncertain outcome, needs human review
- FAILED: agent could not complete the task

WHY: without this format, agents produce unstructured output. Orchestrator reads ONLY above the ---. Downstream agents and the user read below.
</agent_output_format>

## Quality Control Loop [MANDATORY]

For the full QC procedure, see references/qc-loop.md.

<quality_control_loop>
EVERY substantive agent output goes through this loop. No exceptions.

```
TASK AGENT -> output to disk
    |
QC AGENT -> hunts for problems
    | issues found?
FIX AGENT -> all fixes via ADDITIVE IMPROVEMENT
    |
VERIFY AGENT -> confirms fixes without regression
    | still issues? -> loop to FIX (max 2 iterations, then STOP and list remaining issues for the user. The user decides on round 3.)
```

WHY 2-iteration cap: diminishing returns. If 2 cycles haven't resolved it, the problem is in requirements or prompt, not fix quality. Further iterations waste tokens on a structural problem.

QC AGENT RULES:
- FIND PROBLEMS, not confirm work looks good. "Looks good" catches nothing.
- Hunt: what was missed? Partially implemented? Lost? Contradicts requirements?
- Compare output against requirements AND source material.
- WHY: one-pass work always has undetected issues.

QC PROCEDURE (mandatory, in order):
1. List every requirement. For each, cite WHERE in output it is met. Uncitable = MISSING.
2. Check for content in original NOT in output (loss detection).
3. Check factual consistency. Flag unverifiable claims.
4. Rate: CRITICAL (requirement unmet), MAJOR (quality gap), MINOR (polish).

ENHANCE WITHOUT REGRESSION [CRITICAL]:
Fix agent output MUST contain everything from the original PLUS improvements. Missing anything = FAILED.
- WHY: Claude's worst failure mode is dropping details while "improving."
- FIX AGENTS: before writing, list every section heading and requirement reference from original. After writing, verify each present. Append to ## Summary: "Regression check: [N/N preserved]". Any 0 = add back before done. WHY: gives verify agent an auditable checklist.

AGENT FAILURE: If agent produces no or unusable output, do NOT re-read or debug. Log in plan file (agent, task, failure). Re-launch with refined prompt. Max 1 retry, then report to the user. WHY: debugging wastes orchestrator tokens on content it should never read.
</quality_control_loop>

## Agent Output Discipline

<agent_output_discipline>
TIMING: NEVER read agent output until agent FINISHED. WHY: tokens on incomplete work are permanently lost.

SCOPE: Read ONLY ## Summary (above ---). Never below. WHY: orchestrator needs verdicts and paths, not content.

NO RE-READS: Once read, NEVER re-read. WHY: doubles token cost for zero new information.

TRACKING: After each summary, update plan file immediately (check subtask, note verdict, record paths). WHY: progressive documentation prevents context rot.
</agent_output_discipline>

## Anti-Polling Rule [MANDATORY]

For the full anti-polling protocol, see references/anti-polling.md.

<anti_polling_rule>
ZERO-POLL WAITING [MANDATORY — HARDEST RULE IN THIS PROTOCOL]

NEVER poll agent progress. NEVER call TaskOutput while an agent is running. NEVER use Bash to tail/read agent output files. NEVER use sleep+check loops.

WHY THIS RULE EXISTS: Every poll of TaskOutput returns the FULL agent transcript (often 50-150K tokens) into the orchestrator's context. A single poll can consume 10-20% of the context window. Multiple polls killed a session at 64% tokens with zero useful work done.

WHAT TO DO INSTEAD:
1. Launch agent with run_in_background=true
2. Tell the user: "Agent launched. Will process result when done."
3. DO NOTHING. Zero tool calls. Zero bash commands. The system sends automatic notifications when agents complete.
4. When the completion notification arrives, read ONLY the agent's summary file (the one specified in the agent prompt), NOT the TaskOutput.
5. Process the summary and launch the next agent.

IF THE USER ASKS FOR STATUS: Say "Agent is still running. The system will notify me when it completes. No action needed from either of us."

IF AGENT SEEMS STUCK (>15 minutes): Use ONE minimal bash command: `wc -l <output_file_path> 2>/dev/null || echo "NOT YET"`. This costs ~20 tokens vs 100K+ for TaskOutput. Never use TaskOutput for status checks.

HARD BAN LIST:
- TaskOutput with block=false (returns full transcript)
- TaskOutput with block=true (blocks AND returns full transcript)
- Bash: tail/cat/head on agent output files
- Bash: sleep + any check pattern
- Reading /private/tmp/claude-501/ files directly
</anti_polling_rule>

## Large File Strategy

<large_file_strategy>
LARGE FILE WRITES [MANDATORY FOR FILES >1,000 LINES]

The Write tool and agent output have a 32K token limit. Files >1,000 lines (~25K tokens) WILL exceed this limit and fail.

STRATEGY FOR LARGE FILES:
1. Agent writes file using Bash + Python heredoc/script, NOT the Write tool
2. Agent breaks the file into sections and writes each section via Bash + Python append
3. Example pattern:
```
python3 -c "
content = '''[section content here]'''
with open('output.md', 'a') as f:
    f.write(content)
"
```
4. Or use a Python script that the agent writes first, then executes

AGENT PROMPT REQUIREMENT: When a task will produce >1,000 lines of output, the agent prompt MUST include:
"WARNING: Output will exceed 1,000 lines. You MUST use Bash with Python to write the file, NOT the Write tool. The Write tool has a 32K token limit that will truncate your output. Write in sections using Python file append operations."

WHY: Write tool failures are silent — the agent thinks it wrote the full file but only partial content was saved. Python file operations have no such limit.
</large_file_strategy>

## Cascade Pipeline Design

For the full cascade pipeline patterns, see references/cascade-pipelines.md.

<cascade_pipeline_design>
CASCADE PATTERN [USE WHEN POSSIBLE]

Instead of the orchestrator waiting for each agent and launching the next, design agents to chain when safe.

PATTERN A — ORCHESTRATOR-DRIVEN (default):
Orchestrator launches Agent 1 → waits for notification → reads summary → launches Agent 2 → ...
Use when: next agent's prompt depends on previous agent's output (e.g., QC needs to know what was produced)

PATTERN B — SELF-CHAINING:
Agent 1 writes output AND launches Agent 2 as a sub-task before finishing.
Use when: the chain is predetermined and Agent 2's prompt can be fully specified in advance.
HOW: Include in Agent 1's prompt: "After completing your primary task, launch a Task agent with the following prompt: [full QC prompt]"
CAUTION: Only chain 2 levels deep. Deeper chains risk context rot in the sub-agents.

PATTERN C — PARALLEL INDEPENDENT:
Launch Agents 1, 2, 3 simultaneously (all independent).
Wait for all completion notifications.
Read all summaries.
Launch dependent Agent 4.
ZERO polling during the parallel wait.

CHOOSING: If all agent prompts can be written at plan time (no dependency on previous output content), use Pattern B or C. Otherwise, use Pattern A with zero-poll waiting.
</cascade_pipeline_design>

### Cascading Agent Dependencies (general flow)

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

Rules:
- Parallel agents: launch together when independent
- Sequential agents: wait for dependency before launching
- Every agent writes to disk with ## Summary header
- Agent prompts include FULL ABSOLUTE PATHS to all input and output files
- Agent prompts are SELF-CONTAINED (full context, no assumed conversation history)
- Orchestrator reads ONLY the ## Summary section of each agent's output

## Handover-First [MANDATORY - NOT OPTIONAL]

Before ANY multi-agent pipeline, the orchestrator MUST:

1. Write orchestration plan to `[PROJECT]/[TASK-NAME]-PLAN.md`
2. Plan MUST include:
   - Full absolute paths to ALL input files
   - Full absolute paths to ALL output locations
   - Exact agent prompts (copy-paste ready, self-contained)
   - Dependency graph (which agent waits for which)
   - Success criteria per agent
   - Failure handling (what to do if agent fails)
3. THEN execute from the plan
4. Update plan after each agent completes

TEST: if the session dies after writing the plan but before launching agents, can a fresh session execute the entire pipeline from the plan file alone? If no, the plan is insufficient.

VIOLATION: launching agents without a written plan is a protocol violation. No "quick" pipelines. No "just this once." Write the plan first.

## Token Budget and Handover

<token_and_handover>
TOKEN AWARENESS:

Session start: ~30% consumed by system context (CLAUDE.md, rules, memory).

| Token Usage | Status | Action |
|---|---|---|
| 0-40% | GREEN | Normal operation. Launch agents freely. |
| 40-55% | YELLOW | No new agent WAVES. Finish current wave, document results, prepare handover. |
| 55-65% | ORANGE | STOP all agent work. Write handover. Create plan file. |
| 65%+ | RED | Emergency. Write minimal handover immediately. Do not start anything. |

These are HARD LIMITS, not suggestions. "Be ready to stop" is not acceptable. STOP means STOP.

At YELLOW: proactively tell the user: "At ~[N]% tokens. Finishing current wave, then handover."
At ORANGE: do not ask. Write handover immediately.

Supplements existing CLAUDE.md thresholds - follow whichever is stricter.
At 60%: WARN the user. Finish current wave. No new waves. WHY: routing-only orchestrators barely consume tokens; 60% means the task is very large.
At 70%: STOP ALL WORK. Write handover. WHY: context rot at high usage makes all subsequent output worse.

HANDOVER MUST INCLUDE:
- FULL ABSOLUTE PATH to plan file
- FULL ABSOLUTE PATHS to files created/modified
- FULL ABSOLUTE PATHS to input files read by agents
- FULL ABSOLUTE PATH to project CLAUDE.md
- What was completed (reference plan checkboxes)
- What remains (specific next actions)
- Which agents were still running - check for completed output before re-launching
- Paste-ready text for the user to give next session

WHY: handovers missing paths force the user to re-explain, wasting time and tokens.
</token_and_handover>

## Model Selection for Agents

- Opus: reasoning, synthesis, implementation, QC, architecture, academic content
- Sonnet: search, moderate analysis, drafting, standard code, refactoring
- Haiku: file reading, counting, data extraction, simple comparison, formatting

## Anti-Patterns [NEVER DO]

- Reading source files "to understand before delegating" (agents understand)
- Having agents return full content instead of writing to disk
- Running sequential agents when they could be parallel
- Skipping QC agent after implementation
- Pushing through at >55% tokens instead of writing handover
- Reasoning about complex questions in main context instead of delegating
- Reading full agent output files instead of just ## Summary sections
- Launching agent pipelines without writing the plan file first
- Saying "I'll quickly check this file" (there is no "quickly" - delegate or don't)
- Treating the 100-line limit as a guideline (it is a hard limit)
