# Model Selection Guide

Reference for the Orchestrator skill. Describes which model tier to assign each sub-agent based on task complexity and stakes.

---

## Tiers

| Tier | Model | Cost | Speed | Use For |
|---|---|---|---|---|
| **Haiku** | claude-haiku-4-5 | Low | Fast | Mechanical, pattern-following tasks |
| **Sonnet** | claude-sonnet-4-6 | Medium | Medium | Judgment, implementation, analysis |
| **Opus** | claude-opus-4-6 | High | Slower | High-stakes, complex reasoning, ambiguous synthesis |

---

## Decision Heuristics

### Use Haiku when:
- The task is mechanical and pattern-following
- Output format is fully specified (parsing, extraction, formatting)
- Errors are low-cost (easily caught by a downstream step)
- Speed matters and quality variance is acceptable

**Examples:**
- Parse a sprint file and extract task list
- Format a list of issues into a Markdown table
- Scaffold boilerplate from a template
- Update a version number or status field
- Write a short summary of what was done

---

### Use Sonnet when:
- The task requires judgment but has clear criteria
- Implementation quality matters but is not life/security-critical
- The output will be reviewed by another step before being used
- Standard development tasks at senior engineer level

**Examples:**
- Implement a feature from a spec
- Review code for quality, conventions, and edge cases
- Write test cases for a module
- Analyze code structure and produce a findings report
- Debug a known-type issue
- Create a sprint plan from requirements

---

### Use Opus when:
- The task is architecturally consequential (hard to undo)
- Security, correctness, or performance is the primary concern
- The input is ambiguous and requires deep reasoning to interpret
- The output will be used directly without a downstream review
- Multiple reviewers disagreed and you need a tiebreaker
- You are synthesizing complex, conflicting information

**Examples:**
- Design a system architecture for a novel problem
- Security review of authentication, authorization, or cryptographic code
- Debug a race condition or complex distributed system failure
- Interpret ambiguous requirements into a precise spec
- Synthesize a code review when Sonnet missed critical issues

---

## Per-Sub-Agent Defaults

| Sub-Agent Type | Default Tier | Escalate To | When to Escalate |
|---|---|---|---|
| Analyzer | Haiku | Sonnet | Codebase is complex or poorly structured |
| Planner | Sonnet | Opus | Architecture decisions are embedded in the plan |
| Architect | Opus | — | Always Opus; too consequential for lower tiers |
| Coder | Sonnet | Opus | Security-sensitive, algorithmic, or high-complexity code |
| Reviewer (quality) | Sonnet | Opus | Critical pre-production review |
| Reviewer (security) | Opus | — | Always Opus for security |
| Tester | Sonnet | — | Standard tier is sufficient |
| Fixer | Sonnet | Opus | Root cause is unclear or fix is architecturally impactful |
| Synthesizer | Haiku | Sonnet | Synthesis requires interpreting conflicting information |

---

## Cost Management

- **Start with defaults.** Don't use Opus everywhere — it adds cost and latency without proportional quality gains for mechanical tasks.
- **Escalate on evidence.** Only escalate when a lower-tier sub-agent actually produces poor output, not preemptively.
- **Fan-out selectively.** When running parallel reviewers, use Sonnet for general review and Opus only for the security pass.
- **Checkpoint before expensive steps.** If you're about to run 5 Opus sub-agents, checkpoint with the user first.
