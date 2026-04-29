<!-- Sources: prior analysis — conversation, 2026-04-29 -->

## Changes from v1

- Added concrete example of a filled-in handoff.md using this task as the case

---

# Document System Enhancement: Handoff-Centric Workflow

## Problem

The current system has two static task file types in `note/task/`:

- `*-request.md` — what to build (input for writing-draft)
- `*-report-plan.md` — how to research (gated input for writing-report)

Both are write-once. They capture intent before work starts but do not record what was decided during a session, what approaches were rejected, or where work stopped. Resuming any topic requires reconstructing state from the latest versioned output alone.

## Proposal

Replace the two static task file types with one **living handoff document** per topic, stored alongside the versioned deliverables:

```
work/<topic>/handoff.md     mutable, joint human + Claude ownership
work/<topic>/*-vN.md        immutable versioned deliverables (unchanged)
```

The handoff document is simultaneously a task spec, a research plan, and a session state record.

## Handoff Document Structure

A handoff.md has six sections. Human writes Goal. Claude writes the rest during work.

```markdown
## Goal
(human writes before starting)
What the deliverable is for; intended audience; any constraints.

## Plan
(Claude writes; human approves before work starts)
Which sources to check, research approach, expected output.
Status: draft | approved

## Decisions
(Claude updates during work)
Each decision made and the reason.

## Traps
(Claude updates during work)
Approaches tried and rejected, and why.

## Open Work
(Claude updates during work)
What is not yet done — state only, no imperatives.

## New Session Prompt
(Claude writes at session end)
Copy-paste entry point to resume in a new session.
```

### Example: this task

The following is what `work/document-system-enhancement/handoff.md` would look like at the end of the current session.

```markdown
## Goal
Design a handoff-centric workflow that replaces static note/task/ files with
living handoff documents. Audience: Mumu (system designer). No deadline.

## Plan
- Review current skill definitions: writing-report, writing-draft,
  writing-review, resolving-docs
- Map note/task/ file types and their friction points
- Propose where handoff.md should live (note/ vs work/)
- Output: proposal document (work/document-system-enhancement/)
Status: approved

## Decisions
- Store handoff.md in work/<topic>/, not note/task/.
  Reason: note/ is source material (what you know, immutable);
  handoff is working state (what you're doing, needs to evolve).
- handoff.md is mutable; *-vN.md files remain immutable.
  Requires a carve-out in writing-convention.md.

## Traps
- Putting handoff.md in note/task/ — conflicts with the immutability rule
  for note/; would require relaxing semantics across all note/ files.
- Keeping -request.md and -report-plan.md alongside handoff.md — creates
  overlap without solving session continuity.

## Open Work
- writing-report skill is not yet updated to create/update handoff.md.
- writing-draft skill is not yet updated to read handoff.md first.
- writing-convention.md does not yet include the handoff.md carve-out.

## New Session Prompt
Read work/document-system-enhancement/handoff.md. The proposal is at
work/document-system-enhancement/2026-04-29-handoff-centric-workflow-v2.md.
Next step: update writing-report and writing-draft skills to use handoff.md.
Read both skill files before proceeding, then wait for instructions.
```

## Skill Changes

**writing-report**: Step 2 creates `work/<topic>/handoff.md` (Goal + Plan sections) instead of `note/task/*-report-plan.md`. Claude updates Decisions and Traps during research.

**writing-draft**: Step 2 checks `work/<topic>/handoff.md` first. If present, Claude reads Open Work + Decisions instead of reconstructing intent from sources.

**writing-review**: No change for single-session reviews.

## Required Rule Change

`writing-convention.md` currently states: MUST NOT modify existing version files in `work/`.

Add a carve-out: `handoff.md` in any `work/<topic>/` directory is a mutable work-in-progress artifact. Only `*-vN.md` files are subject to the immutability rule.

## Tradeoffs

| | Current | Proposed |
|--|---------|----------|
| Files per topic | request + plan + vN files | handoff.md + vN files |
| Session resumption | Reconstruct from latest version | Read handoff.md |
| Decision history | Not preserved | Preserved in Decisions + Traps |
| work/ mutability | Uniform (all immutable) | Mixed (handoff mutable, versions immutable) |

The main cost is mixed mutability within `work/<topic>/`. The benefit is that session state has a persistent, unambiguous home.
