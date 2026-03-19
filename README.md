# raja-marketplace

A Claude Code plugin marketplace.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [virtual-team](plugins/virtual-team/) | 14 specialized personas (brainstormer, architect, developer, reviewer, etc.) |

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add raja-1996/raja-marketplace
```

Then install a plugin:

```
/plugin install virtual-team@raja-marketplace
```

## Adding More Plugins

1. Create a new plugin directory under `plugins/` with a `.claude-plugin/plugin.json` manifest and `skills/` directory
2. Add the plugin entry to `.claude-plugin/marketplace.json`
