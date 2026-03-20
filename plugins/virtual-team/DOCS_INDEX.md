# Docs Index

> LLM entry point. Read this file first to find the right doc for your current task or role.

## Plugin Docs

| Doc | Path | Contains | Use When |
|-----|------|----------|----------|
| Plugin README | `./README.md` | Team roster, workflows, installation, plugin structure | Onboarding, understanding available skills, installing the plugin |
| Plugin Manifest | `./.claude-plugin/plugin.json` | Plugin metadata, version, skill discovery path | Debugging plugin loading or checking version |

## Skill Definitions

Each skill is a self-contained persona with its own mental model, process, and rules. Read a skill doc when you need to understand or invoke that role.

| Skill | Path | Role Summary | Invoke For |
|-------|------|-------------|------------|
| Rewriter | `./skills/rewriter/SKILL.md` | Clarifies vague requests into precise specs | Ambiguous requirements, unclear user intent |
| Brainstormer | `./skills/brainstormer/SKILL.md` | Generates ideas without judgment | Ideation phase, exploring possibilities |
| Strategist | `./skills/strategist/SKILL.md` | Prioritizes and scopes to MVP | Too many ideas, need to converge and decide |
| Engineering Manager | `./skills/engineering-manager/SKILL.md` | Breaks MVP into tasks and sprints | Execution planning, task breakdown |
| Architect | `./skills/architect/SKILL.md` | Designs system structure, stack, patterns | Tech stack decisions, system design |
| UX Thinker | `./skills/ux-thinker/SKILL.md` | Designs user experience and flows | UI/UX decisions, user journey mapping |
| Explorer | `./skills/explorer/SKILL.md` | Maps existing codebase structure and flow | Unfamiliar codebase, orientation |
| Doc Keeper | `./skills/doc-keeper/SKILL.md` | Manages LLM-optimized documentation | Writing/updating docs, surfacing project context |
| Developer | `./skills/developer/SKILL.md` | Builds clean, working code | Implementation, coding tasks |
| Debugger | `./skills/debugger/SKILL.md` | Hypothesis-driven root cause analysis | Something is broken, need to find why |
| Optimizer | `./skills/optimizer/SKILL.md` | Measures, profiles, and improves performance | Slow code, high resource usage |
| Reviewer | `./skills/reviewer/SKILL.md` | Code quality and architecture review | Pre-merge review, quality check |
| QA | `./skills/qa/SKILL.md` | Test coverage and reliability | Need tests, need to verify correctness |
| Analyst | `./skills/analyst/SKILL.md` | Interprets data to drive decisions | Have metrics/data, need insights |
| Security | `./skills/security/SKILL.md` | Identifies vulnerabilities and attack surfaces | Pre-deploy security check, threat modeling |
| Skill Recommender | `./skills/skill-recommender/SKILL.md` | Recommends next skill based on context | Unsure what to do next, need workflow guidance |

## Standard Workflows

> Use these to determine which skills to invoke in sequence.

| Workflow | Sequence | Use When |
|----------|----------|----------|
| Idea to App | brainstormer → strategist → engineering-manager → architect → ux-thinker → developer → reviewer → qa → security → doc-keeper | Building something new from scratch |
| Bug Fix | explorer → debugger → developer → reviewer → qa | Something is broken |
| Optimize | explorer → analyst → optimizer → reviewer → qa | Performance issues |
| Quick Adhoc | rewriter → developer → qa | Small, clear task |
| Data-Driven | analyst → strategist → architect → developer | Decisions informed by data |
