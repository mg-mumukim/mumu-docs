---
name: publish
description: Publish an approved `01_work/` draft to `02_record/`, stripping the version suffix and committing.
---

# Usage
/publish <topic or filename> [optional request]

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

The argument can be:
- A topic to publish the latest version (e.g. `/publish llm-cost-estimation`)
- An existing filename in `01_work/` (e.g. `/publish 01_work/2026-04-10-photo-review-llm-cost-estimation-v2.md`)
- Either form with an optional trailing request (e.g. `/publish llm-cost-estimation finalize the conclusion`)

# Workflow

## 1. Resolve and verify
- If a topic is given, glob `01_work/*<topic>*` and select the latest version(s). Multiple files MAY form a single logical unit.
- Read the resolved file(s) in `01_work/`.
- MUST confirm there are no unresolved `[!NOTE]` callouts (warn the user if any remain).

## 2. Apply optional request
- If the user appended a request (e.g. "finalize", "polish", "fix typos"), apply it to the content before writing to `02_record/`. MUST follow the draft skill's writing conventions.

## 3. Move to `02_record/`
- Copy each file to `02_record/`, stripping the version suffix (e.g. `-v2`).
- MUST remove the `> **Sources**:` block (the `>` quoted paragraph at the top). This metadata is for drafting context only and MUST NOT appear in the published record.
- If a target already exists in `02_record/`, overwrite it (this is a refinement of an existing record).

## 4. Commit documents
- MUST stage all new/updated files in `02_record/` and the related `01_work/` version files in a single commit.
- Commit message example: `Publish <topic> (from <source filename>)`
- MUST push to the current branch after committing.

# Rules
- MUST NOT publish without explicit human approval ("publish", "approve", "finalize").
- MUST NOT modify existing version files in `01_work/` (they remain as-is on disk for history).
