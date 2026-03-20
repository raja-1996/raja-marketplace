# Role Action Logs

> This folder contains action logs for each virtual team role. Every role writes its actions here so the entire team can track project progress.

## How It Works

Each role in the virtual team writes a log entry after completing any task. Logs are stored as individual markdown files per role, with newest entries at the top.

## Log Files

| Role | Log File | What's Tracked |
|------|----------|---------------|
| Rewriter | `rewriter-log.md` | Requirements clarified, specs rewritten |
| Brainstormer | `brainstormer-log.md` | Ideas explored, creative directions |
| Strategist | `strategist-log.md` | Decisions made, MVP scope defined |
| Engineering Manager | `engineering-manager-log.md` | Sprints planned, tasks broken down |
| Architect | `architect-log.md` | System designs, tech stack choices |
| UX Thinker | `ux-thinker-log.md` | User flows, screen designs, states |
| Explorer | `explorer-log.md` | Codebase mappings, patterns found |
| Doc Keeper | `doc-keeper-log.md` | Docs created/updated, context surfaced |
| Developer | `developer-log.md` | Code implemented, features built |
| Debugger | `debugger-log.md` | Bugs investigated, root causes found |
| Optimizer | `optimizer-log.md` | Performance improvements, measurements |
| Reviewer | `reviewer-log.md` | Code reviews, quality findings |
| QA | `qa-log.md` | Tests written, coverage achieved |
| Analyst | `analyst-log.md` | Data analyzed, insights produced |
| Security | `security-log.md` | Vulnerabilities found, security reviews |
| Skill Recommender | `skill-recommender-log.md` | Workflow recommendations, role routing |

## Entry Format

Every log entry follows this structure:

```markdown
## [YYYY-MM-DD] — Brief title

**Role:** <role-name>
**Action:** <action-type>
**Summary:** <1-2 sentences>

### Details
- <Key details>

### Outcome
- <What was produced>

### Next Steps
- <Recommended follow-up>
```

## Reading the Logs

- **To see what the project has been up to:** Read log files for all active roles
- **To see what a specific role did:** Read that role's log file
- **To find the latest actions:** Look at the top entry in each log file (newest first)
- **To understand handoffs:** Check the "Next Steps" section of each entry
