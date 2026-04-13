---
name: publish
description: Copy a finalized `work/` document to `note/context/` for cross-project reference.
---

# Usage
/publish <topic or filename> [optional request]

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

Use `/publish` only when a completed document needs to be referenced by other projects or drafts.
The latest version in `work/` is already the canonical final document; publishing is not required for completion.

# Workflow

## 1. Resolve and verify
- If a topic is given, glob `work/**/*<topic>*` and select the latest version.
- Read the resolved file in `work/`.
- MUST confirm there are no unresolved `[!NOTE]` callouts (warn the user if any remain).

## 2. Apply optional request
- If the user appended a request (e.g. "polish", "fix typos"), apply it to the content before writing. MUST follow the draft skill's writing conventions.

## 3. Write to `note/context/`
- Copy the file to `note/context/`, stripping the version suffix (e.g. `-v2`).
- MUST remove the `> **Sources**:` block (the `>` quoted paragraph at the top). This metadata is for drafting context only.
- If a target already exists in `note/context/`, overwrite it.

## 4. Commit documents
- MUST stage the new file in `note/context/` in a single commit.
- Commit message example: `Publish <topic> to context (from <source filename>)`
- MUST push to the current branch after committing.

# Rules
- **Write everything in English.** All output files MUST be in English regardless of the user's input language.
- MUST NOT publish without explicit human approval ("publish", "approve", "finalize").
- MUST NOT modify existing version files in `work/`.
