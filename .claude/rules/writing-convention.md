# Writing convention for `work/` documents

These rules apply when writing or revising any file under `work/`.

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

## Integrity

- **No modification**: MUST NOT modify files in `note/`. MUST NOT modify existing version files in `work/`.
- **Source memo**: MUST list which files, URLs, or information sources are referred at the top of the document.
- **No local paths in body**: MUST NOT reference local file paths (`note/`, `work/`) in the document body. The body is read outside the working environment. Use descriptive names, URLs, or document titles instead. The source memo at the top is exempt from this rule.
- **Change log**: For revisions (v2+), MUST include a **Changes from v{N-1}** section at the top, directly after the source memo. Each entry is a one-line bullet summarizing what changed. Do not include detailed content.

## Style

- **Language**: MUST write everything in English regardless of the user's input language. SHOULD use plain, practitioner-friendly English.
- **No embellishment**: MUST NOT use metaphors, analogies, or illustrative examples not present in source material.
- **Source-faithful only**: MUST NOT invent specifics, implementation details, or quantities absent from source material. MUST use exact definitions and terminology from the source.
- **No editorial additions**: MUST NOT add opinions, recommendations, or commentary unless the user explicitly requests it.
- **No LaTeX**: MUST use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation.
- **Plain tables**: MUST NOT use inline formatting (bold, italic, strikethrough, etc.) in table cells.
- **Use K/M notation**: SHOULD write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **Verify math with code**: When the document involves numerical calculations (cost estimation, traffic projection, unit conversion, etc.), MUST run the arithmetic in a Python script via Bash and use the output. Do not perform multi-step math in your head.
