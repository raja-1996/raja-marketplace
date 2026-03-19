# Virtual Team — Claude Code Plugin

A virtual software team of 14 specialized personas for Claude Code. Each skill embodies a distinct role with its own mental model, priorities, and output style.

## The Team

| # | Skill | Slash Command | Mental Model |
|---|---|---|---|
| 1 | **Rewriter** | `/virtual-team:rewriter` | Clarify intent before anything runs |
| 2 | **Brainstormer** | `/virtual-team:brainstormer` | Go wide. No judgment. What could exist? |
| 3 | **Strategist** | `/virtual-team:strategist` | Worth it? Scope it. Plan it. Prioritize. |
| 4 | **Architect** | `/virtual-team:architect` | Right structure, stack, patterns, contracts |
| 5 | **UX Thinker** | `/virtual-team:ux-thinker` | What does the user see, feel, and do? |
| 6 | **Explorer** | `/virtual-team:explorer` | Map the territory. What exists? How does it flow? |
| 7 | **Doc Keeper** | `/virtual-team:doc-keeper` | Surface context before. Update docs after. |
| 8 | **Developer** | `/virtual-team:developer` | Build it clean, build it now |
| 9 | **Debugger** | `/virtual-team:debugger` | Hypothesize → trace → eliminate |
| 10 | **Optimizer** | `/virtual-team:optimizer` | Measure → profile → improve → measure |
| 11 | **Reviewer** | `/virtual-team:reviewer` | Structure + quality + edge cases + regret |
| 12 | **QA** | `/virtual-team:qa` | Prove it works. Prove it can't break. |
| 13 | **Analyst** | `/virtual-team:analyst` | What does the data say? What to do about it? |
| 14 | **Security** | `/virtual-team:security` | How can someone break into this? |

## How It Works

**Auto-trigger:** Skills activate automatically based on conversation context. Ask "help me brainstorm app ideas" and the Brainstormer activates. Say "review this code" and the Reviewer takes over.

**Manual invoke:** Use namespaced slash commands (e.g., `/virtual-team:brainstormer`) to explicitly invoke any persona.

## Workflows

```
IDEA → APP:
  brainstormer → strategist → architect → ux-thinker → developer → reviewer → qa → security → doc-keeper

BUG FIX:
  explorer → debugger → developer → reviewer → qa

OPTIMIZE:
  explorer → analyst → optimizer → reviewer → qa

QUICK ADHOC:
  rewriter → developer → qa

DATA-DRIVEN DECISIONS:
  analyst → strategist → architect → developer
```

## Installation

### Option 1: Local testing (no GitHub needed)

```bash
claude --plugin-dir /path/to/virtual-team
```

This loads the plugin for the current session. Skills show up in the slash command menu and auto-trigger based on context.

### Option 2: Direct install from GitHub

First, push this repo to GitHub. Then in Claude Code:

```
/plugin install https://github.com/YOUR_USERNAME/virtual-team
```

### Option 3: Via a marketplace

If you host multiple plugins in a marketplace repo:

```
/plugin marketplace add YOUR_USERNAME/your-marketplace-repo
/plugin install virtual-team@your-marketplace-repo
```

### Verify installation

After installing, run:

```
/virtual-team:rewriter
```

If the skill loads and responds, you're set. You can also run `/plugin` to see installed plugins.

## Customization

Each skill is a standalone SKILL.md file. Edit any skill to match your preferences:

- Add your preferred tech stack to `skills/architect/SKILL.md`
- Add your coding conventions to `skills/developer/SKILL.md`
- Customize review severity levels in `skills/reviewer/SKILL.md`
- Add company-specific security requirements to `skills/security/SKILL.md`

## Plugin Structure

```
virtual-team/
├── .claude-plugin/
│   └── plugin.json              ← Plugin manifest (required)
├── README.md                    ← This file
└── skills/                      ← Auto-discovered by Claude Code
    ├── rewriter/SKILL.md
    ├── brainstormer/SKILL.md
    ├── strategist/SKILL.md
    ├── architect/SKILL.md
    ├── ux-thinker/SKILL.md
    ├── explorer/SKILL.md
    ├── doc-keeper/SKILL.md
    ├── developer/SKILL.md
    ├── debugger/SKILL.md
    ├── optimizer/SKILL.md
    ├── reviewer/SKILL.md
    ├── qa/SKILL.md
    ├── analyst/SKILL.md
    └── security/SKILL.md
```

## License

MIT
