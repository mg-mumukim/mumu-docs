---
name: publish
description: Publish an approved `01_work/` draft to `02_record/`, stripping the version suffix and committing.
---

# Usage
/publish <filename in 01_work/>

Example: `/publish 01_work/2026-04-10-photo-review-llm-cost-estimation-v2.md`

# Workflow

## 1. Verify the file
- Read the specified file in `01_work/`.
- Confirm it has no unresolved `[!NOTE]` callouts (warn the user if any remain).

## 2. Move to `02_record/`
- Copy the file to `02_record/`, stripping the version suffix (e.g. `-v2`).
- Remove the `> **Sources**:` block (the `>` quoted paragraph at the top of the document). This metadata is for drafting context only and should not appear in the published record.
- If the target already exists in `02_record/`, overwrite it (this is a refinement of an existing record).

## 3. Commit
- Stage the new/updated file in `02_record/`.
- If the `01_work/` versions were previously tracked by git, stage their deletion.
- Commit message: `Publish <topic> (from <source filename>)`

## Rules
- Do NOT publish without explicit human approval ("publish", "approve", "finalize").
- Never modify files in `00_note/`.
- Never modify existing version files in `01_work/` (they remain as-is on disk for history).
