---
name: doc-keeper
description: >
  Surface existing project context before work and update documentation after work. Use this
  skill when the user needs to find project conventions, update README, write documentation,
  update changelogs, add inline comments, create API docs, review existing docs, write ADRs
  (architecture decision records), or maintain any project documentation. Trigger on "update
  the docs", "write documentation", "update README", "changelog", "what are our conventions",
  "document this", "add comments", "API docs", "ADR", "write a guide", "onboarding doc",
  "update the wiki", "what's our coding standard", or any request involving reading or writing
  project documentation. Also trigger at the START of work sessions to surface relevant
  context, and at the END to capture what changed.
---

# Doc Keeper — Documentation & Context Manager

The Doc Keeper is institutional memory. Before work: surfaces what the team needs to know. After work: captures what changed so future sessions aren't blind.

## Mental Model

Think like a **technical librarian** — know where everything is documented, keep it current, and present the right context at the right time.

Core question: **"What does someone need to know before touching this, and what changed that needs recording?"**

## Before Work — Surface Context

When a work session begins on a codebase:

1. **Find and present relevant docs:**
   - README.md — project setup, overview
   - CLAUDE.md / CONVENTIONS.md — coding standards
   - CHANGELOG.md — recent changes
   - Architecture Decision Records (ADRs) if they exist
   - Any docs/ folder content relevant to the current task

2. **Summarize the key constraints:**
   - Tech stack and versions
   - Coding conventions in use
   - Known gotchas or warnings
   - Dependencies and their roles

3. **Flag outdated docs.** If documentation contradicts the actual code, note it.

## After Work — Update Documentation

After code changes are made:

1. **README.md updates:**
   - New setup steps if dependencies changed
   - New environment variables
   - Changed build/run commands
   - Updated project description if scope changed

2. **CHANGELOG.md entries:**
   - What changed (feature added, bug fixed, breaking change)
   - Date and brief description
   - Follow Keep a Changelog format if the project uses it

3. **Inline documentation:**
   - Add/update comments for complex logic
   - Update JSDoc/docstrings for changed function signatures
   - Document non-obvious decisions with "why" comments

4. **API documentation:**
   - New/changed endpoints with request/response examples
   - Updated types/interfaces
   - Migration notes for breaking changes

## Creating New Documentation

When starting fresh or filling gaps:

**README.md template:**
- What this project does (one paragraph)
- Quick start (get running in < 5 steps)
- Architecture overview (brief)
- Key commands (dev, build, test, deploy)
- Environment variables
- Contributing guidelines

**ADR format (for significant decisions):**
- Title: Short decision name
- Status: Proposed / Accepted / Deprecated
- Context: What's the situation?
- Decision: What was decided?
- Consequences: What follows from this decision?

## Rules

- **Docs should be minimal and current.** Outdated docs are worse than no docs.
- **Write for the next person** (including future-you who forgot everything).
- **README = "get running fast."** Not a novel. Not a tutorial. Just enough to start.
- **Comments explain WHY, not WHAT.** Code says what. Comments say why.
- **Keep changelogs user-facing.** "Fixed auth redirect bug" not "refactored OAuth2 callback handler to properly encode state parameter."
- **Don't over-document.** If the code is clear, it doesn't need a comment.
