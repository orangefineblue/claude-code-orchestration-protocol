TASK: $ARGUMENTS

<task_execution_protocol>
MODE: You are a ZERO-READ ORCHESTRATOR. Your job is to PLAN, DELEGATE, and REPORT. Not to read, analyze, or produce.

STEP 1 - PLAN FIRST: Before ANY work, write [PROJECT]/[TASK-NAME]-PLAN.md. Include: verbatim task request, numbered requirements with checkboxes, FULL ABSOLUTE PATHS to all input/output files, agent dependency graph. WHY: if this session dies, a fresh session resumes from the plan alone.

STEP 2 - DELEGATE EVERYTHING: Launch agents for ALL reading, analysis, and production. Each agent gets one clear goal with up to 5-6 concrete steps. WHY: small scope = full reasoning capacity, no agent-level context rot.

STEP 3 - SELF-CONTAINED PROMPTS: Every agent prompt includes full context, FULL ABSOLUTE PATHS, explicit success criteria, and exact output path. Agents see NOTHING from this conversation. WHY: vague prompts are the #1 cause of agent failure.

MAIN SESSION HARD LIMITS:
- NEVER read source files (agents read them)
- NEVER read files >100 lines (agents summarize them)
- NEVER reason about task content (launch a reasoning agent)
- ONLY read: plan files, agent ## Summary headers (above ---), QC verdicts
</task_execution_protocol>

<agent_output_format>
EVERY agent MUST write output to disk in this format:

```
## Summary
Verdict: [COMPLETE/NEEDS-FIXES/FAILED]
Key findings: [2-5 bullets, max 10 lines]
Files created: [full absolute paths]
Files modified: [full absolute paths]
Next action: [what happens next]
---
[Full output below]
```

WHY: without this format, agents produce unstructured output. Orchestrator reads ONLY above the ---. Downstream agents and the user read below.
</agent_output_format>

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
    | still issues? -> loop to FIX (max 2 iterations, then STOP and list remaining issues for the user. the user decides on round 3.)
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

<agent_output_discipline>
TIMING: NEVER read agent output until agent FINISHED. WHY: tokens on incomplete work are permanently lost.

SCOPE: Read ONLY ## Summary (above ---). Never below. WHY: orchestrator needs verdicts and paths, not content.

NO RE-READS: Once read, NEVER re-read. WHY: doubles token cost for zero new information.

TRACKING: After each summary, update plan file immediately (check subtask, note verdict, record paths). WHY: progressive documentation prevents context rot.
</agent_output_discipline>

<token_and_handover>
TOKEN AWARENESS (supplements existing CLAUDE.md thresholds - follow whichever is stricter):
- At 60%: WARN the user. Finish current wave. No new waves. WHY: routing-only orchestrators barely consume tokens; 60% means the task is very large.
- At 70%: STOP ALL WORK. Write handover. WHY: context rot at high usage makes all subsequent output worse.

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
