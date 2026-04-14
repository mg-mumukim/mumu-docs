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

## 1. Discover sources
- Glob `note/task/*<topic>*` — request files, probe plans, WIP notes.
- Glob `note/source/*<topic>*` — source material.
- Glob `note/idea/*<topic>*` — idea entries.
- Glob `work/**/*<topic>*` — existing work output (probe reports, prior drafts).
- Read `note/context/` files referenced by CLAUDE.md.
- If a request file (`*-request*`) is found, read it first — it is the primary input.
- If external sources (Slack, Jira, Glean, Notion) are needed, search and collect them.
- Read all discovered files before writing.

## 2. Check existing versions in `work/`
- Glob `work/**/*<topic>*` to find existing versions.
- If versions exist, this is a **revision** — skip to step 3 (Write).
  - Read the latest version to find:
    - `[!NOTE]` callouts with reviewer feedback.
    - Inline comments or requests from the human reviewer.
    - Determine the next version number (e.g. v2 -> v3).
- If no versions exist, start at v1.

## 3. Write the new version

### Resolve project directory
- If this is a revision, use the same directory as the existing version.
- If the user provides a topic and a glob `work/**/*<topic>*` matches, use that directory.
- If no match is found, create a new directory under `work/`.
  - Directory name: `<scope>-<subject>` in lowercase kebab-case.
    - `<scope>`: the broader initiative or product area (e.g. `photo-review`, `mumu-profile`).
    - `<subject>`: the specific deliverable (e.g. `privacy-by-design`, `llm-cost-estimation`).
  - If the scope is ambiguous, ask the user before creating.

### File naming
- `work/<project>/yyyy-MM-dd-<document-slug>-v<N>.md`
- `<document-slug>` SHOULD match or be a subset of the directory name. Allowed to differ when a project has multiple documents (e.g. `photo-review-privacy-by-design/` can contain `privacy-by-design-section-2-2-v1.md`).
- MUST NOT modify existing version files.

### Content
- For new drafts (v1): use today's date in the filename.
- For revisions: keep the date from the existing version. Incorporate all reviewer `[!NOTE]` from the previous version.
- SHOULD use source material from `note/` as the basis.
- MUST memo which files, URLs, or information sources are referred at the top of the document.
- MUST NOT reference local file paths (`note/`, `work/`) in the document body. The body is read outside the working environment. Use descriptive names, URLs, or document titles instead. The sources memo at the top is exempt from this rule.
- For revisions (v2+): MUST include a **Changes from v{N-1}** section at the top of the document, directly after the sources memo. Each entry is a one-line bullet summarizing what changed — like a git commit message. Do not include detailed content; just state what was added, removed, or rewritten.

### Document structure
- MUST follow the structure and heading hierarchy given in the `note/` source material. MUST NOT invent new sections or reorder unless the user requests it.

### Writing convention
- **No embellishment**: MUST NOT use metaphors, analogies, or illustrative examples not present in source material.
- **No LaTeX**: MUST use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation.
- **Plain tables**: MUST NOT use inline formatting (bold, italic, strikethrough, etc.) in table cells.
- **Use K/M notation**: SHOULD write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **Source-faithful only**: MUST NOT invent specifics, implementation details, or quantities absent from source material. MUST use exact definitions and terminology from the source.
- **No editorial additions**: MUST NOT add opinions, recommendations, or commentary unless the source note explicitly requests it.
- **Verify math with code**: When the document involves numerical calculations (cost estimation, traffic projection, unit conversion, etc.), MUST run the arithmetic in a Python script via Bash and use the output. Do not perform multi-step math in your head.

## 4. Report to the user
- Summarize what changed from the previous version (for revisions).
- List which `[!NOTE]` items were addressed (for revisions).
- Note any items that could not be addressed and why.

# Rules
- MUST NOT modify files in `note/`.
- MUST NOT modify existing version files in `work/`.
- MUST create a new version file for each revision in `work/`.
- **Write everything in English.** All output files MUST be in English regardless of the user's input language.
- SHOULD use plain, practitioner-friendly English as the default style.
