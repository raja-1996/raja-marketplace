# Orchestration Patterns

Reference for the Orchestrator skill. Describes the available sub-agent composition patterns and when to use each.

---

## Sub-Agent Types

Each sub-agent maps to one or more virtual team roles:

| Type | Virtual Team Role(s) | Complexity | Use For |
|---|---|---|---|
| **Analyzer** | `explorer`, `analyst` | Low–Medium | Parsing, extraction, mapping existing code or data |
| **Planner** | `strategist`, `engineering-manager` | Medium–High | Task breakdown, sprint planning, execution ordering |
| **Architect** | `architect` | High | System design, stack decisions, contract definition |
| **Coder** | `developer` | Medium | Feature implementation, scaffolding, API building |
| **Reviewer** | `reviewer`, `security` | Medium–High | Code quality, convention checking, vulnerability finding |
| **Tester** | `qa` | Medium | Test writing, coverage, correctness verification |
| **Fixer** | `debugger`, `optimizer` | Medium | Applying fixes, resolving bugs, improving performance |
| **Synthesizer** | `doc-keeper` | Low–Medium | Combining outputs, updating docs, writing summaries |

---

## Composition Patterns

### 1. Linear Pipeline

Steps run in strict sequence. Each step's output is the next step's input.

```
Analyzer → Planner → Coder → Reviewer → Fixer → Tester → Synthesizer
```

**Use when:** Work must be done in order, each phase depends on the last.
**Example:** Sprint task execution — you can't code before you've planned, can't test before you've coded.

---

### 2. Fan-Out (Parallel)

One input distributed to multiple sub-agents working independently, then results merged.

```
         ┌→ Reviewer (quality)  ┐
Coder  →─┤                      ├→ Synthesizer
         └→ Reviewer (security) ┘
```

**Use when:** Multiple independent analyses of the same artifact are needed simultaneously.
**Example:** Code review pipeline — a quality reviewer and a security reviewer can both read the same code independently, then a Synthesizer merges their findings.

---

### 3. Iterative Refinement

A Coder–Reviewer–Fixer loop that repeats until output meets quality bar.

```
Coder → Reviewer → [issues found?] → Fixer → Reviewer → [pass?] → done
```

**Use when:** Output quality is uncertain and the cost of iteration is worth the quality gain.
**Example:** Implementing a complex algorithm or security-sensitive code where "good enough on first try" is unlikely.
**Limit:** Cap refinement loops at 2–3 iterations. If still failing, escalate model or checkpoint with user.

---

### 4. Escalation

Re-dispatch a sub-agent with a higher-tier model when the current tier produces poor output.

```
Reviewer (Sonnet) → [poor output] → Reviewer (Opus)
```

**Use when:** A mechanical or judgment-level sub-agent produces output that is incorrect, incomplete, or clearly insufficient.
**Example:** Sonnet misses a complex race condition in a review → escalate to Opus for that specific review.

---

### 5. Checkpoint-Then-Continue

Insert explicit user checkpoints at high-stakes decision points.

```
Planner → [CHECKPOINT: show plan to user] → Coder → ... → [CHECKPOINT: show diff to user] → merge
```

**Use when:** The next steps are hard to reverse, or the user should validate before significant work proceeds.
**Example:** After generating a plan that will touch 20+ files, checkpoint before starting the coding phase.

---

## Pattern Selection Guide

| Situation | Recommended Pattern |
|---|---|
| Clear sequence of phases | Linear Pipeline |
| Multiple reviewers of same code | Fan-Out |
| Quality-critical code, first-pass uncertain | Iterative Refinement |
| Sub-agent producing poor output | Escalation |
| About to take destructive or large-scope action | Checkpoint-Then-Continue |
| Multiple independent tasks in a sprint | Fan-Out within Linear (parallelize tasks, then synthesize) |

---

## Workspace Convention

Every step writes its output to:

```
orchestrator-workspace/
  step-{NN}-{role}/
    task.md      ← what the sub-agent was asked
    output.md    ← what it produced (or output/ dir for code)
```

Later sub-agents reference earlier steps by reading these files — not by receiving the full conversation history.
