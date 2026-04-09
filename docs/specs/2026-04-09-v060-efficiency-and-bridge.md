# v0.6.0 — Token Efficiency, Superpowers Bridge, and Bug Fixes

**Date:** 2026-04-09
**Status:** Draft — ready for /build-roadmap

## Context

After an adversarial review (Codex) and token efficiency audit of v0.5.0, we identified three categories of work: token bloat that crept in since the v0.3.0 efficiency hardening, a missing integration bridge with the superpowers plugin, and several real bugs in skill logic.

This spec covers all three. The goal is a leaner, more correct v0.6.0 that plays nicely with superpowers for users who have both plugins.

---

## Milestone 1: Token Trimming

Reduce total skill token footprint without removing any operational logic. Only formatting verbosity, cross-skill repetition, and redundant framing are targets.

### 1a. Status Skill Compression

The status skill is ~235 lines. Target: ~120-130 lines.

**What to trim:**
- Remove the full "Complete Example" section at the bottom (the inline format examples in each section are sufficient)
- Remove the "When to Use" section (frontmatter description covers it; "read-only" note is restated in Output Rules)
- Compress progress bar rules to 2 lines (keep the Unicode codepoints and 10-char width — those are needed for consistency)
- Compress completed milestones rendering rules (keep archive parsing logic — where to find name, task count, date in ARCHIVE.md — but cut formatting details like singular/plural task grammar)
- Compress active milestone detail rules (keep format example and dep abbreviation hint, cut the alignment/truncation prose)
- Deduplicate Output Rules (remove "no ANSI" and "read-only" which are already stated elsewhere; keep "fit in a terminal" as a single line)

**What NOT to trim:**
- Step 1 (file reads + empty state guards) — prevents bad output
- Step 2 (parse logic) — already concise
- Step 4 (ROADMAP.md sync) — real logic with specific field mappings
- Step 5 (drafts notice) — already short

### 1b. Build-Roadmap File Merge

Currently split across SKILL.md (~70 lines) and build-roadmap-prompt.md (~150 lines) = ~230 lines loaded into context. These overlap on task contract requirements, naming conventions, and checkpoint behavior.

**Action:** Merge into a single SKILL.md. Deduplicate where the two files restate the same concepts. Target: ~120 lines.

### 1c. Cross-Skill Deduplication

These concepts are restated in multiple skill files. Define each once in its authoritative location, use short references elsewhere.

| Concept | Authoritative Location | Currently Also In |
|---------|----------------------|-------------------|
| Task contract requirements (Instructions, Tests, AC, filesTouch) | build-roadmap | work-task, update-task |
| Archive mechanism (format, cascade, move logic) | update-task | status, design spec |
| Milestone status derivation rules | update-task | status |
| Task naming convention (`<milestone>-task<NNN>`) | build-roadmap | update-roadmap, update-task |

**For each:** Keep the full description in the authoritative skill. In other skills, replace with a 1-line reference like "Task contracts must meet the requirements defined in build-roadmap" or "Archive format follows update-task cascade rules."

### 1d. Guard Clause and Framing Cleanup

- **Pre-check guards:** 5 skills each have 3-5 line "no roadmap found" blocks with slightly different wording and suggestions. Shorten each to 1 line. The session-start hook already handles navigation.
- **Philosophical framing:** Remove lines like "Co-pilot, not autopilot", "This is a conversational skill, not a generator", "This is a read-only overview" — these describe intent but don't instruct behavior. The actual step-by-step instructions already enforce the right behavior.

---

## Milestone 2: Superpowers Bridge

Enable build-roadmap to consume specs and plans created by the superpowers plugin, without modifying superpowers or creating coupling.

### 2a. Multi-Source Scanning in Build-Roadmap

Update build-roadmap's input scanning step to check three locations:

1. `.claude/roadmap/drafts/` — workflow-framework drafts (existing)
2. `docs/superpowers/specs/` — superpowers brainstorm specs (new)
3. `docs/superpowers/plans/` — superpowers plans (new)

**Behavior:**
- If a directory doesn't exist, skip it silently (no error)
- Present findings grouped by source so the user sees where each item came from
- User picks which items to formalize — don't auto-consume everything

### 2b. Source Tracking (Duplication Guard)

The `sources:` field in ROADMAP.md frontmatter records which input files were consumed during roadmap building. This prevents build-roadmap from re-presenting specs that were already formalized.

**Behavior:**
- After building milestones from a spec/draft, add its path to `sources:` in ROADMAP.md
- On future scans, filter out any file whose path is already in `sources:`
- This works across all three input locations (drafts, specs, plans)

### 2c. Draft Skill Stays Independent

No changes to `/draft`. It remains the lightweight quick-capture path for ideas that don't need a full superpowers brainstorming session. Drafts are consumed by build-roadmap the same way — tracked in `sources:` after use.

---

## Milestone 3: Bug Fixes

### 3a. Stale Design Spec

`docs/specs/2026-04-07-context-efficiency-hardening-design.md` still says work-task reads ROADMAP.md. This contradicts the v0.5.0 skill where work-task explicitly does NOT read ROADMAP.md.

**Fix:** Update the spec to reflect v0.5.0 reality, or add a header marking it as superseded by the v0.5.0 implementation.

### 3b. Milestone Review State Exit Path

When all tasks in a milestone are completed or cancelled, update-task derives milestone status as `review`. But nothing transitions `review` → `completed`:
- work-task doesn't check for milestones in `review` state
- update-task has no promotion logic
- status only displays state

**Fix:** Add instructions to work-task: after the last task in a milestone completes review (Step 6), check if the milestone derived status is `review`. If so, present the user with a milestone-level completion prompt. On approval, update milestone status to `completed` via update-task cascade (which triggers archival).

### 3c. Reviewer Dispatch Context

work-task Step 6 says to "spin up a reviewer subagent" but doesn't specify what context the reviewer receives.

**Fix:** Specify: "Dispatch reviewer with: the task contract file, a git diff of changes since dispatch began, and test output."

### 3d. DependsOn Validation

work-task Step 4 validates task contracts (Instructions, Tests, AC, filesTouch) but doesn't check that `dependsOn` references point to task IDs that actually exist in TASK.md. A typo silently blocks a task forever.

**Fix:** Add to Step 4 validation: "Verify all dependsOn task IDs exist in TASK.md. Warn the user if any reference is missing or misspelled."

---

## Milestone 4: Cross-Milestone Parallelism (Future — Not This Release)

Documenting for future planning only.

work-task currently picks only the first `in_progress` or `pending` milestone. Independent milestones (no cross-dependencies) cannot be worked in parallel.

**Design considerations:**
- Need to determine milestone-level dependencies (not just task-level)
- work-task Step 1 would need to identify all "ready" milestones, not just the first
- Parallel dispatch across milestones adds complexity to the review and archive flows
- agent-cc's workflow bridge may have assumptions about single-active-milestone — need to verify before changing

This is a design change, not a bug fix. Requires its own brainstorming pass.

---

## Integration Notes

### Agent-CC Interaction

This plugin integrates with the agent-cc dashboard project. Changes that affect file formats or status values need to be verified against agent-cc's workflow bridge.

**Safe changes (no agent-cc impact):**
- All token trimming (Milestone 1) — agent-cc doesn't read skill files
- Build-roadmap bridge (Milestone 2) — adds to ROADMAP.md frontmatter which agent-cc should eventually read
- B2 stale spec, B4 reviewer context, B5 dependsOn validation

**Needs agent-cc verification:**
- B3 milestone review exit — agent-cc may map milestone statuses. Adding the review→completed transition shouldn't break anything (it's adding a path, not changing one), but worth confirming the workflow bridge handles `review` status if it encounters it.

### Superpowers Plugin

No changes to superpowers. Workflow-framework reads superpowers output locations as an optional input source. If superpowers isn't installed or hasn't been used, those directories don't exist and scanning is silently skipped.
