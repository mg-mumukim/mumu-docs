---
name: publish
description: Publish an approved `01_work/` draft to `02_record/`, stripping the version suffix and committing.
---

# Usage
/publish <filename in 01_work/> [optional request]

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

Examples:
- `/publish 01_work/2026-04-10-photo-review-llm-cost-estimation-v2.md`
- `/publish 01_work/2026-04-10-photo-review-llm-cost-estimation-v2.md finalize the conclusion`

# Workflow

## 1. Verify the file
- Read the specified file in `01_work/`.
- MUST confirm it has no unresolved `[!NOTE]` callouts (warn the user if any remain).

## 2. Apply optional request
- If the user appended a request (e.g. "finalize", "polish", "fix typos"), apply it to the content before writing to `02_record/`. MUST follow the draft skill's writing conventions.

## 3. Move to `02_record/`
- Copy the file to `02_record/`, stripping the version suffix (e.g. `-v2`).
- MUST remove the `> **Sources**:` block (the `>` quoted paragraph at the top of the document). This metadata is for drafting context only and MUST NOT appear in the published record.
- If the target already exists in `02_record/`, overwrite it (this is a refinement of an existing record).

## 4. Commit documents
- MUST stage both the new/updated file in `02_record/` and the related `01_work/` version files in a single commit.
- Commit message example: `Publish <topic> (from <source filename>)`
- MUST push to the current branch after committing.

# Rules
- MUST NOT publish without explicit human approval ("publish", "approve", "finalize").
- MUST NOT modify existing version files in `01_work/` (they remain as-is on disk for history).
