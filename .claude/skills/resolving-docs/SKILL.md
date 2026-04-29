---
name: resolving-docs
description: Find existing documents and source material in `note/` and `work/`, or resolve where to create new ones. Called by writing-draft and writing-report.
---

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

# Workflow

## Find existing material

Given a `<topic>`:

- Glob `note/task/*<topic>*` — request files, probe plans, WIP notes.
- Glob `note/source/*<topic>*` — source material.
- Glob `work/**/*<topic>*` — existing work output (reports, prior drafts).
- If a request file (`*-request*`) is found, read it first — it is the primary input.

## Resolve project directory

- If this is a revision of an existing file, use the same directory.
- If the user provides a topic and a glob `work/**/*<topic>*` matches, use that directory.
- If no match is found, create a new directory under `work/`.
  - Directory name: `<scope>-<subject>` in lowercase kebab-case.
    - `<scope>`: the broader initiative or product area (e.g. `photo-review`, `mumu-profile`).
    - `<subject>`: the specific deliverable (e.g. `privacy-by-design`, `llm-cost-estimation`).
  - If the scope is ambiguous, ask the user before creating.

## File naming

- Pattern: `work/<project>/yyyy-MM-dd-<document-slug>-v<N>.md`
- `<document-slug>` SHOULD match or be a subset of the directory name. Allowed to differ when a project has multiple documents.
- Subsequent revisions increment the version: `-v2.md`, `-v3.md`.

## Version creation

- For new documents (v1): use today's date in the filename.
- For revisions (v2+): copy the latest version to the new filename, then edit only the new file. Keep the original date from the existing version. Never edit the source file.
- Mid-conversation follow-up changes are still revisions — increment the version number.
