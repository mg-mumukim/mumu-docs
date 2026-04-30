<!-- sources:
- note/photo-review/2026-04-30-photo-review-jira-lp.md
- note/photo-review/2026-04-30-photo-review-aura-squad-slack.md
- note/photo-review/2026-04-22-photo-review-status-update-decision-points-apr-15-to-22-gdocs.md
- note/photo-review/2026-04-22-serving-options-gdocs.md
- note/photo-review/2026-04-22-backend-contract-gdocs.md
- note/photo-review/2026-04-16-ml-photo-review-serving-options-trideep-thread.md
- note/photo-review/2026-04-27-tinder-core-rev-profile-coaching-workshop-notion.md
- Notion: MGAI PM-TL Sync 4/30 (351ce00cd35480d6951bffbe26111be7)
- Notion: MGAI PM-TL Sync (Juniper) 4/24 (34bce00cd354809f91d8fed2c736f9e9)
- Notion: MGAI PM-TL Sync (Juniper) 4/17 (344ce00cd35480a78d91fe63d09c9ca5)
- Spec Kick-off Notes (4/27, 4/29): https://docs.google.com/document/d/1VxtA1qjkyZshECqtrLOL7NDe5G8CJiV6Mltmm6HHurw/
- PM Status Update & Decision Points (4/15~4/22): https://docs.google.com/document/d/1VPSzigsFo83AvBCvKO911Efwz11IrVDyrHtbPnZn1Eg/
- PRD WIP Photo Review & Feedback (v17): https://gotinder.atlassian.net/wiki/spaces/SOUP/pages/1898905601/
-->

## Changes from v10

- Header corrected: Biggie Choi as Lead, Mumu Kim as Advisor (v8 and earlier were correct; v9/v10 incorrectly listed Mumu as Lead)
- Per-section contributor/counterpart attribution restored; Trideep Rath correctly attributed to Tinder ML/Gen AI in M1, no longer grouped with SOUP
- Spec kick-off product decisions expanded from shorthand to readable sentences with behavioral context
- SOUP effort estimation expanded to include MGAI planning dependency and Q2 timing implications
- KB injection blocker: causal arrow replaced with descriptive language; impact on Eval 6 timing clarified
- Exited this period: Link Lee's contributions before transfer restored

---

# AURA Squad — Photo Review Status Update (4/16 ~ 4/30)

**Lead:** Biggie Choi  |  **Advisor:** Mumu Kim

---

## Mainstream 1: Engineering Kickoff

**PoC:** Brandon Choi
**Tinder counterpart (ML/Gen AI):** Trideep Rath (Staff SWE ML)
**Tinder counterpart (SOUP):** Ryan Burns (Sr. Eng Manager) · Nathan Lamb (TL) · Henry Au (SWE III) · Evan Keys (Sr. SWE Backend, Canvas)
**Tinder counterpart (PM/Design):** Jaini Shah (PM) · Nancy Wu (Design)

> Option 2 (on-demand Gemini) confirmed ~4/22; Decision Brief delivered 4/29; SOUP review in progress. Timeline: Decision Brief 4/29 → Mock server + design doc 5/6 → ML deployment & testing ~5/11 week → SOUP Q2 capacity late May → **Release QA 6/1**.

**Done**

- **Serving option: Option 2 (on-demand) confirmed** (Brandon + Trideep) — [Serving Options / Decision Brief](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg)
    - Option 2 selected over Option 1 (batch precompute): p50 = 17.5s, p95 = 25.0s (within 30s budget); true realtime (<5s) eliminated as infeasible
    - Cost upper bound ~$66K/mo for the test region vs. ~$33K/mo for Option 1; No Coaching Store, no batch pipeline — every app open triggers fresh generation
- **Option 2 Kick-off Decision Brief** (Brandon) — [Decision Brief](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.h0gjhkoyvzcu#heading=h.6oqa8dr3joz7)
    - Track A documents product decisions still to be finalized; Track B documents the engineering contract for SOUP to review
    - Sent to SOUP engineers (Ryan Burns, Nathan Lamb, Henry Au) and Evan Keys (Canvas); Evan Keys completed his review with no blockers; Henry Au's review is still pending as of 4/30
- **PRD team-wide review 4/28** (Juniper) — engineering sync blockers resolved; LP-613 closed
- **Spec kick-off meetings** (Jaini Shah with SOUP and design team, 4/27 and 4/29) — [Spec Kick-off Notes](https://docs.google.com/document/d/1VxtA1qjkyZshECqtrLOL7NDe5G8CJiV6Mltmm6HHurw/)
    - Insight cadence: users receive at most 3 insights lifetime; Segment 1 (6+ photos, low/medium diversity) has a 1-day cooldown between consecutive insights; Segment 2 (<6 photos) has no cooldown, so a fresh insight can appear on the next app open immediately after a photo action. Both values are backend-configurable levers.
    - Opt-out: no true opt-out in MVP; users can dismiss individual insights, and the combination of max cap and cooldown is expected to prevent over-coaching. MVP experiment is scoped to safe geos only.
    - Experiment structure: Control arm = new profile home with no photo review feature; Variant 1 = new profile home with photo review enabled, where benefit and title copy variants are randomized across sessions. Post-launch, specific lever combinations can be adjusted on the MGAI/ML side without requiring a new client lever.

> "Option 2 for MVP serving" — by Brandon + Trideep, ~4/22. Reason: covers every user regardless of batch timing, no stored inferences eliminates privacy/retention complexity, single endpoint simpler to build and operate. Trade-off: ~30s generation delay means nudge arrives while user is mid-swipe, not at app open.

**In Progress**

- **SOUP effort estimation** (Ryan Burns) — estimate sheet opened; Evan Keys and Henry Au held a working session on 4/30 to fill in the SOUP engineering scope. SOUP Q2 capacity is confirmed for late May, which means the MG AI development schedule also needs to be locked soon — the serving option was not finalized until this week, so MGAI planning has not yet started.
- **PRD Part II** (Juniper) — Track A final text in progress; bottom sheet direction is aligned to the shortest path, but two design items require Nancy Wu's confirmation before this section closes: (1) bottom sheet image approach — Option 1 is a single catch-all image that works for both add and replace CTAs, while Option 2 uses separate images per primary action; and (2) the fallback copy approach, where three options are still under evaluation.

**To Do**

- **Mock server deployment + ML server design doc** (Brandon) — target 5/6 (Wed) PT morning, contingent on contract alignment completing this week
    - Trideep flagged an events logging gap and Mumu flagged a user ID gap; both need to be addressed in the design doc before handoff to SOUP
    - risk: contract alignment not yet fully confirmed; delay here pushes the design doc and downstream integration testing
- **MGAI development + deployment schedule** (Mumu) — not yet planned; scheduling needs to happen the week of 4/30
    - risk: SOUP Q2 capacity is confirmed for late May; if MGAI planning is delayed, the implementation window between MGAI completion and the SOUP integration window compresses

**Open decisions (external input needed)**

- [ ] **Monthly budget for Option 2**: the ~$66K/mo test region upper bound exceeds the $50K cap; actual cost depends on Profile Home view frequency, which requires a data pull from the Tinder data team (via Juniper/Harriet)
- [ ] **Gemini sync QPS quota** on Tinder's Vertex account: Option 2 requires confirmation that the Vertex account can handle peak Option 2 traffic without throttling (Brandon to follow up)
- [ ] **Bottom sheet image approach** (Nancy Wu): Option 1 is a catch-all image that works for both add and replace CTAs; Option 2 uses a separate image for each primary action; unresolved as of 4/29
- [ ] **Fallback copy approach** (Nancy Wu): 3 options under evaluation — (1) fall back to the control generic copy, (2) fall back to functional copy, (3) custom copy per scenario (e.g., ineligible user, dismissed, maxed out, dismissed without taking action)
- [ ] **Non-LLM copy finalization timeline**: a date is needed for when the hardcoded and template copy must be ready, so that copy asset preparation can be planned in parallel with the LLM copy track
- [ ] **Engineering sync timing**: a dedicated sync with SOUP engineering needs to be scheduled; this was raised as an open item in the spec kick-off on 4/29 but no date has been set

---

## Mainstream 2: Demo Eval

**PoC:** Biggie Choi (ML/KB), Juniper Han (Product)
**Contributors:** Parker Cho (Prompt Factory), Ken Lee (Eval support)

> Eval 4 complete and addressed; Eval 5 skipped because the KB needs more time; Eval 6 KB prep is in progress, targeting the week of 5/4. MVP LLM model selection remains open.

**Done**

- **Eval 4 complete** (Flash-Lite vs Pro comparison, Juniper) — key finding: Flash-Lite showed comparatively low accuracy on repetitive photo detection, with the Pro model both missing photos it should flag and flagging photos it should not
    - Prompt v4.6.1 deployed (Biggie): replace/swap actions now target only non-representative photos within a repetitive cluster; the diversity lever condition was simplified; two or more blank or meaningless photos are now treated as a repetitive cluster
    - UI fixes (Parker): repetition indicator priority logic implemented so the lowest-RSRR cluster is surfaced first; replacement icon display range corrected to match Figma spec

**Skipped**

- **Eval 5** (KB v1.0 + prompt v4.6.1 Flash-Lite coaching evaluation) — deferred by Juniper; the Knowledge Base requires more time to stabilize, and running the evaluation against an unstable KB would produce misleading results about Flash-Lite's overall performance rather than isolating KB quality

**In Progress**

- **Eval 6 KB expansion** (Biggie) — KB v2.0 has been statistically validated (Cohen's d, Bonferroni correction, gender × age stratification) and registered in Prompt Factory; prompt v5.0.0 restructuring is running concurrently
    - Diversity taxonomy finalized in KB v2.0: 5 dimensions (Skills/Activities, Social Life, Lifestyle/Interests, Diverse Locations, Diverse Poses); the earlier Interests & Lifestyle split was merged into one dimension, with Activities kept as a separate dimension
    - Blocker: injecting the full KB v2.0 produces 13 or more atoms per profile, making the LLM call too large to run cleanly; a usage logic overhaul to select a relevant subset of atoms per profile is required before Eval 6 can run
    - Biggie is driving both the KB usage logic and the prompt restructure while Juniper is traveling; Eval 6 target is the week of 5/4
- **Photo Review copy/tone alignment** (Juniper, LP-610) — preparing a polished copy baseline before Eval 6 so the evaluation covers both KB quality and copy/tone simultaneously; in progress

**Discussion (internal)**

- [ ] **MVP LLM API model**: Gemini Pro was kept open as a fallback alongside Flash-Lite in the 4/29 meeting — this is a scope lever for the test region rollout, not a signal that Flash-Lite quality is failing. Kevin Celustka is estimating cost. Ben Shin has separately suggested OpenAI API via Bedrock as an alternative; no decision yet.
- [ ] **Post-MVP optimization stream**: Trideep noted that the optimization work should run in parallel after MVP ships — not all coaching features need to be LLM-generated, and a simple classification step before the LLM call is one option worth exploring

---

## Substream: Manual DLP
*(Parker Cho, Ken Lee)*

**Done**

- **Dataset v5 (S3) deleted** (Ken, LP-565) — the rSRR Predictor v1 training dataset has been removed from S3; no AURA demo impact was confirmed with Parker and Brandon before deletion proceeded
- **ldm-models repo cleanup** (Ken, LP-594 Pending) — branch cleanup and final PR sharing are in progress

**In Progress**

- **AURA demo data DLP** (Parker, LP-617) — inactive users (last active 1+ month ago) have been identified as deletion targets; ~470 active users will be preserved; dry run complete
    - Eval pool impact: ~160 profiles will shrink to ~140 after execution; no other system impact expected
    - Parker returns 5/6; execution is planned for that date

---

## Substream: rSRR Predictor *(post-MVP research track)*
*(Ken Lee)*

**Done**

- **v1 training + evaluation complete** (Ken, LP-607) — Qwen3-VL 8B fine-tuned with LoRA for raw rSRR regression; pairwise profile-ranking evaluation shows performance is not worse than the existing profile-ranker baseline; additional ablations completed across regression vs. percentile classification and SmartPhoto alignment

**In Progress**

- **Trideep-facing external brief** (Ken, LP-616) — Trideep requested a lesson-learned writeup covering training findings and how they relate to SmartPhoto; writing is in progress; Ken also identified the face close ratio data in `tinder_events_delta.media_yolo` as a potential future AURA input and has shared it with the team

---

## Exited this period

- **Link Lee** — transferred to Tailor (Chemistry) project ~4/21; prior to leaving, completed analysis tasks, confirmed the rSRR Predictor v1 data path with Parker and Brandon before dataset deletion, and handed off outstanding items
