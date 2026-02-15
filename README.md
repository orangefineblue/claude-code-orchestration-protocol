# Claude Code Orchestration Protocol

A slash command for Claude Code (Opus 4.6) that turns your session into a **zero-read orchestrator** with iterative quality control loops.

## The Problem

When you give Claude Code a complex task, it reads files, reasons, writes code, and runs out of context window before finishing. At 60-70% token usage, output quality degrades (context rot). You lose work, have to re-explain, and start over.

## The Solution

This protocol forces Claude Code to operate as a **pure orchestrator**. It never reads files or reasons about content itself. Instead, it:

1. **Plans** the work in a plan file (with full absolute paths for session recovery)
2. **Delegates** everything to focused agents (each with their own token pool)
3. **Quality controls** every agent output through an iterative loop:
   - Task Agent produces output
   - QC Agent **hunts** for problems (not just validates)
   - Fix Agent applies fixes using **additive improvement** (never drops content)
   - Loop repeats (max 2 iterations, then flags remaining issues to you)
4. **Hands over** cleanly when tokens run low, so a fresh session can pick up exactly where you left off

## Key Features

- **Zero-read orchestrator**: Main session never reads source files. Agents do all the heavy lifting.
- **Iterative QC loops**: Every substantive output gets Task -> QC -> Fix -> Verify. QC agents are instructed to hunt for what's wrong, not confirm what's right.
- **Enhance without regression**: Fix agents must preserve everything from the original plus add improvements. Dropping details while "improving" is the most common Claude failure mode.
- **Structured agent output**: Every agent writes a `## Summary` header that the orchestrator reads. Full output stays on disk for downstream agents.
- **Token-aware handover**: At 60% usage, warns you. At 70%, stops and writes a handover prompt with full absolute paths so the next session resumes seamlessly.
- **Self-contained agent prompts**: Agents don't see conversation history, so every prompt includes full context, paths, and success criteria.

## Installation

Copy `orchestrate.md` to your Claude Code commands directory:

```bash
cp orchestrate.md ~/.claude/commands/orchestrate.md
```

## Usage

Instead of giving Claude Code a task directly, prefix it with `/orchestrate`:

```
/orchestrate fix the authentication bug across the login flow
```

```
/orchestrate analyze all 15 papers in the literature folder and create a comparison table
```

```
/orchestrate refactor the payment module to use the new API, update tests, and document changes
```

Claude Code will then plan the work, delegate to agents, QC everything, and hand over cleanly if it runs low on tokens.

## Status

This is brand new. I just started using it and wanted to share it with the community. Built with the help of Claude Code itself, informed by research across community repos and official Anthropic docs.

If you have feedback on how to make it better, please open an issue or PR. I'd love to learn from others who are thinking about these patterns.

## Research

This protocol was informed by patterns from:
- Official Anthropic subagent and agent teams documentation
- Community repos including multi-agent-shogun, claude_code_agent_farm, voronerd, nexus-hr, and margelo orchestrator patterns
- Real-world experience with context rot, quality regression, and session recovery failures

## License

MIT
