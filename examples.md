# Examples: Investigation Workflows

## Example 1: Security Vulnerability Investigation

**User Request:** "Investigate whether our API is vulnerable to timing attacks on the authentication endpoint"

### Phase 1: Scope & Hypothesis
```
Create task_plan.md:
- Hypothesis: "The /auth/login endpoint uses non-constant-time password comparison"
- Research threads: code analysis, timing attack theory, framework defaults

Create findings.md, progress.md from templates.
Create research/ folder.
```

### Phase 2: Parallel Research via Subagents
```
Launch 3 subagents in parallel:

Task(subagent_type="Explore", model="haiku"):
  "Search the codebase for password comparison logic. Look in auth/,
   middleware/, and any files containing 'compare', 'verify', or 'bcrypt'.
   Write findings to research/code-analysis.md, then return a brief summary."

Task(subagent_type="general-purpose"):
  "Research timing attacks on authentication endpoints. What makes code
   vulnerable? What are the standard mitigations?
   Write findings to research/timing-attack-theory.md with confidence levels."

Task(subagent_type="general-purpose", model="haiku"):
  "Research whether Express.js/Node.js crypto.timingSafeEqual is used
   by default in popular auth libraries (passport, bcrypt, argon2).
   Write findings to research/framework-defaults.md."
```

### Phase 3: Synthesis
```
Read all research/ files and TaskOutput results.

Save to findings.md:
  | Finding | Source | Confidence |
  | bcrypt.compare is constant-time | research/framework-defaults.md | high |
  | Custom token check on line 45 uses === | research/code-analysis.md | high |
  | === is vulnerable to timing attacks | research/timing-attack-theory.md | high |

Update progress.md with Phase 2 actions and Reboot Check.

Re-read task_plan.md → update hypothesis:
  "The password check is safe (bcrypt), but the API token check is vulnerable"
```

### Phase 4: Report
```
Read progress.md, task_plan.md, findings.md to refresh context.
Write investigation-report.md covering:
- Task: timing attack vulnerability assessment
- Finding: API token comparison vulnerable, password check safe
- Action items: replace === with crypto.timingSafeEqual on auth.ts:45
```

---

## Example 2: Performance Degradation Investigation

**User Request:** "Our app became 3x slower after the last deploy. Investigate why."

### Phase 1: Scope & Hypothesis
```
task_plan.md hypotheses:
1. "A new dependency introduced a performance regression"
2. "A database query changed to be less efficient"
3. "A caching layer was accidentally disabled"
```

### Phase 2: Multi-threaded Research
```
Launch 4 subagents:

Task(subagent_type="Bash"):
  "Run 'git log --oneline HEAD~10..HEAD' and 'git diff HEAD~1 -- package.json'
   to identify what changed in the last deploy. Write results to
   research/deploy-changes.md, then return the commit list."

Task(subagent_type="Explore", model="haiku"):
  "Search for files modified in the last commit that contain database queries.
   Look for changes to SQL, ORM calls, or query builders. Write findings to
   research/db-queries.md."

Task(subagent_type="Explore", model="haiku"):
  "Search for any changes to caching configuration, Redis connections, or
   cache invalidation logic in the last 5 commits. Write findings to
   research/cache-config.md."

Task(subagent_type="general-purpose"):
  "Research common causes of 3x performance degradation in Node.js/Express
   applications after deploys. Focus on: N+1 queries, missing indexes,
   disabled caching, synchronous operations. Write to research/perf-patterns.md."
```

### Phase 3: Cross-reference & Follow-up
```
Read all research/ files.

Subagent 2 found: "New query in users.ts joins 3 tables without index"
Subagent 3 found: "Redis TTL changed from 3600 to 10 seconds"

Save to findings.md:
  ## Key Findings
  | 3-table join without index | research/db-queries.md | high |
  | Redis TTL reduced from 1hr to 10s | research/cache-config.md | high |

  ## Contradictions
  | perf-patterns says caching is #1 cause | code shows BOTH query and cache issues |

Update progress.md. Spawn follow-up:

Task(subagent_type="Explore"):
  "Analyze the query at users.ts:89. What tables are joined? Read the
   migration files to check for indexes. Write to research/query-analysis.md."
```

---

## Example 3: Context Recovery After Compaction

When your context is compacted or you start a new session mid-investigation:

```
1. Read progress.md — the Reboot Check table tells you:
   | Current phase | Phase 3: Analysis |
   | Goal | Investigate API timing vulnerability |
   | Key findings so far | See findings.md |
   | Next action | Synthesize research/ files |

2. Read task_plan.md — see full phase structure and current status
3. Read findings.md — see what you've learned so far
4. Continue from where you left off
```

This ensures zero knowledge loss across context boundaries.

---

## Example 4: Error Recovery During Investigation

When research hits a dead end, log it and pivot:

```
Subagent returns: "403 Forbidden when fetching the API docs"

Update task_plan.md:
  ## Errors Encountered
  | 403 on docs.example.com | 1 | Try cached version or alternative source |

Update progress.md with the error.

Launch new subagent with different approach:
Task(subagent_type="general-purpose"):
  "Search for 'example.com API documentation' using web search.
   Look for cached versions, community guides, or GitHub repos
   that document this API. Write findings to research/api-docs-alt.md."
```

Never retry the same failing approach. Mutate your strategy.

---

## Example 5: Subagent Prompt Patterns

### Good: Self-contained with subagent_type and output file
```
Task(subagent_type="general-purpose", model="haiku"):
  "Research the current state of WebAssembly support in major browsers
  (Chrome, Firefox, Safari, Edge) as of 2026. For each browser, report:
  - WASM version supported
  - Any notable limitations
  - Source URL for the information
  Write results to research/wasm-support.md, then return a markdown table summary."
```

### Bad: References context the subagent can't see
```
"Look into that browser issue we found earlier and check if it's related
to the thing in the config file."
```

The subagent has NO access to your conversation. Every prompt must be fully self-contained.
