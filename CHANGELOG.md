# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0] - 2026-04-07

### Added
- Archive system — completed/cancelled milestones automatically move from active indexes (MILESTONE.md, TASK.md) into ARCHIVE.md, keeping active files proportional to current work
- Reactivate Milestone operation in `update-roadmap` — moves archived milestones back to active status
- Scoped reads in `work-task` — identifies active milestone from ROADMAP.md first, then reads only that milestone's data instead of loading everything
- Prior Work Brief capped at 3 most recent completed sibling tasks instead of reading all
- Error guards in `work-task` — missing roadmap directory, empty milestone table, and missing task contract files all produce actionable warnings instead of failing silently
- `status` skill shows archived milestones as compact one-line summaries in a "Completed" section

### Changed
- Standard task contract template trimmed — removed Context and Scope sections (kept in complex template). Standard contracts now have 4 sections: Objective, Instructions, Tests, Acceptance Criteria
- `build-roadmap` prompt distinguishes standard vs complex section lists when creating task contracts
- Removed static boilerplate from TASK.md and MILESTONE.md templates (status legend, explanatory comments)

## [0.2.1] - 2026-04-07

### Added
- `status` skill — read-only terminal dashboard showing milestone tables, progress bars, blocked items, and next-up section
- Contract validation gate in `work-task` — checks task contracts for required fields before dispatch
- Context injection in `work-task` — subagents receive project context, related file contents, and dependency outputs
- Test roadmap artifacts (orchestrator-hardening, status-dashboard milestones) from end-to-end validation run

### Fixed
- `package.json` skills array now includes all 6 skills (was missing `status` after initial v0.2.0)
- Trigger overlap between `status` and `work-task` skills clarified in descriptions

### Known Issues
- Milestone `review` state is skipped — milestones go directly from `in_progress` to `completed` instead of pausing at `review` for user confirmation
- Re-review cycle not enforced — when a subagent fix is applied after review feedback, work-task should dispatch a second review but currently marks complete immediately

## [0.2.0] - 2026-04-07

### Added
- `work-task` orchestrator skill — reads task graph, presents status, dispatches subagents on user approval
- `parallelSafe` and `filesTouch` fields in task templates for execution planning
- Instructions, Tests, and Risk Notes sections in task contract templates
- Parallel and batching opportunity notes in TASK.md template
- marketplace.json for local plugin installation

### Changed
- Task contracts redesigned for AI execution — prescriptive instructions instead of descriptions
- `build-roadmap` now produces AI-executable contracts with enforced naming convention
- `update-task` reclassified as internal plumbing (not user-invoked)
- `update-roadmap` now produces AI-executable task contracts
- Removed "next step" messaging from skill completion outputs
- README updated to reflect orchestrator model

## [0.1.0] - 2026-04-07

### Added
- Plugin scaffolding (package.json, plugin.json, README, LICENSE)
- File templates for ROADMAP.md, MILESTONE.md, TASK.md, and task contracts (standard + complex)
- `setup-roadmap` skill — initialize roadmap directory structure
- `build-roadmap` skill — conversational flow to transform brainstorm drafts into formal roadmap
- `update-roadmap` skill — structural changes (add/cancel milestones, add tasks, reorder phases)
- `update-task` skill — status changes with cascade updates across all index files
- Session-start hook for roadmap detection and skill availability reminders
