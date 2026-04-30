---
name: writing-status
description: Writes a status update or weekly progress report to `work/<topic>`. Triggers when the user asks to summarize current progress, prepare a sync document, or produce a Notion-pasteable team update. Activates on "weekly update", "진행 상황 정리", "현황 보고", "sync 준비", "weekly 작성", "주간 업데이트" and similar. Does not trigger for drafting proposals, writing research reports, or reviewing documents.
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Confirm the reference format

Before any research, find and read the reference document that defines the expected format.

- If the user provided a URL, fetch it. If not, ask for a reference document or existing meeting notes link.
- Extract and present to the user: section structure, format (table vs nested bullets), detail level (ticket numbers? exact counts? high-level only?), and language.
- MUST get explicit user confirmation before proceeding.

## 2. Propose the stream structure

Propose section names and their owners as a numbered list: section name, PoC, one-line description of what it covers.

- MUST get explicit user confirmation before beginning research.

MUST NOT begin research until both format (Step 1) and stream structure (Step 2) are confirmed.

## 3. Collect all sources — breadth first

Search ALL sources for ALL sections before writing any section. Do not interleave research and writing.

**Source priority (most current → most structural):**
1. **Slack** — actual activity, decisions, blockers, role splits. What people said they did overrides what Jira says they own.
2. **Jira** — task ownership, inter-task relationships, status transitions.
3. **Notion / Google Docs / meeting notes** — decisions made, context, prior-week continuity.

Per section, verify: who actually did what (Slack, not just Jira assignment), whether a task is truly Done vs blocked (latest Slack message, not Jira status), and cross-section dependencies.

If the user is actively sharing additional links or context, ask "반영할 내용이 더 있으신가요?" before drafting.

## 4. Write the status document

Use **resolving-docs** for file naming and version creation.

The document MUST follow the approved stream structure. Within each section:

- **Lead with status** (Done / In Progress / To Do / 논의 중), not with background
- **Nested bullets over tables** unless the reference format explicitly uses tables
- **People as sub-items**, not section headers — ownership belongs inside the item
- **Current state before next steps** — what is true now, then what happens next
- **Numbers as supporting detail**, not as the main point — put counts in nested bullets if needed

Do NOT:
- Reference workspace file paths in the body
- Include LP ticket numbers in the main bullet text
- List what was investigated — state conclusions only

**Version management:** Follow naming `yyyy-MM-dd-<slug>-status-v<N>.md`. For v2+, include **Changes from v{N-1}** immediately after the source memo. MUST NOT modify an existing version file — create a new version for each update batch.

## 5. Report to the user

State what is in the document and what was not verifiable from sources. Flag any section based on user-provided context rather than a verified source.
