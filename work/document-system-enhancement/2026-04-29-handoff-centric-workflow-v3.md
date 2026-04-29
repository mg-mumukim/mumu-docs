<!-- Sources: prior analysis — conversation, 2026-04-29 -->

## Changes from v2

- Handoff files are immutable and versioned, consistent with deliverables. Rule carve-out for mutability removed.
- Naming convention clarified: date on a handoff reflects when it was written (session date), not the original topic creation date.
- Added "When to Write" section covering trigger points and who writes.

---

# Document System Enhancement: Handoff-Centric Workflow

## Problem

The current system has two static task file types in `note/task/`:

- `*-request.md` — what to build (input for writing-draft)
- `*-report-plan.md` — how to research (gated input for writing-report)

Both are write-once. They capture intent before work starts but do not record what was decided during a session, what approaches were rejected, or where work stopped. Resuming any topic requires reconstructing state from the latest versioned output alone.

## Proposal

Replace the two static task file types with **versioned handoff documents** stored alongside the deliverables:

```
work/<topic>/
  yyyy-MM-dd-<slug>-handoff-v<N>.md    session snapshots
  yyyy-MM-dd-<slug>-<type>-v<N>.md     deliverables
```

Both follow the same immutability rule: once written, a file is never modified. A new version is created for each update.

One naming difference between handoff and deliverable files: the date on a handoff reflects when it was written (the session date), and changes with each new version. Deliverable versions preserve the original creation date.

## When to Write

Write a handoff when work on a topic will continue in a new session.

| Trigger | Writer |
|---------|--------|
| Starting a new multi-session topic | Human (Goal only) or Claude (full v1) |
| Session ending with work remaining | Claude |
| Before /clear | Claude |

Single-session tasks with no continuation do not need a handoff. The latest handoff version is the entry point for any new session on that topic.

## Handoff Document Structure

A handoff has six sections. Human writes Goal when initiating a topic. Claude writes the rest.

```markdown
## Goal
What the deliverable is for; intended audience; any constraints.

## Plan
Which sources to check, research approach, expected output.
Status: draft | approved

## Decisions
Each decision made during work and the reason.

## Traps
Approaches tried and rejected, and why.

## Open Work
What is not yet done — state only, no imperatives.

## New Session Prompt
Copy-paste entry point to resume in a new session.
```

### Example: this task

The following is what `work/document-system-enhancement/2026-04-29-handoff-centric-workflow-handoff-v1.md` would look like at the end of the current session.

```markdown
## Goal
Design a handoff-centric workflow that replaces static note/task/ files with
versioned handoff documents. Audience: Mumu (system designer). No deadline.

## Plan
- Review current skill definitions: writing-report, writing-draft,
  writing-review, resolving-docs
- Map note/task/ file types and their friction points
- Propose where handoff files should live and how they are versioned
- Output: proposal document (work/document-system-enhancement/)
Status: approved

## Decisions
- Store handoff files in work/<topic>/, not note/task/.
  Reason: note/ is source material (immutable); handoff is working state.
- Handoff files are immutable and versioned, same as deliverables.
  Reason: a session written in a degraded state should not overwrite prior
  state with no rollback; history of session snapshots has value.
- Date on handoff reflects session date (changes per version), not original
  creation date. Reason: the date is meaningful as "when this snapshot was
  taken," not "when the topic started."

## Traps
- Mutable handoff.md — no rollback when a session writes a bad snapshot.
- Putting handoff in note/task/ — conflicts with note/ immutability semantics.
- Keeping -request.md and -report-plan.md alongside handoff — overlap without
  solving session continuity.

## Open Work
- writing-report skill not yet updated to create handoff-v1.md at session start.
- writing-draft skill not yet updated to find and read latest handoff first.
- resolving-docs not yet updated to recognize handoff file pattern.
- writing-convention.md not yet updated with handoff file naming rule.

## New Session Prompt
Read work/document-system-enhancement/2026-04-29-handoff-centric-workflow-handoff-v1.md.
The proposal is at work/document-system-enhancement/2026-04-29-handoff-centric-workflow-v3.md.
Next step: update writing-report, writing-draft, and resolving-docs skills to
use handoff files. Read all three skill files before proceeding, then wait for
instructions.
```

## Skill Changes

**writing-report**: Step 2 creates `work/<topic>/yyyy-MM-dd-<slug>-handoff-v1.md` (Goal + Plan) instead of `note/task/*-report-plan.md`. The approval gate is unchanged. At session end, Claude writes a new handoff version with updated sections.

**writing-draft**: Step 2 checks `work/<topic>/` for the latest `*-handoff-v<N>.md` first. If present, Claude reads Open Work + Decisions instead of reconstructing intent from sources.

**writing-review**: No change for single-session reviews.

**resolving-docs**: When resolving a topic directory, also glob `work/<topic>/*-handoff-v*.md` and return the latest version as the session entry point.

## Tradeoffs

| | Current | Proposed |
|--|---------|----------|
| Files per topic | request + plan + vN files | handoff-vN + deliverable-vN files |
| Session resumption | Reconstruct from latest version | Read latest handoff |
| Decision history | Not preserved | Preserved in Decisions + Traps |
| work/ mutability | Uniform (all immutable) | Uniform (all immutable) |

Handoff file count grows at one per session, which is slower than deliverable versions typically accumulate. No special immutability rule is required.
