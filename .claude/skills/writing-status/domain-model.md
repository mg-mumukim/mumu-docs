# Work Activity Domain Model

Shared vocabulary for interpreting project work across sources (Slack, Jira, GitHub, Notion).

## Entities

**`Person`**
```
id    (Slack handle / Jira username / GitHub login)
name
role  : Lead | TL | PM | MLE | MLSE | Staff-SWE | Designer | Advisor | Counterpart
org   : internal | external
```

`role` guidance:
- `Lead` — project / squad / pod lead; there is exactly one Lead per project
- `TL` — tech lead; advises on architecture and engineering direction
- `Counterpart` — person from an external team who is the primary contact for a dependency; not an owner of any WorkItem in this project
- `org = external` applies to anyone outside the squad (different team, different company)

**`WorkItem`** — stateful unit of work
```
title
owner      : Person
status     : NotStarted | InProgress | Done | Blocked | Skipped | Deferred
blockedBy  : WorkItem[]   // hard: cannot start/complete until these are Done
riskFactor : string?      // soft: can proceed but quality or scope may be affected; not a hard block
events     : Event[]
```

`blockedBy` is only set when explicitly stated in a source or confirmed by the user. Temporal proximity alone (two items with the same target date) does not constitute a hard dependency. If a relationship is inferred but not confirmed, record it in `riskFactor` and surface it as a Stage 3 question.

**`Decision`** — directional agreement, reversible
```
statement
by           : Person
status       : Adopted | Rejected | Superseded | Unresolved
supersededBy : Decision?
```

| Decision.status | Meaning |
|---|---|
| `Adopted` | Agreed and in effect |
| `Rejected` | Considered but not taken |
| `Superseded` | Replaced by a later decision |
| `Unresolved` | Discussion happened but no clear conclusion reached |

**`Milestone`** — a concrete date commitment for a deliverable or phase gate
```
description : string
date        : ISO date
status      : Planned | Confirmed | Slipped | Met
owner       : Person?
```

Milestones capture timeline information: release targets, kick-off dates, phase gates. Collect them from Notion, Google Docs, and Slack whenever a message contains a date-anchored plan (e.g., "Timeline: X → Y → Z" or "target: 5/6").

## Event

All state changes are events. A WorkItem's current status is the result of applying its events in order.

```
type       : StatusChanged | DecisionMade | DecisionSuperseded | Blocked | Unblocked
           | SkippedEvent | DeferredEvent | MilestoneSet
by         : Person
at         : timestamp
confidence : confirmed | inferred | conflicted
```

| Event.type | Meaning |
|---|---|
| `StatusChanged` | Work item moved to a new status |
| `DecisionMade` | A directional decision was reached |
| `DecisionSuperseded` | An earlier decision was reversed or replaced |
| `Blocked` | Work item or person became unavailable |
| `Unblocked` | Blocker was resolved |
| `SkippedEvent` | A planned item was deliberately skipped — reason should be captured |
| `DeferredEvent` | A planned item was pushed to a later period — new target date if known |
| `MilestoneSet` | A concrete date commitment was established or updated |
| `DependencyAdded` | A `blockedBy` relationship was established between two WorkItems |
| `DependencyResolved` | A blocking WorkItem completed, unblocking the dependent item |

## Artifact Handoff

When a substream delivers an artifact (e.g., a KB, dataset, or design doc) to a mainstream, the substream's ownership of that artifact ends. Any subsequent decisions about how to use or adapt the artifact belong to the consuming mainstream — they are not the substream's `blockedBy`. The substream WorkItem is `Done` from this point.

## Scope Alignment

A `blockedBy` relationship is only meaningful when both WorkItems target the same delivery scope (e.g., both in the MVP release). A research WorkItem targeting post-MVP outcomes does not create an operational dependency on MVP delivery WorkItems in the same period — they run on separate tracks. Do not model cross-scope relationships as `blockedBy`; note them as context in the research WorkItem's description.

| Event.confidence | Meaning |
|---|---|
| `confirmed` | Directly stated in a single source |
| `inferred` | Deduced from context (e.g., PR merged but no Jira update) |
| `conflicted` | Two or more sources disagree on this event |

## Source Mapping

| Source signal | Entity / Event |
|---|---|
| Jira status transition, reopen | `StatusChanged` |
| GitHub PR merged or closed | `StatusChanged(→Done)` |
| Slack "we decided to..." | `DecisionMade` |
| Slack later correction or reversal | `DecisionSuperseded` |
| Slack or Jira blocker mention | `Blocked` |
| Slack or Jira blocker resolved | `Unblocked` |
| "Skipping X", "we're not doing X this week" | `SkippedEvent` on that WorkItem |
| "Deferring X to next week / next sprint" | `DeferredEvent` on that WorkItem |
| "Timeline: A → B → C" or "target: date" | `MilestoneSet` per date anchor |
| Google Calendar OOO or leave event | `Blocked` (subject unavailable) |
| Google Calendar event description link | follow link — may yield `DecisionMade` or meeting notes |
| "X is the lead", "lead: X" | `Person.role = Lead` |
| "TL: X", "tech lead: X" | `Person.role = TL` |
| "X is our counterpart at Y", "Tinder PM: X" | `Person.role = Counterpart`, `org = external` |
