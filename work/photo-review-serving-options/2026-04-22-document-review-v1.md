<!-- Sources:
- note/source/2026-04-22-serving-options-gdocs.md (Serving Options tab, downloaded 2026-04-22)
- work/photo-review-cost-estimation/2026-04-13-llm-gpu-cost-estimation-v6.md (prior cost research)
- note/task/2026-04-10-photo-review-llm-cost-estimation-request.md (cost estimation request)
- note/source/2026-04-14-serving-options-docs-template.md (original template)
- note/source/2026-04-16-ml-photo-review-serving-options-trideep-thread.md (Trideep Slack thread)
- work/photo-review-serving-options/2026-04-15-trideep-decision-readiness-v4.md
-->

# Review: Serving Options Document (Photo Review MVP)

**Document reviewed:** Serving Options tab, Photo Review MVP Engineering Docs  
**Last updated in doc:** 2026-04-15 (with a 2026-04-21 update note in "About this document")  
**Review date:** 2026-04-22  
**Reviewer:** Mumu Kim

---

## Stated Purpose

The document states its own goal in the "About this document" section:

> "This document outlines the remaining trade-offs, assumptions, and open questions needed to close that decision [between Option 1 and Option 2]."

This review checks whether the document meets that goal.

---

## What the Document Covers (Purpose Met)

**1. Trade-offs — covered.**  
The Options comparison table addresses latency, UX, cost, dependencies, engineering decisions, product decisions, privacy/legal, and a "caveat that kills the option" row. A separate Risk section ranks risks by likelihood × impact per option and includes the Hybrid configuration. A Timeline section gives week-level estimates per option with per-step breakdowns and owners.

**2. Open questions — covered with tracking structure.**  
The Engineering evidences section presents a numbered collection plan (P0 #1–4, P1 #5–9, P2 #10–12) with status, owner, and target date per item. The Remaining product questions section mirrors this for PM-side decisions.

**3. Eliminated option rationale — covered.**  
Option 3 (True Realtime, 0-5s) is removed with five explicit reasons in a table under "About this document." Revival conditions are noted.

**4. Latency data — present and interpreted.**  
P0 #1 (Gemini Flash-Lite sync latency) is complete: p50 = 17.5s, p95 = 25.0s, p99 = 30.5s, N=2,486 coaching generations from 2026-04-20 to 2026-04-22. The document correctly concludes Option 2 is technically feasible but requires graceful fallback for ~1% timeout at p99.

**5. Cost numbers — partially present.**  
Per-call costs ($0.0086 sync, $0.0043 batch) and Option 1 monthly cost (~$33K/mo, test region) are cited and correctly match prior cost research. These are marked as "Done" in the collection plan.

---

## Gaps Against Stated Purpose

### 1. "Suggestion" row is empty — the document's own key output is missing

The Options comparison table has a "Suggestion" row. Both cells contain the same placeholder: `[To be filled after P0/P1 data arrives, target 4/23 EOD]`. P0 #1 (latency) is done as of 4/22. The document provides a well-structured Decision criteria section with a priority table that maps each stakeholder axis to an option, but stops short of a provisional recommendation. The document cannot close the decision it is designed to support until this row is filled.

### 2. Option 2 monthly cost uses the wrong trigger model

The collection plan marks Option 1 monthly cost ($33K) as "Done" and Option 2 monthly cost as "Partially known: $66K/mo IF every trigger = view." The $66K figure comes from the prior cost research, which was built on trigger rate = 7% of DAU/day (profile change + new login). However, the document defines Option 2's trigger as "every Profile Home open by an eligible user" — a view rate, not a profile-change rate. The document itself notes: "view rate is likely much higher" (Engineering evidences section, P0 #2). The $66K/mo therefore underestimates Option 2 steady-state cost by an unknown factor. The true figure awaits P0 #2 (Profile Home view rate/DAU/day from Tinder data team, target 4/23).

Until P0 #2 arrives, the cost comparison in the decision criteria table ("if > $66K/mo → both options fit; cost is not the deciding factor") may not hold, because the correct Option 2 monthly cost may be substantially higher than $66K.

### 3. Hybrid option absent from the main Options comparison table

The Timeline and Risk sections both include Hybrid (Option 1+2) as a third option. The main "Options in one table" covers only Option 1 and Option 2. If Hybrid is a real candidate — which the timeline and risk discussions imply — it needs a column in the main comparison table, or the document should explicitly state that Hybrid is not a standalone option for the current decision.

### 4. Navigation: TOC headings do not match document body headings

The table of contents at the top links to six sections: "About this document," "Cost Dashboard," "Summary," "Constraints," "Options," and "Remaining Questions." The actual document body uses different headings: "Flow diagram," "Options in one table," "Decision criteria," "Timeline comparison," "Risk comparison," "Remaining product questions," and "Engineering evidences." The "Cost Dashboard" and "Summary" TOC entries appear to be inherited from the original document template and do not correspond to any heading in the current document. This makes navigation unreliable for readers who use the TOC.

### 5. Four critical data points remain open; all block the final decision

The collection plan shows the following as not yet complete:

| # | Item | Target |
|---|------|--------|
| P0 #2 | Profile Home view rate (drives Option 2 cost) | 4/23 |
| P0 #3 | Gemini Batch API turnaround (drives Option 1 cadence) | 4/23 |
| P1 #5 | Precompute set criteria (drives Option 1 cost ceiling) | 4/23 |
| P1 #9 | Staleness tolerance — PM decision (determines Option 1 batch cadence) | 4/23 |
| P2 #11 | Monthly budget cap (frames the cost decision) | 4/23 |

All five gate the "Suggestion" row. The document is correctly structured to receive these inputs, but cannot close the decision before 4/23 at the earliest.

---

## Cross-Reference Against Prior Cost Research

Prior work (LLM + GPU Cost Estimation v6) used the following numbers, all of which appear correctly in this document:

| Item | Prior research | This document |
|------|---------------|---------------|
| Per-call cost, sync | $0.0086 | $0.0086 ✓ |
| Per-call cost, batch | $0.0043 (50% discount) | $0.0043 ✓ |
| Test region DAU | 3.65M | 3.65M ✓ |
| Steady-state trigger rate | 7% DAU/day | 7% DAU/day ✓ |
| Monthly calls, test region | 7.67M | 7.67M ✓ |
| Option 1 monthly cost, test region | $33K | ~$33K ✓ |
| Option 2 monthly cost, test region | $66K (using trigger rate = 7%) | $66K, flagged as trigger-model mismatch ✓ |
| GPU cost (batch, test region) | $50/mo | Marked as appendix / no longer load-bearing ✓ |

The prior research assumed the trigger rate for both options was profile change + new login (7% DAU/day). This document correctly identifies that for Option 2, the relevant rate is Profile Home view rate, not profile-change rate. The prior research figures are still valid as inputs for Option 1 (precompute covers the same user population), but the Option 2 cost number from prior research should not be used as a decision input without the P0 #2 correction.

---

## Summary

The document is well-structured for its stated purpose and covers trade-offs, risks, timeline, and most assumptions in sufficient depth. The decision framework (priority table, open questions collection plan, feasibility criteria) is sound.

It cannot close the decision today because: (1) the "Suggestion" row is empty pending 4/23 data, and (2) Option 2's actual cost is unknown due to a trigger-model mismatch. The remaining gap is data-driven, not structural — the document will be decision-ready once P0 #2, P0 #3, P1 #5, P1 #9, and P2 #11 are filled in.

The two structural issues — Hybrid column absent from the main table, and TOC-heading mismatch — are minor but worth fixing before the document is shared for decision sign-off.
