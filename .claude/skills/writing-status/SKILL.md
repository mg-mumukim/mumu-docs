---
name: writing-status
description: Researches work activity for a person or team on a specific project over a time period, then writes a coherent status summary to `work/<topic>`. Triggers when the user provides a subject (person or team name), a project scope, and a time range and asks for a status or activity summary. Activates on "[팀] [프로젝트] 진행 상황", "지난 [기간] [팀] 뭐 했는지 정리", "[person] status on [project]", "현황 보고", "진행 상황 정리", "weekly update", "주간 업데이트" and similar. Does not trigger for drafting proposals, writing research reports, or reviewing documents.
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

Reference: [domain-model.md](domain-model.md) — shared vocabulary for Person, WorkItem, Decision, and Event used in Stages 2–4.

# Inputs

Confirm before starting research. If any of the following are missing, ask once:

- **Subject**: person name or team name
- **Project / scope**: which project or workstream
- **Time range**: start and end date (absolute)
- **Output intent**: shared with others (team meeting, manager report) or personal reference

## 1. Establish the format

If the user provided a reference document or URL, fetch it and extract: section structure, format (table vs bullets), detail level, and language. Use that as the output format.

If no reference is provided, use the default format: nested bullets, status-first per item (Done / In Progress / To Do / Blocked), current state before next steps.

No confirmation needed for format unless the user explicitly asks to verify it.

## 2. Propose section structure (team subjects only)

If the subject is a team or multiple people, propose section names and their owners as a numbered list: section name, PoC, one-line scope. MUST get explicit user confirmation before beginning research.

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

Run all anchor lookups concurrently.

| Source | Anchor to find | Fallback if unknown |
|---|---|---|
| Slack | Project-dedicated channel name | Search by project name via Glean |
| Jira | Project key | Search by project name in Jira |
| GitHub | Relevant repo(s) | Search by project name within org |
| Notion | Relevant space or page title | Keyword search by project name |
| Google Calendar | Subject's email address | Check `list_calendars()` output |

<HARD-GATE> Do not begin Stage 2 until all anchors are confirmed or marked unavailable. </HARD-GATE>

### Stage 2: Collect — breadth first, then follow links

Before fetching, use **resolving-docs** to check `note/<project-scope>/` for existing source notes (`*-slack.md`, `*-jira.md`, `*-github.md`, `*-notion.md`, `*-gcal.md`). Read files already present; fetch only missing sources. Launch all missing-source queries concurrently. Collect each source in chronological order.

**Source priority (most current → most structural):**
1. **Slack** — project channel + subject's messages within time range. Read full threads; capture both the original statement and any later corrections or reversals.
2. **Jira** — tickets where subject is assignee, updated within time range. Include changelog: status transitions, reassignments, reopens.
3. **GitHub** — PRs authored by subject, merged or closed within time range. Include review iterations and revert commits.
4. **Notion / Google Docs / meeting notes** — meeting notes within time range. Include later entries that amend or override earlier decisions.
5. **Google Calendar** — call `list_events(calendarId=subject_email, startTime, endTime, orderBy=startTime)` for each subject. Collect:
   - Meeting events: title, attendees with response status, description. Extract any linked doc/Jira/Notion URLs from description and add to the internal link follow queue.
   - OOO and leave events: `eventType=outOfOffice` or events created by `hr.sys4@hpcnt.com` — record as periods of subject unavailability (`Blocked`).
   - For team subjects, query all members concurrently.

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

<HARD-GATE> Do not begin writing until Stage 3 is complete or the user has explicitly passed on all remaining questions. </HARD-GATE>

## 4. Write the update document

Use **resolving-docs** for file naming and version creation.

**If output intent is shared (team meeting, manager report):**
- Lead with status (Done / In Progress / To Do / Blocked), not background
- Nested bullets over tables unless the reference format uses tables
- People as sub-items, not section headers
- Numbers as supporting detail in nested bullets

**If output intent is personal reference:**
- Narrative prose over status labels
- Chronological or thematic ordering, not status grouping

In both cases:
- MUST NOT reference workspace file paths in the body
- MUST NOT include Jira ticket numbers in the main bullet text
- For each WorkItem: state the current status (derived from its event sequence) and the key events that explain how it got there — decisions made, blockers raised/resolved, corrections issued
- State conclusions only — do not list what was investigated

**Version management:** Follow naming `yyyy-MM-dd-<slug>-update-v<N>.md`. For v2+, include **Changes from v{N-1}** immediately after the source memo. MUST NOT modify an existing version file.

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

**Unresolved Items** — items the user passed on in Stage 3; carry into Stage 3 of the next session.

**Coverage** — time range covered in this session: `[start] → [end]`

## 6. Report to the user

State what is in the document and what was not verifiable from sources. Flag any section based on user-provided context rather than a verified source.
