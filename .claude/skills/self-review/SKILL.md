---
name: self-review
description: Review a work/ document against writing conventions and internal consistency. Reports issues without modifying any files.
---

# Usage
/self-review [filename or topic]

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

The argument can be:
- A filename in `work/` (e.g. `/self-review 2026-04-14-smartphoto-v4-engineering-report-v3.md`)
- A topic slug (e.g. `/self-review smartphoto-v4`) — resolves to the latest version under `work/<topic>/`
- Omitted — if the user's prior message clearly refers to a specific file, infer it

MUST NOT modify any file. Output is review findings only.

# Workflow

## 1. Locate the target document

- If a filename is given, resolve it under `work/`.
- If a topic slug is given, Glob `work/**/*<slug>*` and select the highest version number (e.g. v3 over v2).
- If the target cannot be determined, ask the user before proceeding.
- Read the target document in full.

## 2. Load review criteria

Read `.claude/rules/writing-convention.md` in full. This is the authoritative rule set.
Also note any project-level overrides in `CLAUDE.md`.

## 3. Check integrity rules

For each rule, check the document and record any violations:

| Rule | What to check |
|------|--------------|
| Source memo | A source list appears at the very top (before any prose). All cited sources are listed. |
| Change log | For v2+, a "Changes from v{N-1}" section appears directly after the source memo. Each entry is one line. |
| Change log accuracy | Each bullet in the change log matches an actual change in the document body. No entries are missing or mislabeled (e.g., wrong finding numbers). |
| No workspace paths in body | Body does not contain `note/...` or `work/...` paths. Source memo is exempt. Code repo paths and external URLs are allowed. |

## 4. Check style rules

| Rule | What to check |
|------|--------------|
| Language | All prose is in English. |
| No embellishment | No metaphors, analogies, or illustrative examples absent from source material. |
| Source-faithful | No invented specifics, quantities, or details not traceable to a listed source. Dates are not converted to relative expressions. |
| No editorial additions | No opinions, recommendations, or commentary unless user explicitly requested them. |
| No LaTeX | No `$$...$$` or `\(...\)` math notation. |
| Plain tables | No bold, italic, or strikethrough inside table cells. |
| K/M notation | Numbers ≥ 10,000 use K/M/B notation (e.g. 18M, not 18,000,000). |

## 5. Check internal consistency

These checks do not require the source material — only the document itself:

- **Cross-reference accuracy**: Every "See Evidence X.Y" or "See Finding X.Y" pointer resolves to a section that exists and covers the claimed content.
- **Numeric consistency**: The same quantity stated in multiple places (e.g. in a finding and in an evidence table) uses the same value and unit. Flag any discrepancy.
- **Section ordering**: Section numbers appear in ascending order in the document. No section is out of sequence.
- **Finding reference coverage**: Each key finding either has a "See Evidence X.Y" pointer or inline attribution sufficient to trace the claim.

## 6. Report findings

Group findings by category. For each issue:
- State the **rule or check** it violates.
- Quote or cite the **specific line or passage** (include line number if helpful).
- If relevant, contrast with the correct or conflicting passage.

Use this structure:

```
## Integrity
[findings, or "None"]

## Style
[findings, or "None"]

## Internal Consistency
[findings, or "None"]
```

If no issues are found in a category, write "None."
MUST NOT suggest fixes or rewrites. Report only.
