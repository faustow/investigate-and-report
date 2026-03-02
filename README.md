# investigate-and-report

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that turns Claude into a thorough research agent. It creates a structured investigation plan, spawns parallel subagents to research different threads simultaneously, maintains persistent markdown files as working memory, and produces a final report.

## How it works

When you invoke `/investigate-and-report`, Claude follows a 4-phase workflow:

1. **Scope & Hypothesis** — Defines the investigation goal, forms hypotheses, and identifies research threads
2. **Research** — Launches 3-5+ parallel subagents to investigate different threads simultaneously
3. **Analysis & Synthesis** — Cross-references findings, resolves contradictions, assesses confidence levels
4. **Report** — Writes `investigation-report.md` with findings, what didn't work, and action items

Throughout the investigation, Claude maintains three working memory files:

| File           | Purpose                                                    |
| -------------- | ---------------------------------------------------------- |
| `task_plan.md` | Roadmap — phases, hypotheses, status tracking              |
| `findings.md`  | Scratch pad — key findings, contradictions, open questions |
| `progress.md`  | Checkpoint — enables recovery after context compaction     |

These files act as persistent memory, ensuring the investigation can survive context window limits and pick up where it left off.

## Install

Clone this repo into your Claude Code skills directory:

```bash
git clone https://github.com/faustow/investigate-and-report.git ~/.claude/skills/investigate-and-report
```

## Usage

In any Claude Code session, type:

```
/investigate-and-report
```

Then describe what you want investigated. Claude will ask clarifying questions if needed, then begin the investigation autonomously.

### Example prompts

- "Investigate whether our API is vulnerable to timing attacks on the authentication endpoint"
- "Our app became 3x slower after the last deploy. Investigate why."
- "Research the current state of WebAssembly support across major browsers"
- "Investigate why our CI pipeline takes 45 minutes and identify optimization opportunities"

## File structure

```
investigate-and-report/
├── SKILL.md              # Skill definition and instructions
├── reference.md          # Context engineering principles
├── examples.md           # Detailed workflow examples
└── templates/
    ├── task_plan.md      # Investigation roadmap template
    ├── findings.md       # Research findings template
    └── progress.md       # Progress checkpoint template
```

## How it uses subagents

The skill orchestrates multiple subagent types:

| Type              | Model  | Best for                                            |
| ----------------- | ------ | --------------------------------------------------- |
| `Explore`         | haiku  | Fast codebase search, file discovery, quick lookups |
| `general-purpose` | sonnet | Complex analysis, multi-step research, web searches |
| `Bash`            | sonnet | Terminal commands, git operations, running scripts  |

Subagents write their results to a `research/` folder, which the orchestrator synthesizes into the final report.

## Credits

The context engineering principles in this skill draw from [Manus AI's context engineering research](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus), Anthropic's agent guidance, and production agent patterns from 2025-2026.

## License

[Apache 2.0](LICENSE)
