# Writing convention for `work/` documents

These rules apply when writing or revising any file under `work/`.

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

## File Naming

Files in `work/` MUST follow the pattern `yyyy-MM-dd-<slug>-<type>-v<N>.md`.

Valid `<type>` values:

| Type | Postfix | Description |
|------|---------|-------------|
| Report | `-report` | Research findings structured as Conclusion → Findings → Evidence |
| Review | `-review` | Structured findings against a document's stated purpose |
| Guide | `-guide` | Step-by-step operational instructions |
| Proposal | `-proposal` | Design proposal with problem, solution, and tradeoffs |
| Handoff | `-handoff` | Session state snapshot for cross-session continuity |
| Update | `-update` | Periodic work status update structured around project WorkItems |

Handoff exception: the date in a handoff filename reflects the session date it was written, and MUST be updated with each new version. All other types MUST preserve the original creation date across revisions.

## Integrity

- **No modification**: MUST NOT modify files in `note/`. MUST NOT modify existing version files in `work/`.
- **Source memo**: MUST list which files, URLs, or information sources are referred at the top of the document.
- **No workspace paths in body**: MUST NOT reference workspace file paths (e.g. `note/...`, `work/...`) in the document body. The body is read outside the working environment. Use descriptive names, URLs, or document titles instead. Code repository paths and external URLs are fine. The source memo at the top is exempt from this rule.
- **Change log**: For revisions (v2+), MUST include a **Changes from v{N-1}** section at the top, directly after the source memo. Each entry is a one-line bullet summarizing what changed. Do not include detailed content.

## Style

- **Language**: MUST write everything in English regardless of the user's input language. SHOULD use plain, practitioner-friendly English.
- **No embellishment**: MUST NOT use metaphors, analogies, or illustrative examples not present in source material.
- **Source-faithful only**: MUST NOT invent specifics, implementation details, or quantities absent from source material. MUST use exact definitions and terminology from the source. MUST preserve dates from source material as-is; MUST NOT convert them to relative expressions (e.g., write "on 4/13" not "last week").
- **No editorial additions**: MUST NOT add opinions, recommendations, or commentary unless the user explicitly requests it.
- **No LaTeX**: MUST use plain text for formulas (e.g., `Daily calls = DAU x 7%`), not `$$...$$` notation.
- **Plain tables**: MUST NOT use inline formatting (bold, italic, strikethrough, etc.) in table cells.
- **Use K/M notation**: SHOULD write large numbers as 18M, 255.5K, $33K — not 18,000,000 or $33,000.
- **Arrow notation**: The `→` symbol MUST be used for chronological / timeline ordering only (e.g., `Decision Brief 4/29 → Mock server 5/6 → Release QA 6/1`). MUST NOT use `→` to express logical or causal relationships. For dependencies, use English: `blocked by`, `depends on`, `gated by`, `feeds into`, `requires`.
- **Verify math with code**: When the document involves numerical calculations (cost estimation, traffic projection, unit conversion, etc.), MUST run the arithmetic in a Python script via Bash and use the output. Do not perform multi-step math in your head.
- **Date format**: MUST write single dates as `M/D` without zero-padding (e.g., `4/27`, `1/3`); MUST write date ranges as `M/D ~ M/D` (e.g., `4/8 ~ 4/18`). MUST use `yyyy-MM-dd` only when the year must be stated explicitly (cross-year reference or disambiguation).
