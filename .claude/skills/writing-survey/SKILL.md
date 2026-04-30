---
name: writing-survey
description: Creates a literature or landscape survey that maps prior work — internal docs, academic papers, industry blog posts — to identify options and inform a proposal or design decision. Triggers when the user wants to understand what exists before committing to an approach. Activates on "literature survey", "landscape survey", "write a survey", "survey 써줘", "survey 작성", "관련 연구 정리", "prior art 정리", "어떤 선택지가 있나", "무엇이 있는지 정리" and similar. Does NOT trigger when the user has already run an experiment or analysis and wants to document results — use writing-report for that.
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Frame the scope
- Identify the problem space being surveyed: what class of approaches, models, or systems is in scope?
- Identify what decision or proposal this survey will inform (e.g., "choosing an architecture for V5", "deciding whether to adopt contextual bandits").
- Determine the source types to cover: industry blog posts, academic papers, internal docs, or a subset.

## 2. Prepare a handoff

### Check for an existing handoff
- Use **resolving-docs** to check `work/<topic>/` for an existing `*-handoff-v*.md`.
- If found, present Goal, Plan (Question, Sources, Scope), and Open Work. Ask: **reuse as-is**, **revise**, or **start fresh**.

### Write a new handoff (when none exists or user chose "start fresh")
- Use **resolving-docs** to resolve the project directory.
- Create `yyyy-MM-dd-<slug>-handoff-v1.md` with:
  - **Goal**: What proposal or design decision this survey informs; intended audience.
  - **Plan**:
    - **Question**: What problem space and options are being mapped.
    - **Sources**: Which sources to cover (internal docs, arXiv, industry blogs, Confluence, etc.) and why.
    - **Scope**: What is explicitly in scope and out of scope.
    - Status: draft
- Present the Plan to the user and request review.

### Gate
- MUST NOT proceed to research until the user approves (Status: approved).

## 3. Search
- List files currently in `note/<scope>/`. For each source listed in the plan, check whether a corresponding file already exists.
  - If all sources are covered: read them directly and skip to Step 4.
  - If partially covered: identify only the missing sources.
  - If none are covered: all sources listed in the plan need fetching.
- **Parallel fan-out (Agent tool):** Launch one subagent per missing source in a single message. Each subagent must: (1) fetch its assigned source (industry blog, arXiv paper, internal doc, Confluence page, etc.), (2) save the result to `note/<scope>/yyyy-MM-dd-<slug>-<source-type>.md` (e.g., `-paper.md`, `-blog.md`, `-notion.md`), and (3) return a one-line summary of what was found, or note if the source was inaccessible. Wait for all subagents to complete before proceeding.
- Read existing files in `work/` to avoid duplicating prior written analysis.

## 4. Write the survey

Use **resolving-docs** for project directory, file naming, and version creation (`<slug>-survey-vN.md`).

Check `work/<topic>/` for existing `*-survey-v*.md`:
- If found → **revision mode**: read the latest version and collect `[!NOTE]` feedback; determine next version number; apply feedback; include a **Changes from v{N-1}** section directly after the sources block.
- If not found → start at v1.

### Document structure

The document MUST follow this structure:

```
<!-- Sources: ... -->

# [Topic] Survey

## Scope
What problem space this covers and what decision or proposal it informs.

## Prior Work

### Industry
One subsection per system or company. Each entry:
- **Method/approach**: what they do
- **Key ideas**: the core insight or technique
- **Relevance**: how this maps to our use case

### Academic
One subsection per paper. Each entry:
- **Method**: core algorithm or technique
- **Key ideas**: main contribution
- **Relevance**: how this maps to our use case

### Internal
Prior internal work, experiments, or documents relevant to this problem space.
One subsection per project or document. Include what was tried, what was found, and what was left open.

## Design Space
Options surfaced by the survey. For each option:
- What it is (one sentence)
- Key tradeoff
- Signal from prior work (which sources support or caution against it)
```

Omit sections that have no entries (e.g., omit Academic if no papers were found).

MUST NOT use a Conclusion → Findings → Evidence structure. The survey is organized by source, not by a single answer.

## 5. Report to the user
- Write a new handoff version (today's date, incremented version):
  - If complete: set `Status: completed`. Future sessions that load this handoff will know the survey is done.
  - If incomplete: set `Status: in-progress`; add Open Work and New Session Prompt sections.
- List which sources were covered and which were not found or inaccessible.
- Name which option in the Design Space has the strongest prior-work signal.
