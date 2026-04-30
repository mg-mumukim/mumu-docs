---
name: writing-status
description: Produces or revises a shared status update document — for team meetings, manager reports, or stakeholder briefings — covering what a person or team did on a project over a time period. Writes to `work/<topic>`. Activates on "[팀] [프로젝트] 현황 보고", "지난 [기간] [팀] 뭐 했는지 정리", "상황 정리해서 보고해줘", "다시 정리해줘", "섹션 재구성해줘", "[person] status on [project]", "현황 보고", "주간 업데이트", "weekly update", "restructure the update" and similar. Output is always meant for sharing with others. Does not trigger for: drafting proposals, writing research reports, reviewing documents, or personal reference notes.
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

Reference: [domain-model.md](domain-model.md) — shared vocabulary for Person, WorkItem, Decision, and Event used in Stages 2–4.

# Inputs

Confirm before starting research. If any of the following are missing, ask once:

- **Subject**: person name or team name
- **Project / scope**: which project or workstream
- **Time range**: start and end date (absolute)
- **Output audience**: team meeting, manager report, or stakeholder update — this skill is for shared output only

## 1. Establish the format

If the user provided a reference document or URL, fetch it and extract: section structure, format (table vs bullets), detail level, and language. Use that as the output format.

If no reference is provided, use the default format: `-` bullets for work items; `>` blockquotes for resolved Decisions; nested bullets for sub-items.

No confirmation needed for format unless the user explicitly asks to verify it.

## 2. Propose section structure (team subjects only)

If the subject is a team or multiple people, propose work-stream-based sections — not role-based or person-based sections. A stream groups members who are tightly or loosely coupled toward a single primary deliverable.

Identify sections as follows:
1. Find the primary streams by member coupling and work item count — typically 2, but may be 1 or 3. Label them **Mainstream 1**, **Mainstream 2**, etc.
2. Name remaining output tracks as **Substream** (smaller, fewer members).
3. If members exited mid-period, plan a brief "Exited this period" note at the bottom — not a standalone section.

For each section, list: stream name | PoC | one-line scope. The project Lead (`Person.role = Lead`) is NOT listed as a stream PoC — stream PoC is whoever owns the deliverable for that stream. External counterparts (`Person.org = external`) are listed separately as "Tinder counterpart:" or similar, never as PoC. If the handoff People table includes a `Lead` or `Advisor` role, the document header SHOULD state **Lead:** [name] and **Advisor:** [name].

Sections SHOULD be MECE at the top level. For cross-cutting items that span multiple streams, apply the deduplication rule in Signal triage rather than forcing an artificial MECE boundary.

MUST get explicit user confirmation before beginning research.

If the subject is a single person, skip this step.

## 3. Collect all sources

Before running Stage 1, search `work/` for an existing handoff file for this project (`*-<project-slug>-handoff-*.md`) using **resolving-docs**.

If found, present a summary of its Coverage, Stage 1 Anchors, and Unresolved Items, then ask the user: **reuse as-is**, **revise**, or **start fresh**.

<HARD-GATE> Do not begin Stage 1 until the user responds. </HARD-GATE>

Based on the user's choice:
- **Reuse**: load Stage 1 anchors, unresolved items, and coverage time range directly; skip Stage 1
- **Revise**: write a new handoff version first; proceed with full Stage 1
- **Start fresh**: ignore the existing handoff; proceed with full Stage 1

If no handoff exists, proceed with full Stage 1.

### Stage 1: Identify anchors

**Parallel fan-out (Agent tool):** Launch one subagent per source in a single message. Each subagent finds its anchor and returns it, or marks it unavailable.

| Source | Anchor to find | Fallback if unknown |
|---|---|---|
| Slack | Project-dedicated channel name | Search by project name via Glean |
| Jira | Project key | Search by project name in Jira |
| GitHub | Relevant repo(s) | Search by project name within org |
| Notion | Relevant space or page title | Keyword search by project name |
| Google Calendar | Subject's email address | Check `list_calendars()` output |

<HARD-GATE> Do not begin Stage 2 until all anchors are confirmed or marked unavailable. </HARD-GATE>

### Stage 2: Collect — breadth first, then follow links

Before fetching, use **resolving-docs** to check `note/<project-scope>/` for existing source notes (`*-slack.md`, `*-jira.md`, `*-github.md`, `*-notion.md`, `*-gcal.md`). Read files already present; fetch only missing sources. **Parallel fan-out (Agent tool):** Launch one subagent per missing source in a single message. Each subagent fetches its source, saves to `note/<project-scope>/`, and returns a one-line summary. Wait for all subagents to complete before proceeding. Collect each source in chronological order.

**Source priority (most current → most structural):**
1. **Slack** — project channel + subject's messages within time range. Read full threads; capture both the original statement and any later corrections or reversals.
2. **Notion / Google Docs / meeting notes** — meeting notes within time range. Include later entries that amend or override earlier decisions. **Exception: meeting notes dated at or after the coverage end date (e.g., a PM-TL sync on the final day of coverage) supersede all other sources including the handoff — collect these first and let them override earlier data for recent decisions.**
3. **Jira** — tickets where subject is assignee, updated within time range. Include changelog: status transitions, reassignments, reopens.
4. **GitHub** — PRs authored by subject, merged or closed within time range. Include review iterations and revert commits.
5. **Google Calendar** — call `list_events(calendarId=subject_email, startTime, endTime, orderBy=startTime)` for each subject. Collect:
   - Meeting events: title, attendees with response status, description. Extract any linked doc/Jira/Notion URLs from description and add to the internal link follow queue.
   - OOO and leave events: `eventType=outOfOffice` or events created by the org's HR system account — record as periods of subject unavailability (`Blocked`). Also check Notion/Slack text for absence mentions (see domain-model.md Source Mapping).
   - For team subjects, launch one subagent per member in a single message (Agent tool).

**Save source notes:** After fetching each source, save a local copy to `note/<project-scope>/` as `yyyy-MM-dd-<slug>-<source-type>.md` (e.g. `-slack.md`, `-jira.md`, `-github.md`, `-notion.md`, `-gcal.md`).

**Follow internal links — one level deep:** If a Slack message links to a doc, a Jira ticket links to a PR, or a PR links to a spec, follow it. Only follow links from already-collected sources. Skip links outside the time range or to unrelated projects.

**Sequence rule:** Collect the full history, not just the current snapshot — final state and the corrections or reversals that led to it.

**Map to domain events:** As you collect, translate each raw signal to a domain event using the source mapping in [domain-model.md](domain-model.md). Build an event sequence per WorkItem. Do not write until the event sequences are assembled.

<HARD-GATE> Do not begin Stage 3 until all Stage 2 queries and link traversals are complete. </HARD-GATE>

If the user is actively sharing additional links or context, ask "반영할 내용이 더 있으신가요?" before drafting.

### Stage 3: Resolve uncertainties

Scan the assembled event sequences for items that need clarification:
- Events with `confidence = conflicted` or `inferred`
- Decisions with `status = Unresolved`

Classify each into one question type and ask one question per message:

**Quick-answer** — the user likely knows the answer immediately:
- Source conflict: "Source A says X, source B says Y — what actually happened?"
- Inconclusive discussion: "There was a discussion about Z but no clear conclusion — was a decision reached?"

**Investigation-needed** — the trail is missing; ask for a pointer:
- "I can't find what happened with [item] — do you have a Jira ticket or Slack thread I can look at?"

After each answer:
1. Update the relevant WorkItem's event sequence: set `confidence = confirmed`, resolve `Decision.status`.
2. Cross-check the updated sequence against all source data already in context — surface any new conflicts introduced by the answer.
3. If new `conflicted` or `Unresolved` items surface, continue the loop.

**Exit condition:** No `conflicted` events remain, and all `Unresolved` decisions are either resolved or explicitly acknowledged as ongoing. If the user passes on a question, mark that item unresolvable and carry it to Step 6 as a flag.

**Confirmation checklist** — in addition to `conflicted` events and `Unresolved` decisions, scan for each of the following and surface as Stage 3 questions if found:

- **Inferred `blockedBy`**: relationship derived from temporal proximity or context, not explicitly stated in a source → confirm before writing as hard block; if unconfirmed, write as `riskFactor`
- **Inferred `iteratesOn`**: set from naming pattern only (e.g., Eval 5 follows Eval 4) — `confidence = inferred`; surface via the standard inferred-event scan above; no source explicitly states this relationship
- **Missing `Event.reason`**: a `SkippedEvent`, `DeferredEvent`, or `Blocked` event has no `reason` → ask why before writing
- **Missing `Decision.rationale`**: a Decision has no rationale in any source → ask the user directly; some decisions are only explained by the people involved, not the documents
- **Silent completion**: a WorkItem that was `InProgress` in the prior handoff has no new events this period → ask if it is still in progress or was completed without announcement
- **Scope ambiguity**: a WorkItem's delivery scope (MVP vs post-MVP) is not explicit in sources → ask which target release it belongs to; do not infer from stream placement alone

<HARD-GATE> Do not begin writing until Stage 3 is complete or the user has explicitly passed on all remaining questions. </HARD-GATE>

### Signal triage

Before writing, classify each WorkItem from the assembled event sequences:

**Reportable** — include in the update body:
- Status changed this period (any `StatusChanged` event)
- New blocker raised or resolved (`Blocked` / `Unblocked` event)
- Key decision made or superseded this period (`DecisionMade` / `DecisionSuperseded` event)
- Decision to skip, defer, or cancel a previously planned item (`SkippedEvent` / `DeferredEvent`) — a non-action decision is as Reportable as completing work

**Background** — omit from the update body:
- Ongoing work with no new events this period
- Team operations items (e.g., internal process docs, team norms, administrative tasks) — always Background regardless of status change; include in handoff WorkItems but never in the update document body

If a section has Background items, collect them in a single "Continuing: [item1], [item2]" line at the section end — no status labels, no sub-bullets.

**Cross-cutting deduplication**: If a single event involves PoCs from multiple sections, assign it to exactly one section (the section where the primary decision-maker or originating action sits). Omit from all other sections.

## 4. Write the update document

Use **resolving-docs** for file naming and version creation.

- Each section MUST open with a `>` blockquote: one sentence covering current state + key upcoming milestone or open decision. If a concrete timeline exists, include it in the quote: `> Timeline: Decision brief 4/29 → Mock server 5/6 → Release QA 6/1`.
- When a mainstream section's Done items span 2+ distinct phases or tracks (regardless of count), use a single `**Done**` header with nested bold sub-phase labels (e.g., `**Done**` → `- **Eval 4**` → `    - item`). If all Done items belong to the same phase or track, list them flat under `**Done**`.
- Top-level bullet = who, what, when — one line. Sub-bullets = rationale, key numbers, dependency notes. Do not mix summary and detail in the same line.
- Lead with status (Done / In Progress / To Do / Blocked), not background
- `-` bullets for all work items; `>` blockquotes for resolved Decisions
- Nested bullets over tables; people as sub-items, not section headers; add `(Owner)` inline in item titles when ownership varies across items within the same section; numbers as supporting detail
- Summarize related items as a sentence rather than enumerating each as a separate bullet when they share a category

- MUST NOT reference workspace file paths in the body
- MUST follow the date and notation format rules in writing-convention.md (date format, arrow usage).
- SHOULD NOT include Jira ticket numbers in the main bullet text; use them in sub-bullets or omit unless the ticket is the primary identifier for that item
- Link source documents inline when an item references specific numbers (latency, cost, counts) or an external doc: `[short label](URL)` immediately after the item title or in the sub-bullet containing the number. Do not link every item — only when the reader would need the source to verify or act.
- For each **Reportable** WorkItem (from signal triage): state the current status (derived from its event sequence) and the key events that explain how it got there — decisions made, blockers raised/resolved, corrections issued
- State the conclusion. For Done items where the reason changes what a reader should do next, add a one-line rationale in a sub-bullet. Do not add rationale for routine completions.
- When a section has `SkippedEvent` or `DeferredEvent` items, render them in a `**Skipped**` section between Done and In Progress: `- item title — one-line reason`.
- For To Do items with a confirmed hard dependency, add a sub-bullet: `- blocked by: [item]`. For unconfirmed or soft dependencies, use: `- risk: [item] — [potential impact]`. Only surface dependencies where the reader needs to act or where a delay would surprise them. Do not write `blocked by` for relationships inferred from source material unless confirmed by the user.
- Open decisions appear in two subsections at the bottom of each section: **Discussion (internal)** for items the team is actively deliberating with no external blocker; **Open decisions (external input needed)** for items waiting on another team, PM, or counterpart. Use `[ ]` here (not `-`) to visually distinguish action-required items from narrative bullets above.

**Version management:** Follow naming `yyyy-MM-dd-<slug>-update-v<N>.md`. For v2+, include **Changes from v{N-1}** immediately after the source memo. MUST NOT modify an existing version file.

**When sharing** (posting to Slack, copying to clipboard, or forwarding to others): MUST exclude the source memo and the Changes from v{N-1} section. Share body content only — starting from the document title heading.

## 5. Save handoff

Write or update the handoff file for this project using **resolving-docs**. Follow the handoff naming convention: `yyyy-MM-dd-<slug>-handoff-v<N>.md` with today's date. MUST NOT modify an existing handoff version — create a new version.

The handoff file MUST contain:

**Stage 1 Anchors**
| Source | Anchor |
|---|---|
| Slack | channel name |
| Jira | project key |
| GitHub | org/repo(s) |
| Notion | space or page title |

**Domain Model Instance** — all WorkItems and Decisions as of end of Stage 3:
- Each WorkItem: title, owner, current status, key recent events
- Each Decision: statement, by, current status

**Milestones** — concrete date commitments collected during Stage 2; each entry: description, date, status (Planned / Confirmed / Slipped / Met), owner.

**Unresolved Items** — items the user passed on in Stage 3; carry into Stage 3 of the next session.

**Coverage** — time range covered in this session: `[start] → [end]`

## 6. Report to the user

State what is in the document and what was not verifiable from sources. Flag any section based on user-provided context rather than a verified source.
