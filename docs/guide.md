# Workflow Framework — User Guide

This guide walks through using the workflow framework to manage a project from idea to completion.

## Overview

The workflow framework is a file-based system for structuring AI-assisted development. Instead of keeping plans in your head or scattered across documents, it organizes everything into a `.claude/roadmap/` directory with clear files for each level of the hierarchy:

- **Roadmap** — the big picture (vision, goals, milestones)
- **Milestones** — groups of related work
- **Tasks** — individual units of work, written as contracts that AI subagents can execute

You stay in control of all decisions. The system handles file management, status tracking, and subagent dispatch.

## The Typical Workflow

### 1. Capture Ideas (`/draft`)

When you have an idea — rough, half-formed, or detailed — capture it immediately:

```
/draft
```

Just describe your idea. The skill saves it to `.claude/roadmap/drafts/` with a timestamp and slug. No planning overhead, no commitment. You can draft multiple ideas over time and formalize them later.

Drafts are seeds. They become input for `/build-roadmap` when you're ready to plan.

### 2. Set Up a New Project (`/setup-roadmap`)

For a brand new project, initialize the workflow structure:

```
/setup-roadmap
```

This creates the `.claude/roadmap/` directory with empty template files (ROADMAP.md, MILESTONE.md, TASK.md). You only need to do this once per project.

### 2b. Onboard an Existing Project (`/adopt-project`)

If your project already has code, docs, or planning artifacts:

```
/adopt-project
```

This scans your repo (read-only), finds existing planning documents (TODOs, specs, READMEs), and copies relevant ones into `drafts/` for you. It triages artifacts as "active" or "completed" so you don't re-plan finished work. Your original files are never modified.

### 3. Build the Roadmap (`/build-roadmap`)

Once you have drafts (or just ideas in your head), formalize them:

```
/build-roadmap
```

This is a **conversational** process — not a generator. It will:

1. Read your drafts and summarize what it found
2. Propose milestone groupings, one at a time, and confirm each with you
3. Break each milestone into task contracts with specific instructions, test cases, and acceptance criteria
4. Plan the execution order — phases, dependencies, what can run in parallel

Each task becomes a markdown contract file that a subagent can pick up and execute without asking questions. If something can't be written that prescriptively, it needs more brainstorming or needs to be split smaller.

If you already have milestones and want to add more, `/build-roadmap` defaults to additive mode — it won't overwrite existing work.

### 4. Work on Tasks (`/work-task`)

This is where execution happens:

```
/work-task
```

The orchestrator reads the task graph and presents:

- Current milestone and progress
- Which tasks are ready to work on (dependencies met)
- Parallel opportunities (tasks that can run simultaneously)
- Blocked items and why

You decide what to work on. When you approve a task, it:

1. Validates the task contract has all required fields
2. Dispatches a subagent with the contract, project context, and prior work brief
3. The subagent follows TDD — writes tests first, then implements
4. A reviewer subagent checks the work against the contract
5. If review fails, one fix cycle is attempted before escalating to you

**You can approve multiple tasks for parallel execution** if they don't touch the same files. The system checks for file overlap before dispatching.

### 5. Check Progress (`/status`)

Get a dashboard view at any time:

```
/status
```

This shows:

- Milestone summary table with progress bars
- Task-level detail for active milestones
- Blocked items
- Tasks ready to work on
- Archived/completed milestones
- Pending drafts count

It also silently syncs ROADMAP.md with the current state of all milestones.

### 6. Adjust the Plan (`/update-roadmap`)

Plans change. When you need to add milestones, cancel work, or restructure:

```
/update-roadmap
```

Use this for structural changes — adding new milestones, cancelling ones that are no longer needed, adding tasks to existing milestones, or reactivating archived work.

## Key Concepts

### Task Contracts

Task files aren't descriptions — they're **execution contracts**. Each one contains everything a subagent needs to do the work independently:

- **Objective** — what to deliver
- **Instructions** — step-by-step implementation guidance
- **Tests** — specific test cases to write (TDD)
- **Acceptance Criteria** — verifiable checklist for completion
- **filesTouch** — explicit list of files to create or modify

If a contract is incomplete, `/work-task` will flag it before dispatch and let you fill gaps or skip the task.

### Status Lifecycle

Tasks flow through: `pending` -> `in_progress` -> `review` -> `completed`

They can also be `blocked` (with a reason) or `cancelled`.

Milestones derive their status from their tasks — no manual tracking needed. A milestone with all tasks completed is automatically marked complete and archived.

### The Archive

When milestones are completed or cancelled, they move from the active index files (MILESTONE.md, TASK.md) into ARCHIVE.md. This keeps the active files small as the project grows. The task contract files stay on disk for reference.

### Co-pilot Model

The system is intentionally not fully automated. Every dispatch requires your approval. You see what's happening, you decide the pace, and you can pause, skip, or redirect at any point. The AI handles the tedious parts — file updates, status cascades, subagent coordination — while you make the decisions.

## Integration with agent-cc

Task files use YAML frontmatter that agent-cc can parse. Status updates cascade across all index files in real-time, so agent-cc's file watcher picks up changes immediately. This enables:

- Kanban board views of your task pipeline
- Session tracking across work-task dispatches
- Cross-project visibility in the agent-cc control center

## Tips

- **Draft liberally.** Capture ideas as they come. You can always decide not to formalize them.
- **Start small.** A roadmap with 2-3 milestones and 5-8 tasks is easier to manage than a 30-task plan.
- **Let the system batch.** Sequential simple tasks that share context can be batched to a single subagent.
- **Check `/status` between sessions.** It's a quick way to get re-oriented on where you left off.
- **Keep contracts specific.** Vague tasks produce vague results. If you can't write clear instructions, brainstorm more first.
