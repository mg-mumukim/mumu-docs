---
name: write-report
description: Research a topic and write a report to `work/<topic>`. Use only when the user explicitly asks for a written report, document, or deliverable — not for quick questions or conversational research.
---

# Usage
/write-report <topic> <optional question>

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Identify the question
- Extract the user's question from the arguments. The question drives the entire research.
- If the question is implicit (e.g. "investigate X"), restate it explicitly before proceeding (e.g. "How does X work, and is it reliable?").

## 2. Prepare a report plan

### Check for an existing plan
- Glob `note/task/*<topic>*report-plan*` to find a matching plan file.
- If a plan exists, read it and present a summary to the user:
  - Show the existing question, sources, scope, and output format.
  - Ask the user whether to **reuse as-is**, **revise**, or **start fresh**.
  - If the user chooses to revise, create a new plan file with today's date (do not modify the old file).
  - If the user chooses to reuse, proceed directly to step 3.

### Write a new plan (only when no existing plan is found, or user chose "start fresh")
- Create `note/task/yyyy-MM-dd-<topic-slug>-report-plan.md`.
- The plan MUST include:
  - **Question**: The restated question from step 1.
  - **Sources to check**: Which sources to search (Slack, Jira, Glean, Notion, Confluence, web, codebase, etc.) and why.
  - **Scope**: What is in scope and what is explicitly out of scope.
  - **Output**: Expected postfix for the report file (e.g. `-report`, `-benchmark`, `-comparison`).
- Present the plan to the user and request review.

### Gate
- MUST NOT proceed to research until the user approves (new or existing plan).

## 3. Search
- After user approval, search the sources listed in the plan.
- Read existing files in `note/source/` and `work/` to avoid duplicating prior research.

## 4. Verify
- Cross-check numbers, formulas, and claims against primary sources (code, docs, Slack threads).
- Flag discrepancies, unverified assumptions, or gaps in evidence.

## 5. Write the report

Use **resolve-docs** for project directory, file naming, and version creation. The file's `<document-slug>` includes the output postfix from the report plan (e.g. `-report`, `-benchmark`).

### Document structure

The document MUST follow a **Conclusion → Findings → Evidence** hierarchy:

1. **Conclusion** — A direct answer to the user's question. The reader MUST understand the answer from this section alone, without reading further. State the question, then the answer, then the critical numbers.
2. **Key Findings** — The reasoning that supports the conclusion. Each finding MUST be a claim with inline evidence. If a fact has a known issue, state the issue next to the fact — not in a separate section.
3. **Evidence / Details** — Supporting data that the findings reference: production cost breakdowns, benchmark tables, scaling scenarios. Only include what a finding explicitly cites.
4. **Reference (appendix)** — Raw data extracted from code or docs (constants, pricing tables, formulas, arithmetic verification). This section exists for reproducibility, not for reading. Do not let it dominate the document.

- MUST NOT use a structure that follows the order of investigation (searched A, then found B, then checked C). The document is organized by what the reader needs, not by how the research was done.

## 6. Report to the user
- Summarize what was found and what remains unverified.
- List any gaps or questions that could not be answered.

# Rules
- MUST NOT proceed to research without user approval of the report plan.
- MUST NOT duplicate existing research in `note/source/` or `work/`.
