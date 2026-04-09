---
name: build-roadmap
description: Use when you have brainstorm drafts or informal plans and want to formalize them into a structured roadmap with milestones and AI-executable task contracts — conversational skill that walks through the process collaboratively
---

# Build Roadmap

Transform informal brainstorming outputs into a formal, structured roadmap with milestones and AI-executable task contracts.

**This is a conversational skill, not a generator.** It follows a deliberate cadence — asking questions, proposing groupings, and confirming intent at each step. The drafts are input context, not marching orders.

## When to Use

- After brainstorming sessions that produced rough plans
- When draft files exist in `.claude/roadmap/drafts/`
- When spec files exist in `docs/superpowers/specs/` that should inform the roadmap
- When the user says "let's turn this into a roadmap" or "formalize the plan"

## Pre-Check

1. Verify `.claude/roadmap/` exists. If not: "No roadmap found. Run `/setup-roadmap` to get started."
2. Scan for input sources:
   - `.claude/roadmap/drafts/*.md` — brainstorm drafts
   - `docs/superpowers/specs/*.md` — design specs
3. If no drafts or specs found: ask the user what they'd like to build the roadmap from. They may want to describe it in conversation, paste notes, or point to other files.
4. Check if ROADMAP.md already has milestones defined:
   - **If milestones exist:** default to additive mode. Ask: "Add new milestones to the existing roadmap, or start fresh?" If additive, read existing milestones from MILESTONE.md to avoid name conflicts.
   - **If no milestones:** proceed with fresh roadmap creation.
5. If multiple drafts exist, let the user choose: "I see N drafts. Which ones should we work with this session?" List them. Accept "all" or specific selections.
6. Check if MILESTONE.md has milestones with no task breakdown (status `planned`). If so, offer to resume: "**<name>** has no task breakdown yet. Want to define tasks for it?"

## Step 1: Read and Summarize

Read all input sources. Present a summary to the user covering:
- **Key themes** — main ideas, features, and goals identified
- **Potential scope** — your sense of how big the work is
- **Questions** — anything unclear or contradictory in the drafts

Ask: "Does this capture it, or am I missing something?" Wait for response and adjust understanding.

## Step 2: Propose Milestones

Propose milestones **one at a time**. For each, confirm:
- The name (lowercase, hyphenated — this becomes the directory name)
- The description
- What it achieves (the "point of culmination")

After all milestones are identified, present the full ordered list and confirm: "Does this order make sense? Should any be reordered, combined, or split?"

**Checkpoint — Milestones Complete:** After the milestone list is confirmed, offer:
1. **Write milestones now** and stop — come back later to define tasks.
2. **Continue to task breakdown.**

If stopping: write milestones to ROADMAP.md (Active Milestones table) and MILESTONE.md with status `planned`. No task files. Report what was written and stop.

## Step 3: Break Down Tasks as Contracts

For each milestone, propose tasks. Present each as: title, one-line description, standard/complex, files touched. Confirm the task list before writing full contracts.

After confirmation, write out the full contract for each task. Confirm each: "Does this task contract capture what you want? Anything to adjust?"

Task files are dispatch payloads for Claude Code subagents. Each must contain enough detail that a subagent can execute without asking questions. If a task can't be written this prescriptively, it's too vague (needs brainstorming) or too large (needs splitting).

**Standard tasks** — sections: Objective, Instructions, Tests, Acceptance Criteria.

**Complex tasks** — add: Context, Technical Approach, Scope, Risk Notes.

**All tasks** also specify: Files to touch (explicit list), parallel-safe (yes/no).

**Naming:** `<milestone-name>-task<NNN>.md` — sequential numbering starting at 001 within each milestone. The filename is an ID; the human-readable title lives in frontmatter.

**Checkpoint — Between Milestones:** After completing task breakdown for a milestone, if more remain, ask: "Move on to **<next-milestone>**, or stop here?" If stopping: write everything confirmed so far — milestones with tasks get status `pending`, remaining stay `planned`. Report and stop.

## Step 4: Plan Sequencing

Once all milestones and tasks are confirmed, propose the phase/execution plan:

- **Phase groupings** — which tasks go in each phase and why
- **Dependencies** — which tasks block others, with rationale
- **Parallel opportunities** — tasks that touch different files with no dependencies and can run simultaneously
- **Batching candidates** — sequential, simple tasks that one subagent could handle in a single session

Adjust based on feedback.

## Step 5: Generate Files

Once everything is confirmed, generate all files at once:

1. **ROADMAP.md** — vision, goals, `sources:` list, `milestones:` list, Active Milestones table (`pending` if tasks defined, `planned` if not).
2. **MILESTONE.md** — section per milestone: description, directory, task list, created date, status_override: null.
3. **Milestone directories** — `.claude/roadmap/<milestone-name>/` for each.
4. **Task files** — use `templates/task-standard.md` or `templates/task-complex.md` from the plugin's root `templates/` directory. Replace all `{{PLACEHOLDER}}` values. Set `dependsOn`, `phase`, `parallelSafe`, `filesTouch`.
5. **TASK.md** — phase sections with tables (Task ID, Milestone, Status, Dependencies, Complexity, Parallel-Safe). Include parallelism and batching notes.

## Completion

After generating all files:

> Roadmap built for **<project-name>**:
> - **Milestones:** <count> (<list names>)
> - **Tasks:** <count> total across all milestones
> - **Phases:** <count>
