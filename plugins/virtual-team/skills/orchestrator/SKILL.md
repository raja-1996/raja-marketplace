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

## Step 1: Understand the Task

Read the user's request and identify:
- What is the desired end state?
- What inputs are available? (files, sprint plans, code, specs)
- What quality constraints exist? (must pass tests, follow conventions, needs security review)
- How much user oversight is wanted? (autonomous vs. checkpoint-heavy)

## Step 2: Create an Orchestration Plan

Before doing significant work, sketch a plan. Consult `references/orchestration-patterns.md` for composition patterns and `references/model-guide.md` for model selection — use these as starting points, not rigid templates.

A plan might include:
- Sub-agent type and virtual team role
- Recommended model tier (Haiku / Sonnet / Opus)
- Clear inputs and expected outputs
- Dependencies on prior steps
- Checkpoints where the user should review

**Always print the orchestration plan before executing — no exceptions.**
Display it in a clear, readable format like:

```
## Orchestration Plan

Step 1 — Analyzer (explorer, Haiku)
  What:   Parse the sprint file, extract task list, statuses, and dependencies
  Input:  sprint.md
  Output: orchestrator-workspace/step-01-analysis/output.md

Step 2 — Planner (engineering-manager, Sonnet)
  What:   Determine execution order and flag any blockers or ambiguities
  Input:  step-01 output
  Output: orchestrator-workspace/step-02-plan/output.md

Step 3 — Coder (developer, Sonnet)
  What:   Implement the feature according to the plan and existing conventions
  Input:  step-02 output
  Output: orchestrator-workspace/step-03-code/

...

Proceeding with execution.
```

Each step must include a `What:` line — one sentence describing the concrete action the sub-agent will take. This makes the plan scannable and helps the user understand what is happening at a glance.

Print the plan, then immediately proceed. Do not wait for user approval unless a checkpoint is explicitly required.

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

## Step 4: Synthesize and Deliver

After all steps complete:
- Collect final artifacts
- If needed, dispatch a Synthesizer (`doc-keeper`) sub-agent to combine outputs
- Present the final result with an orchestration summary

## Workspace Structure

```
orchestrator-workspace/
├── plan.md                    ← Approved orchestration plan
├── step-01-analysis/
│   ├── task.md                ← What the sub-agent was asked
│   └── output.md              ← What it produced
├── step-02-plan/
│   └── ...
├── step-03-code/
│   ├── output/                ← Code files produced
│   └── ...
├── step-04-review/
│   └── ...
└── final/                     ← Assembled deliverables
```

## Example Workflows

These are starting points — adapt, skip steps, combine steps, or invent new workflows as the situation demands.

### Sprint Task Execution
When the user has a sprint file and wants tasks implemented:
1. **Analyzer** (`explorer`, Haiku): Parse sprint file, extract tasks with statuses and dependencies
2. **Planner** (`engineering-manager`, Sonnet): Determine execution order
3. For each task:
   - **Coder** (`developer`, Sonnet): Implement the task
   - **Reviewer** (`reviewer`, Sonnet): Review against spec and conventions
   - **Fixer** (`debugger`, Sonnet): Apply review fixes
   - **Tester** (`qa`, Sonnet): Write and run tests
4. **Synthesizer** (`doc-keeper`, Haiku): Update sprint file with completion status

### Feature Implementation
1. **Planner** (`strategist` + `engineering-manager`, Sonnet/Opus): Break feature into steps
2. For each step: **Coder** → **Reviewer** → **Fixer** cycle
3. **Tester** (`qa`, Sonnet): Integration tests
4. **Synthesizer** (`doc-keeper`, Haiku): Summary and docs

### Code Review Pipeline
1. **Analyzer** (`explorer`, Haiku): Parse code, identify components
2. **Reviewer** (`reviewer`, Sonnet): General quality review
3. **Reviewer** (`security`, Opus): Security and correctness (for critical code)
4. **Synthesizer** (`doc-keeper`, Sonnet): Combined actionable report

## Dispatching Sub-Agents Well

Give each sub-agent a clear virtual team identity. Example:

> "You are the Developer from a virtual software team. Your job is to write clean, working code. Implement the /api/auth/login endpoint as specified below. Write the result to orchestrator-workspace/step-03-code/auth.ts."

**Tips for effective dispatch:**
- Focused sub-agents with clear scope tend to produce better results
- Pass relevant context rather than full conversation history
- Specify expected output format when it matters
- That said, use your judgment — sometimes combining related work in one sub-agent is more efficient

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
