---
name: investigate-and-report
version: "0.2.0"
description: Deep investigation with subagent orchestration and persistent markdown working memory. Spawns parallel research subagents and produces a final investigation-report.md.
user-invocable: true
allowed-tools:
  - Task
  - AskUserQuestion
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - WebFetch
  - WebSearch
hooks:
  Stop:
    - hooks:
        - type: agent
          prompt: |
            Check if an investigation is complete. Try to read task_plan.md.
            If it does not exist, respond: {"ok": true}
            If it exists, check two conditions:
            1. Every phase under the ## Phases section has **Status:** complete
            2. A file named investigation-report.md exists
            If BOTH conditions are true, respond: {"ok": true}
            If either is false, respond: {"ok": false, "reason": "what is incomplete"}
            Output ONLY the JSON object, nothing else.
          model: haiku
---

# Investigate And Report

You are conducting a deep investigation. Prioritize thoroughness over efficiency — launch as many subagents, follow as many threads, and spend as many tool calls as the investigation demands. Use persistent markdown files as working memory — scratch pads for notes, checkpoints for progress, and building blocks for final deliverables.

## Quick Start

1. **Read examples** — Read `${CLAUDE_PLUGIN_ROOT}/examples.md` for investigation workflow patterns
2. **Create `task_plan.md`** — Your roadmap ([template](templates/task_plan.md))
3. **Create `findings.md`** — Your scratch pad ([template](templates/findings.md))
4. **Create `progress.md`** — Your checkpoint ([template](templates/progress.md))
5. **Create `research/` folder** — Each subagent writes to its own file here
6. **Execute phases** — Update plan status and progress log after each phase
7. **Write `investigation-report.md`** — When all phases are complete

All output files go in your project directory. Templates are in `${CLAUDE_PLUGIN_ROOT}/templates/`.

## Core Rules

### 1. Plan First
Create `task_plan.md` before doing anything else. Define your goal, hypotheses, and phases.

### 2. Save Early, Save Often
After every 2 search/fetch operations, save key findings to `findings.md`. Update `progress.md` after each phase with actions taken and files modified. This is your insurance against context loss.

### 3. Re-read Before Decisions
Before major decisions or after completing a subagent batch, re-read `task_plan.md` to keep goals in your attention window.

### 4. Recover Context After Compaction
After context compaction or starting a new session, read `progress.md` first (Reboot Check table), then `task_plan.md`, then `findings.md`. This restores your working state.

### 5. The 3-Strike Error Protocol
Every error goes in the plan file. Escalate on repeated failures:
```
ATTEMPT 1: Diagnose & Fix → Read error, identify root cause, apply targeted fix
ATTEMPT 2: Alternative Approach → Different method, different tool
ATTEMPT 3: Broader Rethink → Question assumptions, search for solutions
AFTER 3 FAILURES: Ask the user for guidance
```

### 6. Update Status After Each Phase
Mark phase status in `task_plan.md`: `pending` → `in_progress` → `complete`. Update `progress.md` with actions taken and files created.

## Context Management

| Situation | Action |
|-----------|--------|
| Starting a new phase | Read plan and findings to re-orient |
| After context compaction / new session | Read `progress.md` → `task_plan.md` → `findings.md` |
| Just wrote a file | Content is still in context — skip re-reading it |
| After completing a subagent batch | Read plan, save synthesized findings |
| Error occurred | Read the relevant file for current state |
| 10+ tool calls since last plan read | Re-read `task_plan.md` |

## Subagent Strategy

Subagents are your primary tool for deep investigation. Do not optimize for token cost — optimize for investigation quality. Launch 3-5+ parallel subagents per research phase. Spawn follow-up subagents whenever you find gaps or contradictions.

### Subagent Types

| `subagent_type` | Model | Best for |
|-----------------|-------|----------|
| `Explore` | haiku | Read-only codebase search, file discovery, quick lookups |
| `general-purpose` | inherits yours | Tasks needing write access, complex analysis, multi-step work |
| `Bash` | inherits yours | Terminal commands, git operations, running scripts |

**Critical constraint:** Subagents cannot spawn other subagents. When delegating, ensure the subagent can complete the work with direct tool calls only.

### When to Spawn
- Parallel research on independent topics (launch 3-5 simultaneously)
- Deep-diving into a specific aspect while you continue with another
- Token-intensive explorations (web searches, large file analysis)
- Tasks that benefit from fresh context

### How to Delegate
1. Write a **self-contained** prompt — the subagent has NO access to your conversation history
2. Include ALL context the subagent needs
3. Specify the exact output format
4. Use `run_in_background: true` for parallel research
5. Use `model: "haiku"` for quick lookups, default (sonnet) for analysis
6. For substantial research, instruct the subagent to write results to `research/<thread-name>.md`

### Result Persistence

| Subagent complexity | Result handling |
|---------------------|----------------|
| Quick lookup (haiku, 3-5 calls) | Return inline — TaskOutput is sufficient |
| Focused research (sonnet, 5-15 calls) | Write to `research/<thread-name>.md` |
| Deep analysis (20+ calls) | Write to `research/<thread-name>.md` |

### Synthesis
After subagents return:
1. Read each result (from TaskOutput or `research/` files)
2. Save key findings to `findings.md` immediately
3. Re-read `task_plan.md` to re-orient
4. Look for contradictions across research threads — log them in `findings.md`
5. Spawn follow-up subagents to resolve contradictions

### Example Subagent Prompt
```
Research [specific topic]. Look for:
- [specific aspect 1]
- [specific aspect 2]

Write your findings to research/[thread-name].md, then return a brief summary.
Include for each finding:
- Finding: [what you found]
- Source: [where you found it]
- Confidence: [high/medium/low]
```

## Final Step: Writing the Report

When all phases are complete, write `investigation-report.md`. Read your planning files to refresh context, then include:

- **Task definition** — What was investigated and why
- **ELI5 summary** — Explain it simply
- **Investigation journey** — The happy path through your phases
- **Negative summary** — What didn't work and why
- **Final summary** — Complete findings and conclusions
- **Action items for user** — Only items truly impossible for you to accomplish yourself. If you COULD do any of them, add them to your plan and continue investigating.

The report must be a working markdown file with a usable table of contents after the title.
