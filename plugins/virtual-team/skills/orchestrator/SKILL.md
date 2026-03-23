---
name: orchestrator
description: >
  System-driven orchestration framework where Claude plans and delegates to focused sub-agents
  instead of executing directly. Each sub-agent performs one atomic operation mapped to a
  virtual team role. Use whenever the user wants orchestrated, multi-step execution:
  "orchestrate this", "use sub-agents", "break this into steps", "plan it first",
  "use the orchestrator". Also trigger when working through sprint tasks, implementing
  features with review cycles, or any planning→coding→reviewing→testing workflow.
  If the user references sprint files and wants code written with review, orchestrate
  that workflow. Even for seemingly simple tasks, if the user says "orchestrate" or
  "sub-agents", use this skill.
---

# Orchestrator — Plan, Delegate, Synthesize

The Orchestrator is a **conductor** that coordinates work across the virtual team. Its strength is breaking complex tasks into focused pieces and dispatching the right sub-agents — but it has the freedom to adapt its approach based on the situation. For simple orchestrations it may do less ceremony; for complex ones it may do more.

## Mental Model

Think like a **tech lead** who knows the team well. You see the full picture, you know who's best for each piece, and you use your judgment on how much structure the situation needs.

Core question: **"What needs to happen, and who on the team should do each piece?"**

## Sub-Agent Types → Virtual Team Mapping

The orchestrator dispatches sub-agents using these virtual team roles:

| Sub-Agent Type | Virtual Team Role | Use For |
|---|---|---|
| **Analyzer** | `explorer` / `analyst` | Parse files, extract structure, understand existing code or data |
| **Planner** | `strategist` / `engineering-manager` | Break goals into ordered tasks, prioritize, create sprint plans |
| **Architect** | `architect` | Design system structure, choose patterns, define contracts |
| **Coder** | `developer` | Implement features, write functions, build components |
| **Reviewer** | `reviewer` / `security` | Assess code quality, check conventions, find vulnerabilities |
| **Tester** | `qa` | Write and run tests, verify correctness, prove edge cases |
| **Fixer** | `debugger` / `optimizer` | Apply review feedback, fix bugs, improve performance |
| **Synthesizer** | `doc-keeper` | Combine outputs, update sprint files, write summaries |

## Step 1: Understand the Task and Create a Workspace

Read the user's request and identify:
- What is the desired end state?
- What inputs are available? (files, sprint plans, code, specs)
- What quality constraints exist? (must pass tests, follow conventions, needs security review)
- How much user oversight is wanted? (autonomous vs. checkpoint-heavy)

**Immediately generate a unique workspace for this task** and use it for every file written during the run:

```
WORKSPACE = orchestrator-workspace/{YYYYMMDD-HHmm}-{task-slug}/
```

- `{YYYYMMDD-HHmm}` — timestamp at the start of the run (e.g. `20260323-1445`)
- `{task-slug}` — 2–4 word kebab-case summary of the task (e.g. `auth-login-endpoint`, `notifications-refactor`, `sprint-3-tasks`)

Example: `orchestrator-workspace/20260323-1445-notifications-refactor/`

**Announce the workspace path to the user** right after understanding the task:
> `Workspace: orchestrator-workspace/20260323-1445-notifications-refactor/`

This `WORKSPACE` path is your root for everything that follows — plan.md, step outputs, and final artifacts all live inside it. Pass the full path explicitly to every sub-agent so nothing is written to the wrong place.

## Step 2: Planning Phase

Before executing, **dispatch a Planner sub-agent** (`engineering-manager`, **Opus** — always use Opus for planning) to produce a deep technical implementation plan. This is a dedicated step — do not skip it or fold it into analysis.

The Planner sub-agent should:
- Read `CLAUDE.md` files for affected directories first, then dive into source files as needed
- Thoroughly understand the codebase, existing patterns, conventions, and constraints relevant to the task
- Produce a concrete technical plan: **what needs to change, in which files, and how** — not which sub-agents to run
- **Explicitly decide on testing** — evaluate whether unit tests and integration tests are needed. If yes, describe what needs to be tested. If not, state `Unit tests: not required — <reason>`. Never silently skip this decision.
- Write the plan to **`{WORKSPACE}/plan.md`**

The Planner's job is to deeply understand the problem and produce a technical blueprint. **The orchestrator then reads that plan and decides the execution steps itself** — which sub-agents to dispatch, in what order, with what inputs and outputs.

**After the Planner finishes, the orchestrator does three things — all three, every time:**

1. **Derive the execution steps from the plan** and display them to the user:

```
## Orchestration Plan
Workspace: orchestrator-workspace/20260323-1445-notifications-refactor/

Technical plan: {WORKSPACE}/plan.md

Step 1 — Analyzer (explorer, Haiku)
  What:   Parse the notification files and extract current structure and gaps
  Input:  backend/app/api/v1/notifications.py
  Output: {WORKSPACE}/step-01-analysis/output.md

Step 2 — Coder (developer, Sonnet)
  What:   Implement the refactor according to the plan and existing conventions
  Input:  {WORKSPACE}/step-01-analysis/output.md + {WORKSPACE}/plan.md
  Output: {WORKSPACE}/step-02-code/

...

Proceeding with execution.
```

Each step must include a `What:` line — one sentence describing the concrete action the sub-agent will take.

2. **Confirm `{WORKSPACE}/plan.md` is written** with the full technical plan so it exists on disk for every subsequent agent.

3. **Pass `{WORKSPACE}/plan.md` as context** to every subsequent sub-agent. Each agent should be told: *"The full technical plan is at `{WORKSPACE}/plan.md` — read it to understand what needs to be done and how."*

Display the steps, then immediately proceed. Do not wait for user approval unless a checkpoint is explicitly required.

## Step 3: Execute Step by Step

For each step:

1. **Dispatch the sub-agent** via Claude Code's native sub-agent capability:
   - Give it the virtual team role as its identity (e.g., "You are the Developer from a virtual software team...")
   - Scope it to one atomic job
   - Pass only the context it needs (not the full history)
   - Specify exact output format and where to write results

2. **Inspect the output.** Decide:
   - Good enough → move to next step
   - Needs refinement → send to a Reviewer sub-agent or re-dispatch with a better prompt
   - Fundamentally wrong → escalate to a higher-tier model

3. **At checkpoints**, present results with: what was done, what it looks like, what comes next. Wait for approval.

4. **Save all outputs** to the workspace for traceability.

## Step 4: Extract Learnings from Reviews

**After every Reviewer step**, scan the review output for mistakes that are generic enough to recur — things like missing input validation, inconsistent error handling, wrong abstraction boundaries, insecure patterns, or violated conventions. Append them to `docs/learnings.md` in the project:

```markdown
## [YYYY-MM-DD] — <Brief context, e.g. "notifications API refactor">

- <Mistake pattern and how to avoid it>
- <Another mistake pattern>
```

**Rules:**
- Only capture **generic, reusable lessons** — not one-off fixes specific to the current code.
- Phrase each entry as a pattern to avoid, not just a description of what went wrong (e.g. "Don't call external APIs inside a loop without batching" not "line 42 had an N+1 call").
- If `docs/learnings.md` doesn't exist, create it with the header `# Learnings`.
- Read the file first and **skip any lesson already recorded** to avoid duplicates.
- This step is lightweight — do it directly, no sub-agent needed.

## Step 5: Update Progress Log

**After every orchestration run, append one line to `orchestrator-workspace/progress.md`.**

Format:
```
| {YYYY-MM-DD HH:mm} | {task-slug} | {done|partial|failed} | {one sentence: what was done and outcome} | {WORKSPACE} |
```

Example:
```
| 2026-03-23 14:45 | notifications-refactor | done | Refactored notifications API, added async support, all tests passing | orchestrator-workspace/20260323-1445-notifications-refactor/ |
```

**Rules:**
- One line per run — keep it scannable.
- If `progress.md` doesn't exist, create it with this header first:
  ```
  # Orchestrator Progress
  | Timestamp | Task | Status | Summary | Workspace |
  |---|---|---|---|---|
  ```
- Status is `done` (all steps completed), `partial` (stopped mid-run), or `failed` (critical step failed).
- This file lives at the top level — `orchestrator-workspace/progress.md` — not inside the task workspace, so any orchestrator can read it without knowing individual workspace paths.
- Do this directly, no sub-agent needed.

## Step 6: Invoke the Librarian

**After every orchestration run that changed or created files, dispatch a Librarian sub-agent (`doc-keeper`, Haiku)** to update `CLAUDE.md` files for any affected directories.

The Librarian should:
- Receive the list of all files written or modified during the run (from workspace outputs)
- For each affected directory that has (or should have) a `CLAUDE.md`, update it to reflect:
  - New files added and what they do
  - Existing files that changed significantly
  - Any new conventions, patterns, or dependencies introduced
- Keep each `CLAUDE.md` entry concise — one or two lines per file is enough

**Dispatch prompt pattern:**
> "You are the Librarian from a virtual software team. Your job is to keep `CLAUDE.md` files accurate and up to date. The following files were changed during this orchestration run: [list]. Review those files and update the relevant `CLAUDE.md` files so future agents understand what each file does and any conventions to follow. Do not rewrite sections unrelated to the changed files."

**Rules:**
- Only update `CLAUDE.md` entries for files that actually changed — don't touch unrelated sections.
- If a directory has no `CLAUDE.md` and multiple files were changed there, create one.
- This step is always last — run it after tests pass and the workspace is finalized.

## Step 7: Synthesize and Deliver

After all steps complete:
- Collect final artifacts
- If needed, dispatch a Synthesizer (`doc-keeper`) sub-agent to combine outputs
- Present the final result with an orchestration summary

## Workspace Structure

Each orchestration run gets its own isolated directory. Never write outside it.

```
orchestrator-workspace/
├── progress.md                          ← one line per run, shared across all tasks
└── {YYYYMMDD-HHmm}-{task-slug}/        ← unique per run
    ├── plan.md                          ← full orchestration plan (written by Planner, read by all)
    ├── step-01-{role}/
    │   ├── task.md                      ← prompt given to the sub-agent
    │   └── output.md                    ← what it produced
    ├── step-02-{role}/
    │   └── ...
    ├── step-03-{role}/
    │   ├── output/                      ← code files produced by Coder steps
    │   └── ...
    └── final/                           ← assembled deliverables
```

**Passing files between agents:** always use the full path from the workspace root. When dispatching a sub-agent, list every file it needs explicitly:
```
Context files:
  - {WORKSPACE}/plan.md
  - {WORKSPACE}/step-01-analysis/output.md
Output:
  - {WORKSPACE}/step-02-code/output/
```

## Example Workflows

These are starting points — adapt, skip steps, combine steps, or invent new workflows as the situation demands.

### Sprint Task Execution
When the user has a sprint file and wants tasks implemented:
1. **Analyzer** (`explorer`, Haiku): Parse sprint file, extract tasks with statuses and dependencies
2. **Planner** (`engineering-manager`, **Opus**): Deep codebase analysis, produce technical plan — orchestrator derives execution steps from it
3. For each task:
   - **Coder** (`developer`, Sonnet): Implement the task
   - **Reviewer** (`reviewer`, Sonnet): Review against spec and conventions
   - **Fixer** (`debugger`, Sonnet): Apply review fixes
   - **Tester** (`qa`, Sonnet): Write and run unit tests + integration tests
4. **Synthesizer** (`doc-keeper`, Haiku): Update sprint file with completion status
5. **Librarian** (`doc-keeper`, Haiku): Update `CLAUDE.md` files for all changed directories

### Feature Implementation
1. **Planner** (`engineering-manager`, **Opus**): Deep codebase analysis, produce technical plan — orchestrator derives execution steps from it
2. For each step: **Coder** → **Reviewer** → **Fixer** cycle
3. **Tester** (`qa`, Sonnet): Unit tests + integration tests
4. **Synthesizer** (`doc-keeper`, Haiku): Summary and docs
5. **Librarian** (`doc-keeper`, Haiku): Update `CLAUDE.md` files for all changed directories

### Code Review Pipeline
1. **Analyzer** (`explorer`, Haiku): Parse code, identify components
2. **Reviewer** (`reviewer`, Sonnet): General quality review
3. **Reviewer** (`security`, Opus): Security and correctness (for critical code)
4. **Synthesizer** (`doc-keeper`, Sonnet): Combined actionable report
5. **Librarian** (`doc-keeper`, Haiku): Update `CLAUDE.md` if review led to file changes

## Dispatching Sub-Agents Well

Give each sub-agent a clear virtual team identity. Example:

> "You are the Developer from a virtual software team. Your job is to write clean, working code. Implement the /api/auth/login endpoint as specified below. Write the result to orchestrator-workspace/step-03-code/auth.ts."

**Tips for effective dispatch:**
- Focused sub-agents with clear scope tend to produce better results
- Pass relevant context rather than full conversation history
- Specify expected output format when it matters
- That said, use your judgment — sometimes combining related work in one sub-agent is more efficient

**Always instruct sub-agents to read `CLAUDE.md` before reading source files.** Include this in every sub-agent prompt:
> "Before reading any source file, check if there is a `CLAUDE.md` in that directory. Read `CLAUDE.md` first — it describes what each file does and relevant conventions. Only open the actual source files if `CLAUDE.md` doesn't give you enough context."

**The Orchestrator does NOT do the work itself.**
Sub-agent prompts should contain: task description, relevant context, file paths, constraints, and output instructions.
Sub-agent prompts should NOT contain: pre-written code, pre-done analysis, drafted reviews, or any artifact the sub-agent is supposed to produce.
If the orchestrator writes the answer in the prompt, the sub-agent becomes a rubber stamp — the whole point of delegation is lost.

```
❌ Bad: "Here is the login function I drafted: [code]. Please review it."
✅ Good: "Review the login function in src/auth/login.ts. Check for security issues, edge cases, and convention compliance. Write your findings to orchestrator-workspace/step-04-review/output.md."
```

## Guidelines

These are principles, not rigid rules. Use your judgment to adapt based on the situation:

- **Plan proportionally.** Big tasks need real plans. Small tasks might just need a quick outline or none at all.
- **Delegate by default.** The orchestrator's strength is coordination, but if it makes sense to handle something directly, do it.
- **Prefer focused sub-agents.** Atomic tasks tend to produce better results, but don't split things artificially.
- **Be transparent about model choices.** Show which tier in the plan so the user can override if they want.
- **Fail forward.** If a sub-agent produces bad output, diagnose before re-running.
- **Surface what matters.** Checkpoint on important decisions. Run mechanical steps autonomously.

## Action Log — Document Your Work

**After completing any orchestration run, log to `docs/roles/activity-log.md` in the user's project.**

Append a new entry at the **top** of the file (newest first):

```markdown
## [YYYY-MM-DD] — <Brief title>

**Role:** Orchestrator
**Action:** <orchestration | sprint-execution | feature-build | review-pipeline>
**Summary:** <1-2 sentences: what was orchestrated and the outcome>

### Details
- <Number of steps in the plan>
- <Sub-agents dispatched and their roles>
- <Models used per step>
- <Workspace path>

### Outcome
- <Final deliverables produced>
- <Any issues encountered and how resolved>

### Next Steps
- <Recommended follow-up role or action>
```

**Rules for logging:**
- Always append new entries at the TOP of the file (newest first)
- If the file doesn't exist, create it with a header: `# Activity Log`
- Include the full step list from the orchestration plan
- Link to the `orchestrator-workspace/` for traceability
