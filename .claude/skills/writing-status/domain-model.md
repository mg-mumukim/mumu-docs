# Work Activity Domain Model

Shared vocabulary for interpreting project work across sources (Slack, Jira, GitHub, Notion).

## Entities

**`Person`**
```
id    (Slack handle / Jira username / GitHub login)
name
role  : Lead | TL | PM | MLE | MLSE | Staff-SWE | Designer | Advisor | Counterpart
org   : internal | external
source : prior_handoff | source_doc | inferred_from_context
```

`role` guidance:
- `Lead` — project / squad / pod lead; there is exactly one Lead per project
- `TL` — tech lead; advises on architecture and engineering direction
- `Counterpart` — person from an external team who is the primary contact for a dependency; not an owner of any WorkItem in this project
- `org = external` applies to anyone outside the squad (different team, different company)

`source` guidance:
- `prior_handoff` — role and org taken verbatim from the prior handoff People table; treat as `confirmed`
- `source_doc` — role or org explicitly stated in a source document (meeting note, Jira, Slack); treat as `confirmed`
- `inferred_from_context` — role or org deduced from session context (e.g., Git user, document author, file creator) with no corroborating source; treat as `inferred`. MUST surface in Stage 3 for confirmation before writing. The project `Lead` MUST NOT be assigned via session context alone — always read from prior handoff or confirmed source.

**`WorkItem`** — stateful unit of work
```
title
owner      : Person
status     : NotStarted | InProgress | Done | Blocked | Skipped | Deferred
blockedBy  : WorkItem[]   // hard: cannot start/complete until these are Done
riskFactor : string?      // soft: can proceed but quality or scope may be affected; not a hard block
iteratesOn : WorkItem?    // this item is a direct successor/iteration of another (e.g., Eval 5 iteratesOn Eval 4)
                          // confidence is almost always inferred — set from naming pattern (e.g., Eval N follows Eval N-1)
                          // no source explicitly states this relationship; always surface as a Stage 3 confirmation
events     : Event[]
```

`blockedBy` is only set when explicitly stated in a source or confirmed by the user. Temporal proximity alone (two items with the same target date) does not constitute a hard dependency. If a relationship is inferred but not confirmed, record it in `riskFactor` and surface it as a Stage 3 question.

**`Decision`** — directional agreement, reversible
```
statement
by           : Person
rationale    : string?    // why this direction was chosen; key constraints or tradeoffs that drove it
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
at         : timestamp    // granularity varies by source: Slack/GCal = exact datetime; Notion/GDoc = date only
reason     : string?      // why this event occurred; required for SkippedEvent, DeferredEvent, Blocked
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

**Status and state changes**

| Source signal | Entity / Event |
|---|---|
| Jira status transition, reopen | `StatusChanged` — treat as low-confidence baseline; Slack/Notion may reflect a more recent state |
| GitHub PR merged or closed | `StatusChanged(→Done)` |
| Slack "we decided to..." | `DecisionMade` |
| Slack later correction or reversal | `DecisionSuperseded` |

**Note:** Jira `issuelinks` is frequently empty in practice. Do NOT rely on Jira for `DependencyAdded` / `blockedBy`. Derive dependencies from meeting notes, Slack, and Stage 3 user confirmation only.

**Blockers and availability**

| Source signal | Entity / Event |
|---|---|
| Slack or Jira explicit blocker mention | `Blocked` |
| Slack or Jira blocker resolved | `Unblocked` |
| Google Calendar `eventType=outOfOffice` or leave event | `Blocked` (subject unavailable) |
| Notion/Slack text: "휴가", "부재", "vacation", "OOO", "returns [date]", "[name] is out" | `Blocked` (confidence = confirmed if date given, inferred otherwise) — GCal OOO may not be visible cross-person |

**Skip, defer, and milestone**

| Source signal | Entity / Event |
|---|---|
| "Skipping X", "X를 skip하기로", "we're not doing X this week" | `SkippedEvent` — `reason` is required; if absent, surface in Stage 3 |
| "Deferring X", "X를 다음 주로", "next sprint" | `DeferredEvent` — `reason` is required; if absent, surface in Stage 3 |
| "Timeline: A → B → C" or "target: date" | `MilestoneSet` per date anchor |

**Rationale and alternatives**

| Source signal | Entity / Event |
|---|---|
| English: "because...", "due to...", "in order to...", "so that..." | `Event.reason` or `Decision.rationale` |
| Korean: "때문에", "로 인해", "을 위해", "이유", "맥락", "목적" | `Event.reason` or `Decision.rationale` |
| GDoc with comparison sections (e.g., "Option 1 vs Option 2", trade-off tables) | `Decision.alternatives` — GDoc is the primary source for alternatives; collect before meeting notes |
| Decision with no rationale in any source | Surface in Stage 3: ask the user directly |

**Iteration and sequence**

| Source signal | Entity / Event |
|---|---|
| WorkItem title follows naming pattern (e.g., "Eval 5" after "Eval 4") | `WorkItem.iteratesOn` (confidence = inferred) — always surface in Stage 3 for confirmation |

**People and roles**

| Source signal | Entity / Event |
|---|---|
| "X is the lead", "lead: X" | `Person.role = Lead`, `source = source_doc` |
| "TL: X", "tech lead: X" | `Person.role = TL`, `source = source_doc` |
| "X is our counterpart at Y", "Tinder PM: X" | `Person.role = Counterpart`, `org = external`, `source = source_doc` |
| GCal event attendees list | `Person` entities — highest-quality source for name, email, org; `source = source_doc` |
| Prior handoff People table | All fields verbatim; `source = prior_handoff` — do not re-derive from other signals |
| Git user / document author / session context alone | `Person.source = inferred_from_context` — do not assign `Person.role = Lead` from this signal alone; MUST surface in Stage 3 for confirmation |
