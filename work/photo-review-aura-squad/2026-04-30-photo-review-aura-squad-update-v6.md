**Sources:**
- Photo Review Status Update Apr 15–22 (Google Doc)
- Backend Contract Google Doc (Brandon Choi, last updated Mar 24)
- Serving Options Google Doc (Juniper Han, Brandon Choi, last updated Apr 15)
- Serving Options thread (Slack #ml-photo-review, Apr 16)
- Tinder Core × Rev Profile Coaching Workshop (Notion, Apr 27–29)
- Option 2 Kick-off Decision Brief Google Doc (Brandon Choi, last updated Apr 29)
- Jira LP project (hyperconnect.atlassian.net): 30 issues updated Apr 27–29
- Slack #mgai-aura: Apr 16–30
- Slack #ml-photo-review: Apr 16–30 (via Glean)
- Google Calendar (mumu.kim@match.com): Apr 16–23
- Notion: MGAI PM-TL Sync Apr 30, Eval 4 feedback & required update

---

**Changes from v5**
- Lead labeling corrected: document-level Lead/TL header added; per-section "Squad lead" replaced with "PoC"
- Mainstream 1 Done: three Parker items added from Notion AURA section — T5 KB injection fix, diversity display policy adjustment, repetitive cluster UI improvement (Owen Dump)
- Mainstream 1: Done items regrouped by phase (Eval 4 → Prompt v4.6.1 → PF tooling → comms); per-item owner attribution added
- Mainstream 2: SOUP estimate and PRD Part II separated into distinct In Progress items with context
- Sentence quality: tightened rationale clauses; reduced interpretive framing

---

# AURA Squad — Photo Review Status Update (Apr 16 – Apr 30, 2026)

**Lead:** Biggie Choi  |  **TL:** Mumu Kim

---

## Mainstream 1: Prompt & Eval Iteration

**PoC:** Biggie Choi (Prompt), Juniper Han (Eval)
**Contributors:** Parker Cho (Prompt Factory), Ken Lee (Eval support)

> Eval 5 skipped — KB needs more time; rushing would produce an inaccurate Flash-Lite assessment. Eval 4 closed with prompt v4.6.1 shipped; Eval 6 in prep, blocked on KB injection logic and prompt v5.0.0 restructure.

**Done — Eval 4 phase**
- [x] **Eval 4 labeling & findings** (Juniper): improvement areas identified — repetition cluster logic, diversity detection, blank photo handling, UI bugs
- [x] **Prompt v4.6.1 shipped** (Biggie + Parker): Eval 4 feedback reflected in prompt and UI
  - replace targets in repetition cluster now limited to non-representative photos
  - diversity lever simplified: face photo + straight full-body shot held → pass
  - blank/meaningless photos (≥2) now treated as a repetition cluster
  - prompt drift and output schema inconsistencies fixed
  - repetitive cluster UI updated to Figma spec; replacement icon scope corrected; targeted repetitive marker shown first
  - when repetition ≥ medium, diversity level displayed one step lower (avoids "looks diverse but has repetitive photos" contradiction)

**Done — Prompt Factory**
- [x] **T5 KB injection fix** (Parker): `photo`-type KB atoms were tagged as `bio` metadata and silently excluded from the prompt; fixed so T5 photo-coachable concepts now reach Profile Coach; dataset re-verified (160 profiles, change limited to user prompt and input messages)
- [x] **Eval infra** (Parker): coaching snapshot caching bug fixed (run N showed run N-1's cache); test set workflow simplified and documented; human eval page (`/promptfactory/aura-evals`) supports per-run result review and evaluator assignment; project-scoped dataset management added (JSON import, dataset lineage via `source_dataset_id`); Testset Compare screen built for A/B run comparison
- [x] **Prompt Factory knowledge share** (Parker): Prompt Factory + Aura Eval System presentation delivered to Chemistry team

**Done — Comms & alignment**
- [x] **Workshop** (Juniper): Tinder Core × Rev Profile Coaching Workshop (Palo Alto, Apr 27–29); coaching strategy, free vs. paid framework, user journey mapping — broader than Photo Review MVP scope

**Skipped**
- [ ] ~~Eval 5~~ — deferred by Juniper (~Apr 29); rushing without a stable KB would produce an inaccurate Flash-Lite assessment

**In Progress**
- [ ] **Eval 6 KB expansion** (Biggie): KB v2.0 statistical validation complete and registered in Prompt Factory — but injecting the full 25-concept KB adds 13+ atoms per profile, creating excessive LLM load; KB usage logic under review
- [ ] **Prompt v5.0.0 restructure** (Biggie): full rewrite for Eval 6 — removing duplications and contradictions, redefining output schema; Biggie to drive with Brandon support during Juniper's absence
- [ ] **DLP compliance** (Parker): ~470 active users identified for retention; dry-run complete — eval profile pool expected to decrease from ~160 to ~140; execution after Parker returns 5/6

**To Do**
- [ ] Eval 6 run (target: next week)
- [ ] Workshop findings summary and team share-out (Juniper, LP-611)
- [ ] Copy/tone alignment for Eval 6 baseline (Juniper, LP-610)
- [ ] Team communication norms doc (Biggie, follow-up from Apr 21 sync)

**Discussion (internal)**
- MVP LLM API: Gemini Pro kept as a scope-adjustment fallback for test region — not a signal of Flash-Lite failure. Ben Shin raised Bedrock-via-OpenAI as an alternative; cost estimate in progress (Kevin).

---

## Mainstream 2: Serving Spec & Integration

**PoC:** Brandon Choi
**Tinder counterparts:** Trideep Rath (Staff SWE ML), Jaini Shah (PM), Evan Keys (engineer)
**SOUP:** Henry Au, Xincen Hao

> Timeline: Decision brief 4/29 → Mock server + ML server design doc 5/6 (Wed PT morning) → ML Deployment & testing 5/11 week → SOUP Q2 capacity late May → **BE Release QA 6/1**

**Done**
- [x] **Serving option decision** (Brandon): Option 2 (On-demand Sync) selected over Option 1 (Batch Precompute); Flash-Lite sync latency measured at p50 17.5s / p95 25.0s (2,486 samples, Apr 20–22); cost ~$33K/mo (Option 1) vs. ~$66K/mo upper bound (Option 2), 3.65M DAU, 7% trigger rate. Flash-Lite not yet confirmed for production; Gemini Pro remains a fallback; eng region 100% rollout deferred until post-optimization.
- [x] **Option 2 Kick-off Decision Brief** (Brandon): Track A (product decisions) + Track B (engineering decisions) documented; SOUP engineers (Ryan Burns, Nathan Lamb, Henry Au, Evan Keys) sent Track B review; Evan Keys reviewed — no blockers, direction and document assessed as clear
- [x] **Backend Contract** (Brandon): `POST /v1/photo-review`, 5-element response (bottom_sheet_title, bottom_sheet_body, coach_title, coach_body, repetitive_clusters)
- [x] **PRD review Part I** (Juniper): team-wide PRD review Apr 28; engineering-side blockers resolved (LP-613)
- [x] **Engineering uncertainties cleared** (Biggie → Trideep, Apr 16): Gemini Batch lag confirmed at 1–6 hr for English regions; Qwen3-VL requires quality proof before adding; cache benefit assessed as negligible (~3% of context)

**Decisions (resolved this period)**
> Custom ML model descoped from MVP — AURA calls Gemini API directly

> Serving: Option 2 (On-demand Sync)

> Coaching trigger: photo add / delete / replace only; bio, interests, reorder excluded from MVP

> Cadence: new coaching next app open after photo change; unchanged photos keep existing coaching

> Bottom sheet timing: next app open, not in same session as generation (Evan Keys + Jaini, Apr 24–25)

> Repetitive cluster model: Best-keep + One-cluster-at-a-time; highest-rSRR photo left unmarked (Apr 28, Jaini + Juniper + Trideep)

> Language scope: English-only for MVP (Evan Keys + Juniper, Apr 28)

> AURA LLM skip signal: status = success / skip (Brandon, Apr 28)

**In Progress**
- [ ] **SOUP effort estimate** (Ryan Burns + Henry Au): estimate sheet started; Evan Keys + Henry Au working session Apr 30; MG AI development/deployment schedule not yet planned — serving option was not finalized until this week
- [ ] **PRD review Part II** (Juniper): Track A final text pending; bottom sheet direction to be aligned on shortest path; Nancy confirmation outstanding. Note: Juniper ran the second kick-off in the middle of the night KST during travel — content may not reflect the latest.

**To Do**
- [ ] **Mock server + ML server design doc** (Brandon, target 5/6 PT morning): two doc types — external (system context + contract for SOUP) and internal (ML server component design, RFC format for other MLSEs). Events logging (flagged by Trideep) and User ID (flagged by Mumu) to be included.
- [ ] Deployment & integration testing — target 5/11 week; e2e tests after SOUP backend is ready
- [ ] Privacy by Design doc for AURA demo (Biggie)
- [ ] Prompt Factory ↔ AURA ActionPlan schema contract (LP-574)
- [ ] QoS investigation for VertexAI real-time and batch paths (LP-621)
- [ ] AURA Demo × Prompt Factory cleanup; Fake User Profile Generation

**Open decisions (external input needed)**
- [ ] Bottom sheet dismiss policy: per-insight dismiss vs. full opt-out — Jaini deferred at Apr 28 Spec Kick-off; engineering preference is full opt-out for MVP
- [ ] Photo tie-breaker metric for repetitive cluster ranking — Tinder team input pending; SmartPhoto score and top impressions/likes both lack full user coverage
- [ ] Profile Restrictions overlap exclusion rule — SOUPD-129 not finalized
- [ ] Markdown rendering effort (client side) — flagged at Apr 28, follow-up pending

**Discussion (internal)**
- Optimization stream (Trideep): post-MVP, a parallel track should prepare for optimization — not all features need to be LLM-generated (e.g., simple classification before LLM call)

---

## Substream: VLM / Concepts KB

**PoC:** Claude Yoon

> KB v2.0 statistical validation complete and registered in Prompt Factory; injection logic under review before Eval 6 integration.

**Done**
- [x] Lifestyle Concepts KB v2.0: 25 photo-coachable concepts validated on 530K rSRR Predictor v1 dataset (425K M / 104K F); Cohen's d + Bonferroni + gender × age stratification on 28,687 NYC profiles
- [x] `ldm-models` organized: PR #38 (training + causal inference pipeline), PR #43 (modularized inference, stacked on #38)
- [x] Demo-to-ML logic conversion: 28 coaching items with explicit and implicit logic specified
- [x] Concept labeling dataset (Apr 16 snapshot, ~5,000 training points)

**In Progress**
- [ ] LLM-based concept description verification: 10-persona multi-agent approach generating and cross-validating descriptions

---

## Substream: rSRR Predictor

**PoC:** Ken Lee; external delivery: Biggie Choi

> Not in MVP scope — research to establish post-MVP direction. Trideep asked for results to compare against SmartPhoto lesson-learnt.

**Done**
- [x] Predictor V1 experiment round 1: Qwen3-VL 8B + LoRA, raw rSRR regression; R², MAE, Pearson, Kendall/Spearman, profile-ranker comparison, and SmartPhoto counterfactual alignment evaluated; results delivered to Trideep (notified Apr 27, delivered Apr 29)
- [x] Training dataset v5 cleaned up — S3 deletion confirmed; no AURA Demo impact verified with Parker and Brandon before deletion
- [x] Face close ratio investigation: YOLO face detection table (`tinder_events_delta.media_yolo`) identified as a photo-level data source and shared

**In Progress / Pending**
- [ ] External doc for Trideep — in progress
- [ ] `ldm-models` repo and branch cleanup (LP-594) — pending final PR sharing

---

## Exited this period

**Link Lee** (→ Tailor/Chemistry ~Apr 21): completed assigned analysis, submitted Offloading experiment PR #69 to Prompt Factory with a lessons-learned doc, confirmed rSRR Predictor v1 data path before transfer.
