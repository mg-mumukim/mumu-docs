---
name: writing-report
description: Researches a topic and writes a report to `work/<topic>`. Triggers only when the user explicitly asks for a written report, document, or deliverable — not for quick questions or conversational research. Activates on "write a report", "research and write", "보고서 작성", "리포트 작성", "조사해서 정리", "분석 보고서", "report 써줘" and similar.
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Examples

Before Step 5 (Write the report), if uncertain about how to structure the sources block, changes log, Conclusion density, or the Conclusion → Key Findings → Evidence hierarchy, glob `examples/` in this skill's directory. Read the filenames and pick the most relevant one by topic or format. Then read its header — it states what the example demonstrates and how much to read.

# Workflow

## 1. Frame the problem
- Restate the user's question as a specific unknown: what fact, number, or judgment is missing?
- Identify what depends on the answer: what decision, action, or document is blocked until this is known?
- From that, determine what a sufficient answer looks like — its form (a number, a comparison, a recommendation), its precision (order-of-magnitude vs exact), and its scope.

## 2. Prepare a handoff

### Check for an existing handoff
- Use **resolving-docs** to check `work/<topic>/` for an existing `*-handoff-v*.md`.
- If a handoff exists, read it and present a summary to the user:
  - Show Goal, Plan (Question, Sources, Scope, Type), and Open Work.
  - Ask the user: **reuse as-is**, **revise**, or **start fresh**.
  - Reuse: proceed to step 3.
  - Revise: write a new handoff version (today's date, incremented version); present for review.
  - Start fresh: proceed to "Write a new handoff" below.

### Write a new handoff (only when no existing handoff is found, or user chose "start fresh")
- Use **resolving-docs** to resolve the project directory.
- Create `yyyy-MM-dd-<slug>-handoff-v1.md` with:
  - **Goal**: What the deliverable is for; intended audience; any constraints.
  - **Plan**:
    - **Question**: The restated question from step 1.
    - **Sources**: Which sources to search (Slack, Jira, Glean, Notion, Confluence, web, codebase, etc.) and why.
    - **Scope**: What is in scope and what is explicitly out of scope.
    - **Type**: File type postfix for the output (e.g. `-report`, `-benchmark`, `-comparison`).
    - Status: draft
- Present the Plan section to the user and request review.

### Gate
- MUST NOT proceed to research until the user approves (Status: approved).

## 3. Search
- List files currently in `note/<scope>/`. For each source listed in the plan, check whether a corresponding file already exists.
  - If all sources are covered: read them directly and skip to Step 4.
  - If partially covered: fetch only the missing sources, then proceed.
  - If none are covered: fetch all sources listed in the plan.
- For each fetched external document (Notion page, GDoc, Slack thread, etc.), save a local copy to `note/<scope>/` using the `yyyy-MM-dd-<slug>-<source-type>.md` naming convention (e.g., `-notion.md`, `-gdocs.md`, `-thread.md`).
- Read existing files in `work/` to avoid duplicating prior written analysis.

## 4. Verify
- Cross-check numbers, formulas, and claims against primary sources (code, docs, Slack threads).
- Flag discrepancies, unverified assumptions, or gaps in evidence.

## 5. Write the report

Use **resolving-docs** for project directory, file naming, and version creation. The file's `<document-slug>` includes the Type postfix from the handoff Plan (e.g. `-report`, `-benchmark`).

Check `work/<topic>/` for existing report versions matching the Type postfix:
- If found → **revision mode**: read the latest version and collect `[!NOTE]` feedback; determine next version number; apply feedback in the new version; include a **Changes from v{N-1}** section directly after the sources block.
- If not found → start at v1.

### Document structure

The document MUST follow a **Conclusion → Findings → Evidence** hierarchy:

1. **Conclusion** — A direct answer to the user's question. The reader MUST understand the answer from this section alone, without reading further. State the question, then the answer, then the critical numbers.
2. **Key Findings** — The reasoning that supports the conclusion. Each finding is a claim with a brief inline citation (a number, a source name, a one-line quote). Do not embed full tables or raw data here — reference the Evidence section instead.
3. **Evidence / Details** — The full supporting data that findings reference: production cost breakdowns, benchmark tables, scaling scenarios. Only include what a finding explicitly cites.
4. **Reference (appendix)** — Raw data extracted from code or docs (constants, pricing tables, formulas, arithmetic verification). This section exists for reproducibility, not for reading. Do not let it dominate the document.

- MUST NOT use a structure that follows the order of investigation (searched A, then found B, then checked C). The document is organized by what the reader needs, not by how the research was done.

## 6. Report to the user
- Summarize what was found and what remains unverified.
- List any gaps or questions that could not be answered.
- If work is incomplete or will continue in a new session, write a new handoff version (today's date, incremented version) with Decisions, Traps, Open Work, and New Session Prompt sections added.
