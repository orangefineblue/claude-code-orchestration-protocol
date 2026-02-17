# Quality Control Loop — Detailed Reference

This file contains the full QC procedure referenced by the orchestrate skill SKILL.md.

## Loop Structure

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

## QC Agent Rules

- FIND PROBLEMS, not confirm work looks good. "Looks good" catches nothing.
- Hunt: what was missed? Partially implemented? Lost? Contradicts requirements?
- Compare output against requirements AND source material.
- WHY: one-pass work always has undetected issues.

## QC Procedure (mandatory, in order)

1. List every requirement. For each, cite WHERE in output it is met. Uncitable = MISSING.
2. Check for content in original NOT in output (loss detection).
3. Check factual consistency. Flag unverifiable claims.
4. Rate: CRITICAL (requirement unmet), MAJOR (quality gap), MINOR (polish).

## Enhance Without Regression [CRITICAL]

Fix agent output MUST contain everything from the original PLUS improvements. Missing anything = FAILED.
- WHY: Claude's worst failure mode is dropping details while "improving."
- FIX AGENTS: before writing, list every section heading and requirement reference from original. After writing, verify each present. Append to ## Summary: "Regression check: [N/N preserved]". Any 0 = add back before done. WHY: gives verify agent an auditable checklist.

## Agent Failure Protocol

If agent produces no or unusable output, do NOT re-read or debug. Log in plan file (agent, task, failure). Re-launch with refined prompt. Max 1 retry, then report to the user. WHY: debugging wastes orchestrator tokens on content it should never read.
