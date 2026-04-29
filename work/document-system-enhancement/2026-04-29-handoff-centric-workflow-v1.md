<!-- Sources: prior analysis — conversation, 2026-04-29 -->

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

| Section | Owner | Content |
|---------|-------|---------|
| Goal | Human | What the deliverable is for; audience; constraints |
| Plan | Claude | Research approach, sources, output format |
| Decisions | Claude | Choices made during work and why |
| Traps | Claude | Approaches tried and rejected |
| Open Work | Claude | Current status — declarative, not imperative |
| New Session Prompt | Claude | Entry point for the next session |

Human writes Goal before starting. Claude populates the rest during work. The approval gate in writing-report is unchanged: it moves from "approve the plan file" to "approve the Plan section."

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
