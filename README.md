# workflow-framework

A file-based development workflow system for AI-assisted coding. Turns brainstorming sessions into structured roadmaps, milestones, and AI-executable task contracts.

Currently built as a Claude Code plugin. Future roadmap includes support for additional AI agents and providers.

## What It Does

Provides a structured development lifecycle — from capturing ideas to dispatching work to AI subagents — using plain markdown files as the source of truth.

| Skill | Purpose |
|-------|---------|
| **draft** | Quick-capture an idea to `drafts/` — no milestones, no tasks, just a seed |
| **adopt-project** | Onboard an existing project: scan repo, find planning artifacts, bootstrap workflow |
| **setup-roadmap** | Initialize the workflow structure for a new project |
| **build-roadmap** | Transform brainstorm drafts into a formal roadmap with milestones and task contracts |
| **update-roadmap** | Add/remove/modify milestones and tasks after direction changes |
| **work-task** | Orchestrator: reads the task graph, presents status, dispatches subagents on your approval |
| **status** | Dashboard + sync: milestone progress, blocked items, next-up, and ROADMAP.md sync |

Internal (not user-invoked):

| Skill | Purpose |
|-------|---------|
| **update-task** | Cascade status updates through task -> TASK.md -> MILESTONE.md |

## How It Works

The system creates a `.claude/roadmap/` directory in your project:

```
.claude/roadmap/
├── ROADMAP.md                          # Big picture: vision, goals, milestone overview
├── MILESTONE.md                        # Index of active milestones with derived status
├── TASK.md                             # Battle plan: execution order, phases, parallelism
├── ARCHIVE.md                          # Completed/cancelled milestones (auto-created)
├── drafts/                             # Brainstorm notes (input for build-roadmap)
├── <milestone-name>/
│   ├── <milestone-name>-task001.md     # Task contract
│   ├── <milestone-name>-task002.md
│   └── ...
└── ...
```

### Task Contracts

Tasks are contracts designed for AI subagents to execute autonomously. Standard contracts contain:

- **Objective** — What the subagent should deliver
- **Instructions** — Step-by-step implementation guidance
- **Tests** — What tests to write and what they should verify
- **Acceptance criteria** — How to know the task is done

Complex contracts add: Context, Technical Approach, Scope boundaries, and Risk Notes.

### Execution Model

The orchestrator (`work-task`) reads the task graph, understands phase ordering, dependencies, and parallel opportunities, then:

1. Presents current state and recommends what's next
2. Waits for your approval before dispatching work
3. Spins up subagents to execute approved task contracts
4. Monitors results and runs review cycles (max 2 per task)
5. Cascades status updates across all index files
6. Presents the next options

**Co-pilot, not autopilot.** You make the decisions — what to work on, how many in parallel, when to pause. The AI handles execution and bookkeeping.

### Statuses

**Tasks:** `pending` -> `in_progress` -> `review` -> `completed` (also: `blocked`, `cancelled`)

**Milestones:** `planned` (on roadmap, no tasks yet) -> `pending` (tasks defined) -> derived from task statuses. Manual override available via `status_override`.

**Roadmap:** `active`, `paused`, `completed`, `archived`

## Installation

```bash
# Add the marketplace and install
claude plugin marketplace add <path-to-plugin>
claude plugin install workflow-framework

# To update after changes
claude plugin uninstall workflow-framework
claude plugin marketplace update workflow-framework-dev
claude plugin install workflow-framework
```

## Quick Start

```
/draft          — capture an idea
/setup-roadmap  — initialize the workflow structure
/build-roadmap  — formalize drafts into milestones and task contracts
/work-task      — dispatch subagents to execute tasks
/status         — check progress
```

See [docs/guide.md](docs/guide.md) for a full usage walkthrough.

## Compatibility

Designed for integration with [agent-cc](https://github.com/tronbonjovi/agent-cc). Task files use YAML frontmatter compatible with agent-cc's task parsing and Kanban board. Status updates cascade across all index files, so agent-cc's file watcher sees changes in real-time.

## Future

- Support for additional AI agent providers beyond Claude Code
- Deeper agent-cc integration for cross-session workflow tracking
- Workflow templates for common project types

## License

MIT
