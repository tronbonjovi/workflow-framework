# claude-workflow

A file-based development workflow plugin for Claude Code. Turns brainstorming sessions into structured roadmaps, milestones, and executable task contracts.

## What It Does

- **setup-roadmap** — Initialize the workflow structure for a project
- **build-roadmap** — Transform brainstorm drafts into a formal roadmap with milestones and tasks
- **update-roadmap** — Add/remove/modify milestones and tasks after direction changes
- **update-task** — Change task status, add notes, mark blocked/completed

## How It Works

The plugin creates a `.claude/roadmap/` directory in your project with:

- `ROADMAP.md` — Big picture: vision, goals, milestone overview
- `MILESTONE.md` — Index of all milestones with derived status
- `TASK.md` — Battle plan: execution order, phases, parallelism
- `<milestone>/` — Directories containing individual task contract files

Skills own all file creation and updates, ensuring cross-file consistency.

## Installation

```bash
claude plugin add claude-workflow
```

## Workflow

1. Brainstorm freely — save rough plans to `roadmap/drafts/`
2. Run `/build-roadmap` to formalize into structured roadmap
3. Pick up tasks per session — run `/update-task` to track progress
4. Revisit and adjust — run `/update-roadmap` after new brainstorming

## Compatibility

Designed for integration with [agent-cc](https://github.com/tronbonjovi/agent-cc). Task files use YAML frontmatter compatible with agent-cc's task parsing and Kanban board.
