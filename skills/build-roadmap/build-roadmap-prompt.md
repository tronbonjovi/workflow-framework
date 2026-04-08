# Build Roadmap — Detailed Process

This document defines the conversational flow for building a roadmap. Follow these steps in order.

## Step 1: Read and Summarize

Read all available input sources:
- All files in `.claude/roadmap/drafts/`
- All files in `docs/superpowers/specs/` (if they exist)

Present a summary to the user:

> "I've read through your brainstorming notes. Here's what I'm picking up:
>
> **Key themes:** [list the main ideas/features/goals you identified]
>
> **Potential scope:** [your sense of how big this is]
>
> **Questions already:** [anything unclear or contradictory in the drafts]
>
> Does this capture it, or am I missing something?"

Wait for the user's response. Adjust your understanding based on their feedback.

## Step 2: Propose Milestones

Based on the drafts and conversation, propose milestones **one at a time**:

> "I think the first milestone should be **<milestone-name>**: <description>.
>
> This would cover: <what achieving this milestone means>
>
> Does this feel right as a milestone, or should it be scoped differently?"

For each milestone, confirm:
- The name (lowercase, hyphenated — this becomes the directory name)
- The description
- What it achieves (the "point of culmination")

After confirming a milestone, move to the next. Continue until all milestones are identified.

Then present the full milestone list:

> "Here's the full milestone plan:
> 1. **<name>** — <description>
> 2. **<name>** — <description>
> ...
>
> Does this order make sense? Should any be reordered, combined, or split?"

## Step 3: Break Down Tasks as Contracts

For each confirmed milestone, propose tasks. Remember: these are **execution contracts for subagents**, not descriptions for humans.

> "For the **<milestone-name>** milestone, I'm thinking these tasks:
>
> 1. **<task-title>** — <one-line description>. <standard/complex>. Touches: <files>.
> 2. **<task-title>** — <one-line description>. <standard/complex>. Touches: <files>.
> ...
>
> Does this task list cover what's needed for this milestone?"

After the user confirms the task list for a milestone, write out the full contract for each task. The sections depend on the task's complexity:

**Standard-complexity tasks** — write these sections:
- Objective (specific outcome)
- Instructions (step-by-step: what to create, what to modify, what patterns to follow)
- Tests (specific test cases — TDD style)
- Acceptance criteria (verifiable checklist)

**Complex-complexity tasks** — write these sections:
- Context (why this task exists, what it builds on)
- Objective (specific outcome)
- Technical approach (architecture decisions, patterns to use)
- Instructions (step-by-step: what to create, what to modify, what patterns to follow)
- Tests (specific test cases — TDD style)
- Acceptance criteria (verifiable checklist)
- Scope (in/out boundaries)
- Risk notes (what could go wrong, edge cases)

For all tasks, also specify:
- Files to touch (explicit list)
- Whether it's parallel-safe (can it run alongside other tasks in the same phase?)

Confirm: "Does this task contract capture what you want? Anything to adjust?"

Move to the next milestone's tasks after confirmation.

## Step 4: Plan Sequencing

Once all milestones and tasks are confirmed, propose the phase/execution plan:

> "Here's how I'd sequence the work:
>
> **Phase 1: <name>**
> These need to happen first: <task list with brief rationale>
>
> **Phase 2: <name>**
> Once phase 1 is done: <task list>
> Parallel opportunity: <task-a> and <task-b> touch different files, no dependencies — can run simultaneously.
> Batching opportunity: <task-c>, <task-d>, <task-e> are sequential and simple — one subagent could handle all three.
>
> ...
>
> Dependencies:
> - <task-b> depends on <task-a> because <reason>
> - <task-d> depends on <task-c> because <reason>
>
> Does this match your instinct for how to approach the work?"

Adjust based on feedback.

## Step 5: Generate Files

Once everything is confirmed, generate all files:

1. **Update ROADMAP.md:**
   - Fill in vision and goals from the conversation
   - Set `sources:` to list all draft files that were read
   - Add all milestone names to `milestones:` list
   - Populate the milestone overview table

2. **Update MILESTONE.md:**
   - Add a section for each milestone with: status (pending), description, directory, task list, created date, status_override: null

3. **Create milestone directories:**
   - `.claude/roadmap/<milestone-name>/` for each milestone

4. **Create individual task files:**
   - Use `templates/task-standard.md` or `templates/task-complex.md` from the plugin's root `templates/` directory (the `claude-workflow/templates/` folder, not the skill's own folder)
   - Replace all `{{PLACEHOLDER}}` values with the confirmed content
   - Set `dependsOn` based on the dependencies identified in step 4
   - Set `phase` based on the phase plan
   - Set `parallelSafe` based on file overlap analysis
   - Set `filesTouch` based on the files identified for each task
   - **Naming convention:** `<milestone-name>-task<NNN>.md` — sequential numbering starting at 001

5. **Update TASK.md:**
   - Create phase sections with tables
   - Include columns: Task ID, Milestone, Status, Dependencies, Complexity, Parallel-Safe
   - Include parallelism and batching notes
   - Populate all task rows

Report completion as described in the main SKILL.md.
