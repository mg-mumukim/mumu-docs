---
name: write-draft
description: Draft or revise a document from `note/` sources into `work/<topic>`. Use when asked to write, draft, or summarize a deliverable document.
---

# Usage
/write-draft <topic or filename> <optional feedback>

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

The argument can be:
- A topic to start a new draft (e.g. `/write-draft privacy-by-design`)
- An existing filename in `work/<topic>` to revise (e.g. `/write-draft privacy-by-design-v1.md add executive summary`)

# Workflow

## 1. Analyze the request
- Identify the **topic** from the argument.
- Identify the **audience** (executives, engineers, external, etc.).
- Separate user instructions into two categories:
  - **Goal**: What the user ultimately wants to achieve. This is an outcome, not a procedure. (e.g. "make the cost section convincing", "emphasize privacy compliance")
  - **Directive**: Specific procedures, constraints, or formatting the user explicitly dictated. These MUST be followed as stated. (e.g. "keep it under 2 pages", "use the same structure as v1", "include a table comparing options")
- If the topic cannot be determined, ask the user before proceeding.

In all subsequent steps, MUST apply **directives** literally and use **goal** to guide judgment calls (structure, depth, emphasis, what to include or omit).

## 2. Gather material
- Glob `note/task/*<topic>*` — request files, probe plans, WIP notes.
- Glob `note/source/*<topic>*` — source material.
- Glob `work/**/*<topic>*` — existing work output (probe reports, prior drafts).
- Read `note/context/` files referenced by CLAUDE.md.
- If a request file (`*-request*`) is found, read it first — it is the primary input.
- Collect any external sources (Slack, Jira, Glean, Notion) specified by the user or needed by the request file.
- If existing versions are found → **revision mode**: read the latest version, collect `[!NOTE]` feedback, determine next version number.
- If no existing versions → start at v1.
- Read all discovered files before writing.

## 3. Write the new version

Use **resolve-docs** for project directory, file naming, and version creation.

### Structure
- If source material provides a heading hierarchy, MUST follow it. MUST NOT invent new sections or reorder unless a **directive** overrides it.
- If source material lacks structure, determine an appropriate structure based on the **goal** and **audience**.

### Depth and focus
- Adjust detail level and emphasis to match the **audience** (e.g. executives → impact/cost focus, engineers → technical detail).
- Use the **goal** to decide what to emphasize, include, or omit.

### Content
- SHOULD use source material from `note/` as the basis.
- For revisions: incorporate all `[!NOTE]` feedback from the previous version and any user feedback from step 1.

## 4. Report to the user
- Summarize what changed from the previous version (for revisions).
- List which `[!NOTE]` items were addressed (for revisions).
- Note any items that could not be addressed and why.
