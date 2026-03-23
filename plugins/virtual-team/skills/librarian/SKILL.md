---
name: librarian
description: >
  Index the entire codebase by creating CLAUDE.md files in every folder, giving LLMs a
  high-density map to navigate without opening files. Use this skill when the user wants
  to document the codebase for LLM navigation, generate a codebase index, create folder-level
  summaries, or set up a two-tier read system where an LLM reads CLAUDE.md first and drills
  into actual files only when needed. Trigger on "index the codebase", "create CLAUDE.md",
  "document all folders", "map the codebase for LLM", "generate codebase index", "librarian",
  "create file summaries", "help the LLM navigate the codebase", or any request to create
  an AI-readable map of the project structure.
---

# Librarian — LLM Codebase Indexer

The Librarian catalogs the entire codebase into dense, machine-readable `CLAUDE.md` files — one per folder. The goal: an LLM should read `CLAUDE.md` first to orient itself, and only open actual files when it needs deeper detail. Two-tier navigation. Maximum signal, minimum tokens.

## Mental Model

Think like a **librarian building a card catalog for AI agents**. Every folder gets an index card. Every file on that card gets a precise, dense description. No prose. No padding. Just the facts an LLM needs to decide: "Is this the file I need?"

Core question: **"What does an LLM need to know about this file WITHOUT opening it?"**

## What to Write in Each CLAUDE.md

### Folder Header
One line. What is this folder's single responsibility in the system?

```
# <folder-name>
<One-line description of the folder's role in the system.>
```

### File Entries
One bullet per file. Dense, structured, no full sentences.

```
- `filename.ts` — <what it does in one phrase>
  - exports: <key functions, classes, constants, types exported>
  - deps: <internal deps (../x) and notable external packages>
  - types: <key interfaces/types defined here and their shape>
  - side-effects: <DB calls, API calls, file system, global state mutations — or "none">
  - env: <env vars it reads — or omit if none>
  - pattern: <architectural pattern used — or omit if obvious>
  - gotcha: <non-obvious behavior, known caveats, important warnings — or omit if none>
```

**Rules for file entries:**
- Purpose line: describe what the file *does*, not what it *is*
- `exports`: list only what external code would import — skip private internals
- `deps`: internal paths first, then external packages; skip `react`, `fs`, obvious stdlib
- `types`: include the shape if it's non-obvious (e.g., `User {id, email, role}`)
- `side-effects`: be explicit — this is what burns people. "DB write (user table)", "calls Stripe API", "mutates global config"
- `env`: list specific var names (e.g., `JWT_SECRET`, `DATABASE_URL`)
- `pattern`: only include when non-obvious (middleware, singleton, factory, HOC, repository, pub-sub)
- `gotcha`: tribal knowledge — things that would surprise a reader. This is the most valuable field
- **Skip any field that doesn't apply** — don't write `side-effects: none` if there are none, just omit it
- `index.ts` / `index.js` barrel files: just write `- index.ts — re-exports: <list what it re-exports>`

## How to Execute

### Step 1 — Scan the codebase
Walk every directory recursively. Build a mental map of folders before writing anything.

Skip these folders entirely (do not create CLAUDE.md inside them):
- `node_modules/`
- `.git/`
- `.next/`
- `dist/`, `build/`, `out/`
- `.turbo/`, `.cache/`
- Any folder that only contains generated or compiled files

### Step 2 — Read files to understand them
For each folder, read each file before writing its entry. Do not guess. Read the actual imports, exports, types, and logic. Accuracy over speed.

### Step 3 — Write CLAUDE.md per folder
- Create `CLAUDE.md` at the root of each folder (not subfolders — each folder gets its own)
- If a `CLAUDE.md` already exists, overwrite it with fresh content
- Write in the format defined above — dense, structured, no prose

### Step 4 — Write root CLAUDE.md last
After all folders are indexed, write `CLAUDE.md` at the project root. This is the top-level entry point.

Root CLAUDE.md format:
```
# <project-name>
<One-line description of what this project is and does.>

## Stack
- <language/runtime>: <version if known>
- <framework>: <what it's used for>
- <database>: <what it stores>
- <key packages>: <what they do>

## Entry Points
- `<file>` — <how the app starts / main entry>
- `<file>` — <alternative entry, e.g. API server>

## Folder Map
- `<folder>/` — <one-line purpose>
- `<folder>/` — <one-line purpose>
...

## Key Conventions
- <naming convention>
- <import convention>
- <state management approach>
- <testing approach>
- <any other non-obvious pattern used throughout>

## Environment Variables
| Variable | Required | Purpose |
|----------|----------|---------|
| `VAR_NAME` | yes/no | <what it controls> |
```

### Step 5 — Verify coverage
After writing all CLAUDE.md files, do a final scan. Every non-skipped folder should have a CLAUDE.md. Report the count to the user.

## Output to User

When done, report:

```
## Librarian — Indexing Complete

**Folders indexed:** <N>
**Files cataloged:** <N>
**CLAUDE.md files created:** <N>

### Coverage
- <folder/> ✓
- <folder/> ✓
...

### Skipped
- <folder/> — <reason>

### Notes
- <any unusual patterns found>
- <any files that were ambiguous or hard to classify>
```

## Quality Bar

Each CLAUDE.md should pass this test: **Can an LLM read only this file and know exactly which file to open next for any task in this folder?**

If the answer is no — the entries are too vague, missing key exports, or skipping important side-effects — rewrite until the answer is yes.

## Rules

- **Read before you write.** Never summarize a file without reading it. Guessing creates false maps.
- **Dense > verbose.** This is for LLMs, not humans. Pack maximum signal into minimum tokens.
- **Accuracy > completeness.** A missing field is better than a wrong field.
- **Gotchas are gold.** Non-obvious behavior is the most valuable thing to document — it's what saves debugging sessions.
- **No prose.** No full sentences except the one-line folder description. Everything else is structured key-value.
- **Overwrite on re-run.** CLAUDE.md files are always regenerated fresh — they are not manually maintained.
- **Skip generated code.** node_modules, dist, build — never index these.
