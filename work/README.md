# Work

Active working documents organized by project. The latest version is the canonical final document.

Structure: `work/<project>/yyyy-MM-dd-<slug>-<type>-v<N>.md`

Example: `work/photo-review-cost-estimation/2026-04-10-llm-cost-estimation-report-v1.md`

## Document Types

| Type | Postfix | Description |
|------|---------|-------------|
| Report | `-report` | Research findings structured as Conclusion → Findings → Evidence |
| Review | `-review` | Structured findings against a document's stated purpose |
| Guide | `-guide` | Step-by-step operational instructions |
| Proposal | `-proposal` | Design proposal with problem, solution, and tradeoffs |
| Handoff | `-handoff` | Session state snapshot for cross-session continuity |

**Handoff versioning exception**: the date in a handoff filename reflects when it was written (the session date), and changes with each new version. All other types preserve the original creation date across revisions.

## Handoff Structure

A handoff file has two required sections written before work starts, and four optional sections written at session end.

```
## Goal
What the deliverable is for; intended audience; any constraints.

## Plan
- Question: [specific unknown being answered, for report tasks]
- Sources: [sources to check and why]
- Scope: in: ... / out: ...
- Type: [-report | -benchmark | -guide | -proposal | ...]
Status: draft | approved

## Decisions          ← added at session end if work is incomplete
## Traps              ← added at session end if work is incomplete
## Open Work          ← added at session end if work is incomplete
## New Session Prompt ← added at session end if work is incomplete
```

When the session ends with work remaining, write a new handoff version (today's date, incremented version) with the four session-end sections populated.

## Ownership

- **Writer**: Claude creates new version files. Human does not write here directly.
- **Reviewer**: Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments within the file.

## Versioning Rules

1. Claude never modifies existing version files directly.
2. Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments.
3. Claude incorporates the feedback into a new version file (`-v2`, `-v3`, ...).
4. Previous version files are preserved as-is (history).
