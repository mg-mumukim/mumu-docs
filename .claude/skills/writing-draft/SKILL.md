---
name: writing-draft
description: Drafts or revises a document from `note/` sources into `work/<topic>`. Triggers when asked to write, draft, or summarize a deliverable document. Activates on "draft this", "write a document", "작성해줘", "초안 써줘", "문서 써줘", "정리해서 작성", "draft 해줘", "써줘" and similar. Does not trigger for research reports (use writing-report), document reviews (use writing-review), or status updates (use writing-status).
---

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

# Examples

Before Step 3 (Write the new version), if uncertain about expected structure or depth for a new v1 draft, glob `examples/` in this skill's directory. Read the filenames and pick the most relevant one by topic or format. Then read its header — it states what the example demonstrates and how much to read.

# Workflow

## 1. Analyze the request
- From the user's request, identify:
  - **Topic**: the subject to draft or the filename to revise.
  - **Audience** (executives, engineers, external, etc.).
- Separate the instruction into two categories:
  - **Goal**: What the user ultimately wants to achieve. This is an outcome, not a procedure. (e.g. "make the cost section convincing", "emphasize privacy compliance")
  - **Directive**: Specific procedures, constraints, or formatting the user explicitly dictated. These MUST be followed as stated. (e.g. "keep it under 2 pages", "use the same structure as v1", "include a table comparing options")
- If the topic cannot be determined, ask the user before proceeding.

In all subsequent steps, MUST apply **directives** literally and use **goal** to guide judgment calls (structure, depth, emphasis, what to include or omit).

## 2. Gather material
- Glob `note/**/*<topic>*` — source material across all scopes.
- Glob `work/**/*<topic>*` — existing work output (reports, prior drafts, handoffs).
- If a request file (`*-request*`) is found, read it first — it is the primary input.
- Collect external sources (Slack, Jira, Glean, Notion) from three origins: specified by the user, needed by the request file, or **linked within any gathered source material**. Follow references found during reading.
- If existing versions are found → **revision mode**:
  - Read the latest version.
  - Scan the **Changes from v{N-1}** sections of prior versions to understand the document's evolution.
  - Determine next version number.
- If no existing versions → start at v1.
- Read all discovered files before writing.

## 3. Write the new version

Use **resolving-docs** for project directory, file naming, and version creation.

### Structure
- If source material provides a heading hierarchy, MUST follow it. MUST NOT invent new sections or reorder unless a **directive** overrides it.
- If source material lacks structure, determine an appropriate structure based on the **goal** and **audience**.

### Depth and focus
- Adjust detail level and emphasis to match the **audience** (e.g. executives → impact/cost focus, engineers → technical detail).
- Use the **goal** to decide what to emphasize, include, or omit.
- When source material implies a causal mechanism, explain it explicitly for the audience. Do not copy conclusions without unpacking the reasoning behind them.
- Assume the audience lacks internal context. Clarify units, abbreviations, and numbers that could be ambiguous (e.g. "100K requests" not just "100K").

### Content
- SHOULD use source material from `note/` as the basis.
- For revisions, apply inputs in this priority order:
  1. **Directives** from step 1 — the user's current explicit instructions (highest priority).
  2. **Change logs** from prior versions — do not revert previous intentional changes unless a directive explicitly overrides them.

## 4. Verify

Before finalizing the document, cross-check its contents against source material:

- **Proper nouns**: library names, model names, service names, people's names, channel names — confirm each appears in source material verbatim. MUST NOT use a name that is only plausible but not found in sources.
- **Dates and timelines**: verify each date or relative reference against the source. Do not paraphrase (e.g. do not write "last week" if the source says a specific date).
- **Numbers and units**: confirm values and units are present in source. Flag any number that was derived or inferred rather than stated.
- **Attributions**: if a claim is attributed to a person or team, confirm the attribution is present in source.
- **Inferred specifics**: if a specific detail (e.g. a library, a threshold, a mechanism) is absent from source but was included based on plausibility, remove it.

If any item cannot be confirmed, remove it and note it to the user in step 5.

## 5. Report to the user
- Summarize what changed from the previous version (for revisions).
- Note any items that could not be addressed and why.
- List any items removed as unverified during step 4.
