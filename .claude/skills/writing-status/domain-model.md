# Work Activity Domain Model

Shared vocabulary for interpreting project work across sources (Slack, Jira, GitHub, Notion).

## Entities

**`Person`**
```
id    (Slack handle / Jira username / GitHub login)
name
```

**`WorkItem`** — stateful unit of work
```
title
owner   : Person
status  : NotStarted | InProgress | Done | Blocked | Abandoned
events  : Event[]
```

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

## Event

All state changes are events. A WorkItem's current status is the result of applying its events in order.

```
type       : StatusChanged | DecisionMade | DecisionSuperseded | Blocked | Unblocked
by         : Person
at         : timestamp
confidence : confirmed | inferred | conflicted
```

| Event.confidence | Meaning |
|---|---|
| `confirmed` | Directly stated in a single source |
| `inferred` | Deduced from context (e.g., PR merged but no Jira update) |
| `conflicted` | Two or more sources disagree on this event |

## Source Mapping

| Source signal | Event |
|---|---|
| Jira status transition, reopen | `StatusChanged` |
| GitHub PR merged or closed | `StatusChanged(→Done)` |
| Slack "we decided to..." | `DecisionMade` |
| Slack later correction or reversal | `DecisionSuperseded` |
| Slack or Jira blocker mention | `Blocked` |
| Slack or Jira blocker resolved | `Unblocked` |
| Google Calendar OOO or leave event | `Blocked` (subject unavailable) |
| Google Calendar event description link | follow link — may yield `DecisionMade` or meeting notes |
