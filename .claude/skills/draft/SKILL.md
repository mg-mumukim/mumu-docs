---
name: draft
description: Draft or revise a document from `00_note/` sources into `01_work/` versioned files.
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
- Use source material from `00_note/` as the basis if possible.
- Memo which files or information sources are refered at the top of the document.
- Do NOT modify any existing version files.

### Writing style rules
- **No embellishment**: No metaphors, analogies, or illustrative examples not present in source material.
- **No LaTeX**: Use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation by default.
- **No bold in tables**: Table cells should be plain text without `**bold**` markup.
- **Use K/M notation**: Write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **No fabricated specifics**: Do not invent specific times, durations, or quantities not stated in source material. If the source says "up to 24 hours", do not add "e.g., at 1pm".
- **No unsolicited opinions**: Do not add "When to choose this", recommendation sections, or editorial commentary unless the source note explicitly requests it. Stick to factual analysis.
- **Source-faithful terminology**: When describing domain concepts (e.g., what counts as a "profile change"), use the exact definition from the source, not a reinterpretation.
- **Don't over-specify implementation**: If the source says "batch processing", do not assume "single batch job per day" or other implementation details not stated.
-

## 4. Report to the user
- Summarize what changed from the previous version.
- List which `[!NOTE]` items were addressed.
- Note any items that could not be addressed and why.

## Rules
- Never modify files in `00_note/`.
- Never modify existing version files in `01_work/`.
- Always create a new version file for each revision in `01_work/`.
