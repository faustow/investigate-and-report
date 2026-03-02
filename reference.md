# Reference: Context Engineering Principles

Principles drawn from Manus AI's context engineering work, Anthropic's agent guidance, and production agent research (2025-2026).

> "Markdown is my 'working memory' on disk. Since I process information iteratively and my active context has limits, Markdown files serve as scratch pads for notes, checkpoints for progress, building blocks for final deliverables." — Manus AI

## The 7 Core Principles

### Principle 1: Design Around KV-Cache

> "KV-cache hit rate is THE single most important metric for production AI agents."

- Cached tokens cost ~10x less than uncached tokens
- Keep prompt prefixes stable (single-token change invalidates cache)
- Avoid timestamps or dynamic content in system prompts
- Make context append-only with deterministic serialization

### Principle 2: Mask, Don't Remove

Don't dynamically remove tools (breaks KV-cache). Use logit masking or consistent tool sets instead.

### Principle 3: Filesystem as External Memory

```
Context Window = RAM (volatile, limited)
Filesystem = Disk (persistent, unlimited)
```

- Write anything important to files — context will be compressed or lost
- Keep URLs and file paths even when dropping content (preserve pointers)
- Three-file working memory: roadmap (`task_plan.md`), scratch pad (`findings.md`), checkpoint (`progress.md`)

### Principle 4: Targeted Attention Manipulation

Re-read planning files strategically to push goals into the model's recent attention span.

**When to re-read (targeted approach):**
- Starting a new phase
- Before major decisions
- After completing a batch of subagent work
- Every ~10 tool calls

**When NOT to re-read (avoid the todo.md trap):**
- Before every single tool call (production agents found this wasted ~33% of actions)
- Right after writing the file (content is still in context)

### Principle 5: Checkpoint for Recovery

> "Progress files are your insurance against context loss."

After each phase, update `progress.md` with:
- Actions taken and files modified
- Current state of the Reboot Check table
- Key decisions made

This ensures zero knowledge loss across context compaction, session boundaries, or crashes.

### Principle 6: Keep the Wrong Stuff In

> "Leave the wrong turns in the context."

Failed actions with stack traces let the model implicitly update beliefs. Error recovery is "one of the clearest signals of TRUE agentic behavior."

### Principle 7: Vary Your Patterns

> "Uniformity breeds fragility."

Repetitive action-observation pairs cause drift and hallucination. Introduce controlled variation — vary phrasings, switch strategies, recalibrate on repetitive tasks.

---

## Context Engineering Strategies

### Strategy 1: Context Reduction (Compaction)

- Apply compaction to stale (older) tool results
- Keep recent results full to guide the next decision
- Summarize when compaction reaches diminishing returns

### Strategy 2: Context Isolation (Sub-agents)

Sub-agents are the primary scaling mechanism for complex tasks:

```
┌──────────────────────────────┐
│     ORCHESTRATOR (you)       │
│  └─ Maintains the plan       │
│  └─ Delegates to sub-agents  │
│  └─ Synthesizes results      │
├──────────────────────────────┤
│     SUB-AGENTS               │
│  └─ Perform focused tasks    │
│  └─ Have fresh context       │
│  └─ Return structured results│
│  └─ Cannot spawn subagents   │
└──────────────────────────────┘
```

**Key insight:** Early agent designs updated todo.md on every turn, but this wasted ~33% of actions on bookkeeping. A planner-agent architecture — where the orchestrator delegates to focused sub-agents with clean context windows — is far more efficient.

### Strategy 3: Progressive Disclosure

- Load reference material only when needed (not all upfront)
- Use `glob` and `grep` for just-in-time information retrieval
- Store full results in files, reference them by path

---

## Key Quotes

> "Context window = RAM. Filesystem = Disk. Anything important gets written to disk."

> "if action_failed: next_action != same_action"

> "Error recovery is one of the clearest signals of TRUE agentic behavior."

> "Removing complexity yields more gains than adding it."

---

## Sources

- Manus: [Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
- Anthropic: "Effective Context Engineering for AI Agents" (2025)
- Lance Martin: Manus architecture analysis (October 2025)
- Phil Schmid: "Context Engineering Part 2" (December 2025)
