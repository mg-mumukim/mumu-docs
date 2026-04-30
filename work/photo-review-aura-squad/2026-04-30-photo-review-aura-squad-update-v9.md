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
-->

# Photo Review · AURA Squad · Status Update

**Lead:** Mumu Kim
**Period:** 4/16 ~ 4/30
**Tinder counterpart (eng):** Trideep Rath · Henry Au · Ryan Burns (SOUP)
**Tinder counterpart (PM):** Jaini Shah · Nancy (design, bottom sheet)

---

## Mainstream 1: Engineering Kickoff
*(Brandon Choi, PoC)*

> Option 2 (on-demand Gemini) confirmed ~4/22; Decision Brief delivered 4/29; SOUP review in progress. Timeline: Decision Brief 4/29 → Mock server + design doc 5/6 → ML deployment & testing ~5/11 week → SOUP Q2 capacity late May → **Release QA 6/1**.

**Done**

- **Serving option: Option 2 (on-demand) confirmed** — Brandon + Trideep sync + cross-team alignment ([Serving Options](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg))
    - p50 = 17.5s, p95 = 25.0s (under 30s budget); true realtime (<5s) eliminated; cost ~$66K/mo test region upper bound vs. Option 1 ~$33K
    - No Coaching Store, no batch pipeline; every app open triggers fresh generation
- **Option 2 Kick-off Decision Brief** (Brandon) — Track A (product decisions to finalize) + Track B (engineering contract for SOUP) ([Decision Brief](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.h0gjhkoyvzcu#heading=h.6oqa8dr3joz7))
    - SOUP engineers (Ryan Burns, Nathan Lamb, Henry Au, Evan Keys) sent Track B review
    - Evan Keys: initial review done, no blockers; Henry Au: additional review pending
- **PRD team-wide review 4/28** (Juniper) — engineering sync blockers resolved (LP-613 Done)

> "Option 2 for MVP serving" — by Brandon + Trideep, ~4/22. Reason: covers every user regardless of batch timing, no stored inferences eliminates privacy/retention complexity, single endpoint simpler to build and operate. Trade-off: ~30s generation delay means nudge arrives while user is mid-swipe, not at app open.

**In Progress**

- **SOUP effort estimation** — Ryan Burns started estimate sheet; Evan Keys + Henry Au working session 4/30
- **PRD Part II** (Juniper) — Track A final text; bottom sheet direction aligned to shortest path, Nancy final confirmation pending

**To Do**

- **Mock server deployment + ML server design doc** (Brandon) — target 5/6(Wed) PT morning, conditional on contract alignment completing this week
    - Events logging gap (Trideep flag) and user ID gap (Mumu flag) to be addressed in design doc
    - risk: contract alignment not yet fully confirmed
- **MGAI development + deployment schedule** (Mumu) — not yet planned; needs scheduling the week of 4/30
    - risk: SOUP Q2 capacity confirmed late May; late MGAI planning compresses implementation window

**Open decisions (external input needed)**

- [ ] **Monthly budget for Option 2**: ~$66K/mo test region upper bound exceeds $50K cap; Profile Home view frequency needed for actual cost (data ask: Tinder data team via Juniper/Harriet)
- [ ] **Gemini sync QPS quota** on Tinder's Vertex account: needed to confirm Option 2 can handle peak traffic without throttling (Brandon)

---

## Mainstream 2: Demo Eval
*(Juniper Han, PoC: product; Biggie Choi, PoC: ML/KB)*

> Eval 4 complete and addressed; Eval 5 skipped (KB needs more time); Eval 6 KB prep in progress targeting week of 5/4. MVP LLM model selection still open.

**Done**

- **Eval 4 complete** (Flash-Lite vs Pro comparison) — key finding: repetitive detection accuracy comparatively low in Flash-Lite; Pro missing photos it should catch, or flagging photos it shouldn't
    - Prompt v4.6.1 deployed (Biggie): replace/swap now targets only non-representative photos in repetitive cluster; diversity lever condition simplified; 2+ blank/meaningless photos treated as repetitive cluster
    - UI fixes (Parker): repetition indicator priority logic implemented; replacement icon display range corrected

**Skipped**

- **Eval 5** (KB v1.0 prompt v4.6.1 Flash-Lite coaching evaluation) — Juniper: Knowledge Base improvement requires more time; rushing with an unstable KB would produce an inaccurate evaluation of Flash-Lite performance overall

**In Progress**

- **Eval 6 KB expansion** (Biggie) — KB v2.0 statistically validated (Cohen's d, Bonferroni, gender×age stratification), registered in Prompt Factory; prompt v5.0.0 restructuring in progress concurrently
    - Issue: injecting full KB v2.0 results in 13+ atoms per profile → LLM call becomes too large; usage logic overhaul needed before Eval 6
    - Biggie to drive with Brandon's support while Juniper is out; Eval 6 target: week of 5/4
- **Photo Review copy/tone alignment** (Juniper, LP-610) — polished baseline preparation before Eval 6 copy evaluation; in progress

**Discussion (internal)**

- [ ] **MVP LLM API model**: Gemini Pro kept open as fallback alongside Flash-Lite (4/29 meeting decision); not the same as saying Flash-Lite quality is failing — this is a scope lever for test region rollout. Kevin estimating cost. Ben Shin separately suggested OpenAI API via Bedrock; no decision yet.
- [ ] **Post-MVP optimization stream**: Trideep noted MVP first, then parallel optimization track; not all coaching features need to be LLM-generated (e.g., a simple classification step before LLM call is possible)

---

## Substream: Manual DLP
*(Parker Cho, Ken Lee)*

**Done**

- **Dataset v5 (S3) deleted** (Ken, LP-565) — rSRR Predictor v1 training dataset removed; no AURA demo impact confirmed with Parker and Brandon before deletion
- **ldm-models repo cleanup** (Ken, LP-594 Pending) — branch cleanup, final PR sharing in progress

**In Progress**

- **AURA demo data DLP** (Parker, LP-617) — inactive users (1+ month) identified as deletion targets; ~470 active users preserved; dry run complete
    - Eval pool impact: ~160 → ~140 profiles after deletion; no other system impact expected
    - Target: Parker returns 5/6; execution on that date

---

## Substream: rSRR Predictor *(post-MVP research track)*
*(Ken Lee)*

**Done**

- **v1 training + evaluation complete** (Ken, LP-607) — Qwen3-VL 8B + LoRA, raw rSRR regression; pairwise profile-ranking task shows performance not worse than existing profile-ranker; additional ablations (regression vs. percentile classification, SmartPhoto alignment) completed

**In Progress**

- **Trideep-facing external brief** (Ken, LP-616) — Trideep requested lesson-learned context; writing in progress; Ken also shared face close ratio data from `tinder_events_delta.media_yolo` as potential future AURA input

---

## Exited this period

- **Link Lee** — transferred to Tailor (Chemistry) project ~4/21
