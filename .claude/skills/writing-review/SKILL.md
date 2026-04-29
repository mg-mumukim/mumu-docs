---
name: writing-review
description: Evaluate an existing document against its stated purpose and produce structured findings. Use when asked to review, audit, or assess whether a document meets its goal — not when drafting new content. Triggers on "review", "리뷰", "검토", "assess", or "이 문서가 목적을 달성하는지" type requests.
---

# Usage
/writing-review <document reference> [focus area]

The key words MUST, MUST NOT, SHOULD, and MAY in this document are to be interpreted as described in RFC 2119.

The document reference can be:
- A Google Docs URL (use the downloading-gdocs skill if not already downloaded)
- A path or filename in `note/source/`

The optional focus area narrows emphasis (e.g. "cost section", "privacy obligations").

# Examples

When uncertain about findings format or severity labeling, glob `examples/` in this skill's directory. Read the filenames and pick the most relevant one by topic or format. Then read its header — it states what the example demonstrates and how much to read.

# Workflow

## 1. Resolve the target document

- If the reference is a Google Docs URL not already in `note/source/`, use the **downloading-gdocs** skill first.
- Otherwise, find the file in `note/source/` and read it in full.
- Extract from the document:
  - **Stated purpose** — verbatim quote from "About this document", "Purpose", or equivalent. If absent, infer from document type and flag it as a finding.
  - **Intended audience** — who is this written for? Their information needs define the standard for Reader Satisfaction.
  - **Claims that require external verification** — numbers, dates, attributions, or comparative assertions that reference outside data sources. List them before proceeding to step 2.

## 2. Fact-check (C — Truthfulness)

Gather the minimum supporting material needed to verify the claims flagged in step 1.

- Search `note/source/`, `note/task/`, and `work/` for documents that contain the referenced data.
- Read only what is directly relevant to the flagged claims. Do not gather broadly.
- For each flagged claim: does the supporting material confirm, contradict, or leave the claim unverified? Record the result as a finding or confirm it is clean.

## 3. Evaluate — three passes

Work through the document once per pillar. Each pass has a distinct lens.

### Pass F — Format and Structure
Read the document as a navigator: does the skeleton match the body?
- Does the table of contents accurately reflect the actual headings?
- Are sections ordered to match how a reader would use the document (context before options, data before conclusions)?
- Are heading levels consistent? Are related items grouped?
- Is length proportionate — not padded, not so compressed that a reader must go elsewhere for essential context?
- Are tables and evidence placed near the claims they support?

### Pass R — Reader Satisfaction
Read the document as the intended reader trying to act on it: can you do what it was designed to enable?
- After reading, can the intended reader make the decision, take the action, or draw the conclusion the document exists to support?
- Are open items clearly distinguished from resolved ones, with an owner and target date?
- Is scope bounded — does the reader know what is and is not covered?
- Are all inputs the reader needs present in the document, or must they go elsewhere for essential context?
- Is the right person/team named for each decision or action?

### Pass C — Content: Logic and Fluency
Read the document as a critic: is the substance correct and clearly expressed?
- **Logic**: Do conclusions follow from evidence? Are conditional claims flagged as conditional, not stated as fact? Are options or cases treated consistently across sections?
- **Fluency**: Are terms defined at first use and named consistently throughout? Are abbreviations expanded? Does the level of detail stay consistent — no unexplained shifts between high-level and implementation detail?
- **Truthfulness**: Incorporate results from step 2. Flag any claim that is unverified or contradicted.

## 4. Write the output

Save to `work/<topic>/YYYY-MM-DD-<topic-slug>-review-v1.md` (use **resolving-docs** for naming).

Each finding MUST state:
- **Finding** — one sentence starting with what is missing or wrong (not what is present).
- **Evidence** — exact quote or specific row/section from the document, and/or the conflicting source. A vague reference (e.g. "the cost section") is not sufficient.
- **Action** — what specifically needs to change. If blocked on external data or a decision, name the blocking item.

Severity levels:
- **Critical** — blocks the document from fulfilling its stated purpose as-is
- **Major** — significantly limits usefulness; requires action before the document is used for decisions
- **Minor** — reduces clarity or completeness; should be fixed but does not block use
- **Observation** — informational; no action required

Sort findings by severity (Critical first). Within severity, order by pillar: F → R → C.
Omit severity subsections that have no findings.
MUST NOT include a section on what the document does well.

### Output template

```
<!-- Sources: ... -->

# Review: <Document Title>

**Document reviewed:** <title, URL or path>
**Reviewed on:** <date>
**Reviewer:** <name>
**Focus area:** <if specified, else omit>

## Stated Purpose

> <verbatim quote>

## Verdict

<One sentence: does the document meet its stated purpose? State the primary reason if not.>

## Findings

### Critical

**1. [F] <title>**
Finding: ...
Evidence: ...
Action: ...

**2. [R] <title>**
...

### Major

**3. [C] <title>**
...

### Minor

...

### Observations

...
```

## 5. Report to the user

- State the verdict in one sentence.
- List finding numbers and titles only (no full text — that is in the file).
