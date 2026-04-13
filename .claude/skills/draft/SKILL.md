---
name: draft
description: Draft or revise a document from `00_note/` sources into `01_work/` versioned files.
---

# Usage
/draft <topic or filename> <request or feedback>

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

The argument can be:
- A topic to start a new draft (e.g. `/draft privacy-by-design-section-3`)
- An existing filename in `01_work/` to revise (e.g. `/draft privacy-by-design-section-2-8-v1.md`)

# Workflow

## 1. Gather source material
- Read `INDEX.md` to find files related to the requested topic.
- Read all files in `00_note/` that are relevant to the requested topic.
- If a specific note file is referenced, read that file.
- If external sources (Slack, Jira, Glean, Notion) are needed, search and collect them.

## 2. Check existing versions in `01_work/`
- Glob `01_work/*<topic>*` to find existing versions.
- If versions exist, read the latest version to find:
  - `[!NOTE]` callouts with reviewer feedback.
  - Inline comments or requests from the human reviewer.
  - Determine the next version number (e.g. v2 -> v3).
  - Do not increase date.
- If no versions exist, start at v1.

## 3. Write the new version
- Create a new file: `01_work/yyyy-MM-dd-<topic>-v<N>.md`.
- Use today's date (!`date '+%Y-%m-%d'`) for the new version file.
- Incorporate all reviewer `[!NOTE]` from the previous version.
- SHOULD use source material from `00_note/` as the basis.
- MUST memo which files or information sources are referred at the top of the document.
- MUST NOT modify any existing version files.

### Document structure
- MUST follow the structure and heading hierarchy given in the `00_note/` source material. MUST NOT invent new sections or reorder unless the user requests it.

### Writing convention
- **No embellishment**: MUST NOT use metaphors, analogies, or illustrative examples not present in source material.
- **No LaTeX**: MUST use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation.
- **Plain tables**: MUST NOT use inline formatting (bold, italic, strikethrough, etc.) in table cells.
- **Use K/M notation**: SHOULD write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **Source-faithful only**: MUST NOT invent specifics, implementation details, or quantities absent from source material. MUST use exact definitions and terminology from the source.
- **No editorial additions**: MUST NOT add opinions, recommendations, or commentary unless the source note explicitly requests it.

## 4. Update INDEX.md
- If the new draft introduces a topic not yet in `INDEX.md`, add it.
- Add the new `01_work/` file under the appropriate topic.
- If the topic already has a `02_record/` entry for this document, do not add the `01_work/` file.

## 5. Report to the user
- Summarize what changed from the previous version.
- List which `[!NOTE]` items were addressed.
- Note any items that could not be addressed and why.

# Rules
- MUST NOT modify files in `00_note/`.
- MUST NOT modify existing version files in `01_work/`.
- MUST create a new version file for each revision in `01_work/`.
- SHOULD use plain, practitioner-friendly English as the default language.
