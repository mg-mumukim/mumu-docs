<!--
Decision document review (serving options for an ML feature).
Shows: SKILL.md-canonical output format — Stated Purpose (verbatim quote) → Verdict (one sentence) → ## Findings → ### Critical/Major/Minor with **N. [pillar] title** entries, each with Finding: / Evidence: / Action: fields.
Use to calibrate: the canonical review output structure and finding format.
Read to: one line before ### Major (captures Stated Purpose, Verdict, and all Critical findings).
  grep -n "^### Major" <this-file> | head -1
-->

<!-- Sources:
- note/source/2026-04-22-serving-options-gdocs.md (primary document)
- work/photo-review-cost-estimation/2026-04-13-llm-gpu-cost-estimation-v6.md (cost verification)
- note/source/2026-04-22-photo-review-status-update-decision-points-apr-15-to-22-gdocs.md (decision timeline context)
-->

# Review: Serving Options (Photo Review MVP)

**Document reviewed:** "Serving Options" tab, Photo Review MVP Engineering Docs (https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg)
**Reviewed on:** 2026-04-22
**Reviewer:** Mumu Kim

## Stated Purpose

> "This document outlines the remaining trade-offs, assumptions, and open questions needed to close that decision [between Option 1 and Option 2]."

## Verdict

The document does not meet its stated purpose: the Suggestion row — the document's primary decision output — is empty, and the cost threshold used to frame the decision in the Decision criteria section is built on a trigger model the document itself identifies as incorrect for Option 2.

## Findings

### Critical

**1. [R] Suggestion row empty — primary decision output missing**

Finding: The document cannot close the decision between Option 1 and Option 2 because both cells in the Suggestion row contain only a placeholder.

Evidence: Options in one table, Suggestion row, both columns: "[To be filled after P0/P1 data arrives, target 4/23 EOD]." The only data dependency cited as blocking this row — P0 #1 (latency) — is marked Done in the Engineering evidences section: "P0 #1 — collected 2026-04-22 from PF database (N=2,486 coaching generations, prompt v4+, outlier-excluded)."

Action: Fill the Suggestion row. P0 #1 is resolved; the Decision criteria priority table already maps each stakeholder axis to an option. At minimum, produce a conditional recommendation: one branch for each remaining open P0/P1 answer (P0 #2, P0 #3, P1 #9).

### Major

**2. [C] Option 2 cost in Decision criteria is derived from a trigger model that does not match Option 2's definition**

Finding: The $66K/mo figure used as the Option 2 cost threshold in the Decision criteria table is calculated from the profile-change + new-login trigger rate (7% of DAU/day), but Option 2 fires on every Profile Home open by an eligible user — a view rate the document itself flags as "likely much higher."

Evidence: Decision criteria table, "Monthly budget cap" row: "if > $66K/mo → Both options fit; cost is not the deciding factor." Options in one table, Cost row, Option 2 column: "Standard Gemini × N_profile_home_views. [P0 #2 pending]." Engineering evidences, P0 #2: "this is profile change rate, not Profile Home view rate — view rate is likely much higher." Cost verification: $66K/mo = 7.67M calls × $0.0086, where 7.67M = 3.65M DAU × 7%/day × 30 days (from LLM + GPU Cost Estimation v6, Section 3.3) — a trigger-rate figure, not a view-rate figure.

Action: Replace the single $66K/mo threshold in the Decision criteria table with a conditional: one row assuming N_views ≈ N_triggers ($66K/mo), and one row acknowledging that if P0 #2 confirms view rate > trigger rate, Option 2 monthly cost exceeds $66K by a proportional amount. A reader who only consults the Decision criteria table will incorrectly conclude that cost is not a deciding factor.

**3. [F] Hybrid has no column in the main Options comparison table**

Finding: The Hybrid (Option 1+2) configuration is named and analyzed in both the Timeline and Risk sections but is absent from the main "Options in one table," leaving readers without a consolidated trade-off view of Hybrid alongside Option 1 and Option 2.

Evidence: Timeline comparison: "Option 1+2 Hybrid" table, total 7-9 weeks. Risk comparison, Side-by-side summary: three-column table (Option 1, Option 2, Hybrid). Options in one table: two columns only (Option 1 and Option 2).

Action: Either add a Hybrid column to the main Options table, or add an explicit statement in the Options section that Hybrid is excluded from the primary decision — with a reason (e.g., scope, timeline) — so readers know the omission is intentional.

### Minor

**4. [F] TOC entries do not match any section headings in the document body**

Finding: All TOC entries that follow "About this document" refer to headings that do not exist in the body, making navigation unreliable.

Evidence: TOC lists: "Cost Dashboard," "Summary," "Constraints," "Options," "Remaining Questions." Actual body headings: "Flow diagram," "Options in one table," "Decision criteria," "Timeline comparison," "Risk comparison," "Remaining product questions," "Engineering evidences." Neither "Cost Dashboard," "Summary," nor "Constraints" appear as headings anywhere in the body.

Action: Update the TOC to list the actual section headings in the document body.

**5. [C] P1 #6 (Gemini sync QPS quota) shows no result despite passing its own target date**

Finding: The Engineering evidences section lists P1 #6 with a target of 4/22 and owner Brandon/Mumu but records no result, leaving Option 2's technical viability at production scale unconfirmed.

Evidence: Engineering evidences, P1 #6: "[Brandon Choi / Mumu Kim — target 4/22]. Check Google Cloud Console for Tinder-account Gemini sync quota (RPM, TPM). Cross-reference with estimated peak QPS from Profile Home entry rate (P0 #2). Reference: CRS v2 noted prior concerns — 'NanoBanana struggling at 1 QPS'." All numbers collection plan, P1 #6 status: "Unknown."

Action: Record the current quota status in the P1 #6 section before the 4/23 decision meeting.
