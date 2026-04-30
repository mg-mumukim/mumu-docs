---
name: writing-status
description: Write a status update or weekly progress report to `work/<topic>`. Use when the user asks to summarize current progress, prepare a sync document, or produce a Notion-pasteable update. Triggers on "weekly update", "진행 상황 정리", "현황 보고", "sync 준비", "weekly 작성", "주간 업데이트" and similar.
---

# Usage
/writing-status <topic> <optional: reference doc URL or description>

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Lesson
The two root causes of poor status documents:
1. **Writing before understanding the format** — producing a document and then iterating on its structure wastes versions and misaligns with the reader's expectations.
2. **Writing before all sources are collected** — partial research produces inaccurate ownership, missing items, and stale status. Slack is the most current source and MUST be read before writing.

These are addressed by the gates in this workflow. Do not skip them.

# Workflow

## 1. Confirm the reference format

Before any research, find and read the reference document that defines the expected format.

- If the user provided a URL, fetch it. If not, ask: "어떤 문서 형식에 맞춰야 하나요? 기존 회의록이나 레퍼런스 문서가 있으면 링크해 주세요."
- From the reference, extract and present to the user:
  - **Section structure**: how many sections, what they are called
  - **Format**: table vs nested bullet, column names if table
  - **Detail level**: how specific (LP ticket numbers? exact counts? or high-level only?)
  - **Language**: Korean / English / mixed
- MUST get explicit user confirmation before proceeding.

## 2. Propose the stream structure

Based on the topic and confirmed format, propose the section names and their owners.

- Present as a numbered list: section name, PoC, one-line description of what it covers
- Ask: "이 구조로 진행할까요, 아니면 수정할 부분이 있나요?"
- MUST get explicit user confirmation before proceeding to research.

### Gate
MUST NOT begin research until both format (Step 1) and stream structure (Step 2) are approved.

## 3. Collect all sources — breadth first

Search ALL sources for ALL sections before writing any section. Do not interleave research and writing.

**Source priority (most current → most structural):**
1. **Slack** — actual activity, decisions, blockers, role splits. Read channel messages and key threads. What people said they did is more authoritative than what Jira says they own.
2. **Jira** — task ownership, inter-task relationships, status transitions.
3. **Notion / Google Docs / meeting notes** — decisions made, context, prior-week continuity.

**Per section, verify:**
- Who actually did what (Slack), not just who is assigned (Jira)
- Whether a task is truly Done vs In Progress vs blocked — check latest Slack message, not just Jira status
- Any cross-section dependencies (e.g., one person's output is another section's input)

**Before writing, confirm no more information is coming:**
- If the user is actively sharing links or content, ask: "반영할 내용이 더 있으신가요?" before drafting.

## 4. Write the status document

Use **resolving-docs** for file naming and version creation.

### Document structure

The document MUST follow the approved stream structure (Step 2). Within each section:

- **Lead with status** (Done / In Progress / To Do / 논의 중), not with background
- **Nested bullets over tables** unless the reference format explicitly uses tables
- **People as sub-items**, not section headers — ownership belongs inside the item, not as a heading
- **Current state before next steps** — what is true now, then what happens next
- **Numbers as supporting detail**, not as the main point — put counts/sizes in nested bullets if needed

Do NOT:
- Reference workspace file paths in the body
- Include LP ticket numbers in the main bullet text (reference only if needed)
- List what was investigated — only state conclusions

### Version management

- Follow naming: `yyyy-MM-dd-<slug>-status-v<N>.md` (Type = `-status`)
- For v2+: include **Changes from v{N-1}** section immediately after the source memo
- MUST NOT modify an existing version file — create a new version for each update batch
- When new information arrives mid-session: collect all of it, then create one version

## 5. Report to the user
- State what is in the document and what was not verifiable from sources
- If any section is based on user-provided context rather than a source, flag it explicitly
