<!-- Sources:
- note/source/2026-04-22-serving-options-gdocs.md (updated document, re-downloaded 2026-04-22)
- work/photo-review-cost-estimation/2026-04-13-llm-gpu-cost-estimation-v6.md (cost verification)
-->

# Review: Serving Options (Photo Review MVP)

**Document reviewed:** "Serving Options" tab, Photo Review MVP Engineering Docs (https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg)
**Reviewed on:** 2026-04-22
**Reviewer:** Mumu Kim

## Stated Purpose

No "About this document," "Purpose," or equivalent section exists. Inferred from document type and structure: enable all stakeholders (PM, engineering, product) to evaluate Option 1 (Batch Precompute) vs Option 2 (On-demand Sync) and reach a serving decision for Photo Review MVP.

_Absence of explicit stated purpose is flagged as a finding below._

## Verdict

The document does not close the decision: the Recommendation names Option 2 as "the MVP serving path" but two open PM constraints — nudge timing acceptability and the $50K/mo budget — are acknowledged in the same section as capable of selecting Option 1 instead. The recommendation should be conditional, not declarative.

## Findings

### Major

**1. [R] Recommendation headline is unconditional despite two unresolved constraints that could reverse it**

Finding: The Recommendation leads with "Option 2 (On-demand Sync) as the MVP serving path" as a declarative statement, but two caveats listed immediately below identify unresolved PM decisions that would select Option 1 over Option 2.

Evidence: Recommendation section headline: "**Option 2 (On-demand Sync) as the MVP serving path**, with caveats below." Caveat 1 (UX): "If nudge timing is the highest priority, Option 1 is the better path despite its infrastructure overhead." Caveat 2 (budget): Open questions for PM: "Monthly budget is $50K/mo. Can Option 2 fit within it? At current estimate ($66K), no." At-a-glance table, Budget row: "$50K/mo cap. Option 2 exceeds unless actual trigger rate is lower than assumed."

Action: Reframe the Recommendation headline as conditional. State the two decision branches explicitly: "Option 2 if PM accepts a delayed nudge (~30s) and budget above $50K/mo; Option 1 if nudge must fire at app open or budget is capped at $50K/mo." A decision-maker who reads only the Recommendation headline will proceed with Option 2 before the PM questions are answered.

### Minor

**2. [R] Stated purpose is absent**

Finding: The document has no framing section stating its scope or intended use, so readers must infer both the goal and the out-of-scope items from context.

Evidence: The document body begins at "Part 1: Decision Summary (for all stakeholders)" with no preceding "About this document" or "Purpose" section. The previous version of this document had an "About this document" section that stated the purpose verbatim.

Action: Add 1-2 sentences above Part 1 stating what decision this document is designed to close and what is explicitly excluded (e.g., Hybrid as a primary option, True Realtime).

**3. [F] "Timeline" TOC entry in Part 3 links to a non-existent section**

Finding: The TOC includes a "Timeline" entry pointing to a fragment anchor (`#heading=h.3feo6fqm9zg3`) that has no corresponding heading anywhere in the document body.

Evidence: TOC item: "[Timeline](#heading=h.3feo6fqm9zg3)." Part 3 body headings in sequence: "Architecture," "Option 1: Batch flow," "Option 2: On-demand flow," "Measured numbers," "Gemini Flash-Lite sync latency," "Gemini Batch turnaround," "Cost," "Data not yet collected," "Risk comparison." No "Timeline" heading exists.

Action: Either add a Timeline section to Part 3, or remove the TOC entry.

**4. [C] Timeline estimate is identical for both options despite SOUP LOE being unverified**

Finding: The At-a-glance table shows "5-6 wks + 1 wk QA" for both Option 1 and Option 2, but "SOUP backend LOE per option" is listed in the "Data not yet collected" table as unverified.

Evidence: At-a-glance, "Time to ship" row: "MGAI + SOUP parallel (contract finalized). 5-6 wks + 1 wk QA." (identical for both columns). "Data not yet collected" table: "SOUP backend LOE per option — Validates whether SOUP effort differs between options." The previous version of this document differentiated: 4-5 weeks for Option 2 vs 6-8 weeks for Option 1.

Action: Either confirm equal SOUP LOE with Henry Au before publishing, or add a note to the Timeline row acknowledging that SOUP effort per option is pending validation.

**5. [C] Batch turnaround dry run used gemini-2.5-pro; production cost estimate uses Flash-Lite pricing**

Finding: The Batch turnaround section reports data from a dry run using gemini-2.5-pro, but the Cost section calculates Option 1 batch cost at $0.0043/call — which is the 50% Batch API discount applied to the Flash-Lite rate ($0.0086). The document does not state which model Option 1 will use in production.

Evidence: Measured numbers, Gemini Batch turnaround: "Source: LP-532 dry run (2026-04-14), gemini-2.5-pro via Vertex Batch." Cost table: "Per-call cost (batch, 50% off) | $0.0043 | Gemini Batch API pricing." Sync latency section model: "gemini/gemini-3.1-flash-lite-preview." Gemini Pro pricing is substantially higher than Flash-Lite, so if Option 1 uses Pro in production, $0.0043 understates the batch cost.

Action: Confirm in the Cost section which model Option 1 will use for production batch. If Flash-Lite, the $0.0043 figure is correct. If Pro, recalculate.
