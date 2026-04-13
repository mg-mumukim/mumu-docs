---
name: probe
description: Research a topic — plan in `note/task/`, report in `work/<topic>`. Use when asked to investigate, compare, estimate, or fact-check something.
---

# Usage
/probe <topic> <optional question>

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Identify the question
- Extract the user's question from the arguments. The question drives the entire research.
- If the question is implicit (e.g. "investigate X"), restate it explicitly before proceeding (e.g. "How does X work, and is it reliable?").

## 2. Prepare a probe plan

### Check for an existing plan
- Glob `note/task/*<topic>*probe-plan*` to find a matching plan file.
- If a plan exists, read it and present a summary to the user:
  - Show the existing question, sources, scope, and output format.
  - Ask the user whether to **reuse as-is**, **revise**, or **start fresh**.
  - If the user chooses to revise, create a new plan file with today's date (do not modify the old file).
  - If the user chooses to reuse, proceed directly to step 3.

### Write a new plan (only when no existing plan is found, or user chose "start fresh")
- Create `note/task/yyyy-MM-dd-<topic-slug>-probe-plan.md`.
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

### Resolve project directory
- Glob `work/**/*<topic>*` to find an existing project directory.
- If found, use that directory.
- If not found, create a new directory under `work/` with `<scope>-<subject>` naming in lowercase kebab-case.
- If the scope is ambiguous, ask the user before creating.

### File naming
- `work/<project>/yyyy-MM-dd-<topic-slug>-<output>-v1.md`, where `<output>` is the postfix defined in the probe plan (e.g. `-report`, `-benchmark`, `-comparison`).
- Subsequent revisions increment the version: `-v2.md`, `-v3.md`.
- MUST NOT modify existing version files.

### Document structure

The document MUST follow a **Conclusion → Findings → Evidence** hierarchy:

1. **Conclusion** — A direct answer to the user's question. The reader MUST understand the answer from this section alone, without reading further. State the question, then the answer, then the critical numbers.
2. **Key Findings** — The reasoning that supports the conclusion. Each finding MUST be a claim with inline evidence. If a fact has a known issue, state the issue next to the fact — not in a separate section.
3. **Evidence / Details** — Supporting data that the findings reference: production cost breakdowns, benchmark tables, scaling scenarios. Only include what a finding explicitly cites.
4. **Reference (appendix)** — Raw data extracted from code or docs (constants, pricing tables, formulas, arithmetic verification). This section exists for reproducibility, not for reading. Do not let it dominate the document.

- MUST NOT use a structure that follows the order of investigation (searched A, then found B, then checked C). The document is organized by what the reader needs, not by how the research was done.
- MUST memo which files, URLs, or information sources are referred at the top of the document.

### Writing convention

- **No embellishment**: MUST NOT use metaphors, analogies, or illustrative examples not present in source material.
- **No LaTeX**: MUST use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation.
- **Plain tables**: MUST NOT use inline formatting (bold, italic, strikethrough, etc.) in table cells.
- **Use K/M notation**: SHOULD write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **Source-faithful only**: MUST NOT invent specifics, implementation details, or quantities absent from source material.
- **No editorial additions**: MUST NOT add opinions or recommendations unless the user explicitly requests it. Identifying factual discrepancies and unverified assumptions is not editorial — it is verification.

## 6. Report to the user
- Summarize what was found and what remains unverified.
- List any gaps or questions that could not be answered.

# Rules
- **Write everything in English.** All output files MUST be in English regardless of the user's input language.
- SHOULD use plain, practitioner-friendly English as the default style.
- MUST NOT proceed to research without user approval of the probe plan.
- MUST NOT modify existing version files in `work/`.
- MUST NOT duplicate existing research in `note/source/` or `work/`.
