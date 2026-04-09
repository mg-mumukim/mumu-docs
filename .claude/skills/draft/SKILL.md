---
name: draft
description: Draft or revise a document following the `00_note/` -> `01_work/` -> `02_record/` workflow.
---

# Usage
/draft <topic or filename> <request or feedback>

The argument can be:
- A topic to start a new draft (e.g. `/draft privacy-by-design-section-3`)
- An existing filename in `01_work/` to revise (e.g. `/draft privacy-by-design-section-2-8-v1.md`)

# Workflow

## 1. Gather source material
- Read all files in `00_note/` that are relevant to the requested topic.
- If a specific note file is referenced, read that file.
- If external sources (Slack, Confluence, Glean) are needed, search and collect them.

## 2. Check existing versions in `01_work/`
- Glob `01_work/*<topic>*` to find existing versions.
- If versions exist, read the latest version to find:
  - `[!NOTE]` callouts with reviewer feedback.
  - Inline comments or requests from the human reviewer.
  - Determine the next version number (e.g. v2 -> v3).
  - Do not increase date.
- If no versions exist, start at v1.

## 3. Write the new version
- Create a new file: `01_work/yyyy-MM-dd-<topic>-v<N>.md`
- Use today's date for the new version file.
- Incorporate all reviewer `[!NOTE]` from the previous version.
- Use source material from `00_note/` as the basis.
- Memo which files or information sources are refered at the top of the document.
- Do NOT modify any existing version files.

### 4. Report to the user
- Summarize what changed from the previous version (if applicable).
- List which `[!NOTE]` items were addressed.
- Note any items that could not be addressed and why.

### 5. Graduation (only when human says "approve" or "finalize")
- Move the approved version to `02_record/`, stripping the version suffix.
- If the draft is about refinement of exsisting file, overwrite it.
- Do NOT graduate without explicit human approval.

## Rules
- Never modify files in `00_note/`.
- Never modify existing version files in `01_work/`.
- Require approve to create or update files in `02_record/`
- Always create a new version file for each revision.
