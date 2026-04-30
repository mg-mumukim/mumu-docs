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
- Notion: MGAI PM-TL Sync Apr 30, Eval 4 feedback & required update, Eval 5 Prompt 해부 세션

---

**Changes from v4**
- Factual corrections from Apr 30 PM-TL Sync: Eval 5 skipped (not in progress); next eval is Eval 6; prompt version is v4.6.1 with specific changes; KB injection problem documented; timeline added to Mainstream 2
- rSRR Predictor: "not in MVP scope" added
- DLP specifics added: 470 active users retained, 160→140 eval profile pool
- MVP LLM API discussion and optimization stream added
- Format: each section opens with `>` headline; key Done items include one-line rationale; Discussion split into internal vs. external
- Absence context added

---

# AURA Squad — Photo Review Status Update (Apr 16 – Apr 30, 2026)

참고: 이 기간 팀원 부재 — Claude 6일, Brandon 2일, Parker 1일, Biggie 1일, Juniper 출장 (Apr 27–29 Workshop)

---

## Mainstream 1: Prompt & Eval Iteration

**Squad lead:** Biggie Choi (Prompt), Juniper Han (Eval)
**Contributors:** Parker Cho (Prompt Factory), Ken Lee (Eval support)

> Eval 5 skipped this week — KB needs more time. Closing Eval 4 with prompt v4.6.1 deployed; Eval 6 planned next week, pending KB injection logic and prompt v5.0.0 restructure.

**Done**
- [x] **Eval 4 labeling & findings** (Juniper): improvement areas identified — repetition cluster logic, diversity detection, blank photo handling, UI bugs
- [x] **Prompt v4.6.1 deployed** (Biggie + Parker): Eval 4 feedback reflected
  - replace targets in repetition cluster now limited to non-representative photos
  - diversity lever simplified: face photo + straight full-body shot → pass
  - blank/meaningless photos (2+) now handled as a repetition cluster
  - prompt drift and output schema inconsistencies corrected
- [x] **Prompt Factory eval infra** (Parker): coaching snapshot caching bug fixed (run N was showing run N-1's cached results); test set add workflow simplified and documented; human eval page (`/promptfactory/aura-evals`) supports run-by-run result review and evaluator assignment
- [x] **Prompt Factory knowledge share** (Parker): Prompt Factory + Aura Eval System presentation delivered to Chemistry team
- [x] **Workshop** (Juniper): Tinder Core × Rev Profile Coaching Workshop attended (Palo Alto, Apr 27–29); coaching strategy, free vs. paid framework, user journey mapping

**Skipped**
- [ ] ~~Eval 5~~ — deferred; rushing to run Eval 5 without a stable KB would produce an inaccurate assessment of Gemini Flash-Lite performance

**In Progress**
- [ ] **Eval 6 KB expansion** (Biggie): KB v2.0 statistical validation complete (Cohen's d, Bonferroni, gender × age stratification); registered in Prompt Factory — however, injecting the full 25-concept KB adds 13+ atoms per profile, creating excessive LLM load; KB usage logic under review
- [ ] **Prompt v5.0.0 restructure** (Biggie): full prompt rewrite for Eval 6 — removing duplications and contradictions, redefining output schema. Scope is large; Biggie to run with Brandon support during Juniper's absence
- [ ] **DLP compliance** (Parker): active users (~470) identified for retention; dry-run complete — expected impact: eval profile pool 160 → ~140; deletion planned after Parker returns 5/6

**To Do**
- [ ] Eval 6 run (target: next week)
- [ ] Workshop findings summary and team share-out (Juniper, LP-611)
- [ ] Copy/tone alignment for Eval 6 baseline (Juniper, LP-610)
- [ ] Team communication norms doc (follow-up from Apr 21 sync)

**Discussion (internal)**
- MVP LLM API: Gemini Pro kept as a fallback option for MVP — not because Flash-Lite is failing, but as a scope-adjustment lever if needed in test region; cost estimate in progress (Kevin). Ben Shin raised Bedrock-via-OpenAI API as an alternative.

---

## Mainstream 2: Serving Spec & Integration

**Squad lead:** Brandon Choi
**Tinder counterparts:** Trideep Rath (Staff SWE ML), Jaini Shah (PM), Evan Keys (engineer), Henry Au / Xincen Hao (SOUP backend)

> Timeline: Decision brief 4/29 → Mock server + ML server design doc 5/6 (Wed PT morning) → ML Deployment & testing 5/11 week → SOUP Q2 capacity late May → **BE Release QA 6/1**

**Done**
- [x] **Serving option decision** (Brandon): Option 2 (On-demand Sync) selected over Option 1 (Batch Precompute); latency benchmarked at p50 17.5s / p95 25.0s (2,486 samples, Apr 20–22); cost ~$33K/mo (Option 1) vs. ~$66K/mo upper bound (Option 2), 3.65M DAU, 7% trigger rate. Flash-Lite model is not confirmed for production; Pro is a fallback; eng region 100% rollout deferred until post-optimization.
- [x] **Option 2 Kick-off Decision Brief** (Brandon): Track A (product decisions) + Track B (engineering decisions) documented; SOUP engineers (Ryan Burns, Nathan Lamb, Henry Au, Evan Keys) sent Track B review request; Evan Keys reviewed — no blockers, direction and document assessed as clear
- [x] **Backend Contract** (Brandon): `POST /v1/photo-review`, 5-element response (bottom_sheet_title, bottom_sheet_body, coach_title, coach_body, repetitive_clusters)
- [x] **PRD review Part I** (Juniper): team-wide PRD review held Apr 28; engineering-side blockers resolved (LP-613)
- [x] **Prompt logical contradictions** (Brandon): identified and documented; Gemini Batch lag confirmed 1–6 hr for English; Qwen3-VL requires quality proof; cache benefit assessed as negligible (~3% of context)

**Decisions (resolved this period)**
> Custom ML model descoped from MVP — AURA calls Gemini API directly

> Serving: Option 2 (On-demand Sync)

> Coaching trigger: photo add / delete / replace only; bio, interests, reorder excluded

> Cadence: new coaching next app open after photo change; unchanged photos keep existing coaching

> Bottom sheet timing: next app open, not in same session as generation (Evan + Jaini, Apr 24–25)

> Repetitive cluster model: Best-keep + One-cluster-at-a-time; highest-rSRR photo left unmarked (Apr 28, Jaini + Juniper + Trideep)

> Language scope: English-only for MVP (Evan + Juniper, Apr 28)

> AURA LLM skip signal: status = success / skip (Brandon, Apr 28)

**In Progress / Pending**
- [ ] **SOUP effort estimate** (Ryan Burns, Henry Au, Evan Keys): estimate sheet started; Evan + Henry working session scheduled 4/30; MG AI side development/deployment schedule not yet planned (Mumu: "저번 주까지는 serving option이 정해지지 않았고 이번 주는 업무일이 별로 없어서 아직 제대로 planning하지 않았음")
- [ ] **PRD review Part II** (Juniper): Track A final text update pending; bottom sheet direction on shortest path, pending Nancy confirmation

**To Do**
- [ ] Mock server + ML server design doc — Brandon; target 5/6 Wed PT morning (conditional on contract alignment this week)
  - Two doc types: external (system context + contract for SOUP) + internal (ML server component design, RFC format for MLSE review)
  - Events logging (flagged by Trideep) and User ID (flagged by Mumu) to be included
- [ ] Privacy by Design doc for AURA demo
- [ ] Prompt Factory ↔ AURA ActionPlan schema contract (LP-574)
- [ ] QoS investigation for VertexAI real-time and batch paths (LP-621)
- [ ] AURA Demo × Prompt Factory cleanup; Fake User Profile Generation

**Open decisions (external input needed)**
- [ ] Bottom sheet dismiss policy: per-insight dismiss vs. full opt-out — Jaini deferred at Apr 28 Spec Kick-off; engineering preference is full opt-out for MVP
- [ ] Photo tie-breaker metric for repetitive cluster ranking — Tinder team input pending
- [ ] Profile Restrictions overlap exclusion rule — SOUPD-129 not finalized
- [ ] Markdown rendering effort (client side) — flagged at Apr 28, follow-up pending

**Discussion (internal)**
- Optimization stream (Trideep): post-MVP, a parallel optimization track should be prepared — not all features need to be LLM-generated (e.g., a simple classification step before LLM call)

---

## Substream: VLM / Concepts KB

**Lead:** Claude Yoon

> KB v2.0 statistical validation complete and registered in Prompt Factory; injection logic under review before Eval 6 integration.

**Done**
- [x] Lifestyle Concepts KB v2.0: 25 photo-coachable concepts validated on 530K rSRR Predictor v1 dataset (425K M / 104K F); Cohen's d + Bonferroni + gender × age stratification on 28,687 NYC profiles
- [x] `ldm-models` organized: PR #38 (training + causal inference pipeline), PR #43 (modularized inference, stacked on #38)
- [x] Demo-to-ML logic conversion: 28 coaching items specified with explicit and implicit logic
- [x] Concept labeling dataset (Apr 16 snapshot, ~5,000 training points)

**In Progress**
- [ ] LLM-based concept description verification: 10-persona multi-agent approach generating and cross-validating descriptions

---

## Substream: rSRR Predictor

**Lead:** Ken Lee; external delivery: Biggie Choi

> Not in MVP scope — ongoing research to establish post-MVP goals; results shared with Trideep as requested.

**Done**
- [x] Predictor V1 experiment round 1: Qwen3-VL 8B + LoRA, raw rSRR regression; R², MAE, Pearson, Kendall/Spearman, profile-ranker comparison, and SmartPhoto counterfactual alignment evaluated; results delivered to Trideep (notified Apr 27, delivered Apr 29)
- [x] Training dataset v5 cleaned up (S3 deletion confirmed; no impact on AURA Demo verified with Parker and Brandon)
- [x] Face close ratio investigation: YOLO face detection table (`tinder_events_delta.media_yolo`) identified as a photo-level data source and shared

**In Progress / Pending**
- [ ] External documentation for Trideep — in progress (Trideep requested lesson-learnt vs SmartPhoto model)
- [ ] `ldm-models` repo and branch cleanup — pending final PR sharing (LP-594)

---

## Exited this period

**Link Lee** (transferred to Tailor/Chemistry ~Apr 21): completed assigned analysis, submitted Offloading experiment PR #69 to Prompt Factory with a lessons-learned doc, and confirmed rSRR Predictor v1 data path before transfer.
