# claude-workflow

A file-based development workflow plugin for Claude Code. Turns brainstorming sessions into structured roadmaps, milestones, and AI-executable task contracts.

## What It Does

| Skill | Purpose |
|-------|---------|
| **setup-roadmap** | Initialize the workflow structure for a project |
| **build-roadmap** | Transform brainstorm drafts into a formal roadmap with milestones and task contracts |
| **update-roadmap** | Add/remove/modify milestones and tasks after direction changes |
| **work-task** | Orchestrator: reads the task graph, presents status, dispatches subagents on your approval |
| **status** | Read-only dashboard: milestone progress, blocked items, next-up summary |

Internal (not user-invoked):

| Skill | Purpose |
|-------|---------|
| **update-task** | Cascade status updates through task → TASK.md → MILESTONE.md → ROADMAP.md |

## How It Works

The plugin creates a `.claude/roadmap/` directory in your project with:

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

Tasks are contracts designed for Claude Code subagents to execute autonomously. Standard contracts contain:

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

**Co-pilot, not autopilot.** You make the decisions — what to work on, how many in parallel, when to pause. Claude handles execution and bookkeeping.

### Statuses

**Tasks:** `pending` → `in_progress` → `review` → `completed` (also: `blocked`, `cancelled`)

**Milestones:** Derived from tasks by default. All pending → pending, any in_progress → in_progress, all completed → review (awaits user confirmation), then completed. Manual override available via `status_override`.

**Roadmap:** `active`, `paused`, `completed`, `archived`

## Installation

```bash
# From local path
claude plugin marketplace add <path-to-plugin>
claude plugin install claude-workflow

# To update after changes
claude plugin uninstall claude-workflow
claude plugin marketplace update claude-workflow-dev
claude plugin install claude-workflow
```

## Workflow

1. **Brainstorm freely** — save rough plans to `roadmap/drafts/`
2. **`/build-roadmap`** — formalize into structured roadmap with task contracts
3. **`/work-task`** — see status and dispatch subagents for task execution
4. **`/status`** — quick read-only progress check
5. **`/update-roadmap`** — adjust after new brainstorming or direction changes

## Compatibility

Designed for integration with [agent-cc](https://github.com/tronbonjovi/agent-cc). Task files use YAML frontmatter compatible with agent-cc's task parsing and Kanban board. Status updates cascade across all index files, so agent-cc's file watcher sees changes in real-time.

## Known Issues

- Milestone `review` state is skipped — milestones go directly from `in_progress` to `completed` instead of pausing for user confirmation
- Re-review cycle not enforced after subagent fixes

## License

MIT
