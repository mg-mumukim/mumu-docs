**Sources:**
- Photo Review Status Update Apr 15–22 (Google Doc)
- Backend Contract Google Doc (Brandon Choi, last updated Mar 24)
- [Serving Options / Option 2 Kick-off Decision Brief](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U) (Brandon Choi, last updated 4/29)
- [Notes — Spec Kick-off Photo Review](https://docs.google.com/document/d/1VxtA1qjkyZshECqtrLOL7NDe5G8CJiV6Mltmm6HHurw) (Jaini Shah, 4/27 ~ 4/29)
- [WIP Photo Review & Feedback](https://gotinder.atlassian.net/wiki/spaces/SOUP/pages/1898905601) (Confluence, SOUP space)
- [SOUP effort estimate sheet](https://docs.google.com/spreadsheets/d/1Hfuz19NhpCp02qL5PH_8zq3hqm_zKkKtykf0y2A6X8w) (Ryan Burns)
- Tinder Core × Rev Profile Coaching Workshop (Notion, 4/27 ~ 4/29)
- Jira LP project (hyperconnect.atlassian.net): 30 issues updated 4/27 ~ 4/29
- Slack #mgai-aura: 4/16 ~ 4/30
- Slack #ml-photo-review: 4/16 ~ 4/30 (via Glean)
- Google Calendar (mumu.kim@match.com): 4/16 ~ 4/23
- Notion: MGAI PM-TL Sync 4/30, Eval 4 feedback & required update

---

**Changes from v7**
- VLM substream headline corrected: KB v2.0 handed off to Mainstream 1 (Biggie); "injection logic blocks Eval 6" removed — that is Mainstream 1's internal problem
- DLP reframed: not a hard blocker for Eval 6; risk factor only (pool 160→140 after execution); Parker on vacation, verbal agreement for 5/6
- rSRR Predictor: cross-stream dependency on Mainstream 2 removed; post-MVP research runs independently of MVP delivery
- Date format: M/D throughout

---

# AURA Squad — Photo Review Status Update (4/16 ~ 4/30)

**Lead:** Biggie Choi  |  **Advisor:** Mumu Kim

---

## Mainstream 1: Prompt & Eval Iteration

**PoC:** Biggie Choi (Prompt), Juniper Han (Eval)
**Contributors:** Parker Cho (Prompt Factory), Ken Lee (Eval support)

> Eval 5 skipped; Eval 6 target week of 5/5, blocked by KB injection logic resolution and prompt v5.0.0 restructure — both Biggie's in-progress work.

**Done — Eval 4**
- [x] **Eval 4 labeling & findings** (Juniper, week of 4/22)
  - improvement areas: repetition cluster logic, diversity detection, blank photo handling, UI bugs
- [x] **Prompt v4.6.1 + AURA Demo UI fixes** (Biggie + Parker)
  - replace targets in repetition cluster limited to non-representative photos
  - diversity lever simplified: face photo + straight full-body shot held → pass
  - blank/meaningless photos (≥2) treated as a repetition cluster; prompt drift and schema inconsistencies fixed
  - repetitive cluster UI updated to Figma spec; replacement icon scope corrected; targeted repetitive marker prioritized
  - when repetition ≥ medium, diversity level shown one step lower (prevents "looks diverse but actually repetitive" contradiction)

**Done — Prompt Factory**
- [x] **T5 KB injection fix** (Parker) — [T5 Concept Pipeline](https://www.notion.so/hpcnt/T5-Concept-Pipeline-25-Photo-Coachable-Features-VLM-Tabular-340ce00cd35481bfba3dc5a3a3885006)
  - `photo`-type KB atoms were tagged as `bio` metadata and silently excluded from prompt
  - dataset re-verified: 160 profiles; change limited to user prompt and input messages
- [x] **Eval infra improvements** (Parker)
  - caching bug fixed: run N was showing run N-1's cached results
  - test set workflow simplified; project-scoped dataset management added (JSON import, lineage via `source_dataset_id`)
  - Testset Compare screen built for A/B run comparison
- [x] **PF knowledge share** (Parker) — [presentation](https://www.notion.so/hpcnt/34ece00cd3548092bc9cc4514c1ae98b)
  - Prompt Factory + Aura Eval System walkthrough delivered to Chemistry team

**Done — Comms**
- [x] **Workshop** (Juniper, 4/27 ~ 4/29) — Tinder Core × Rev Profile Coaching Workshop, Palo Alto
  - broader than Photo Review MVP: coaching strategy, free vs. paid framework, user journey mapping

**Skipped**
- [ ] Eval 5 — deferred by Juniper (~4/29)
  - KB needs more stability; rushing would produce an inaccurate Flash-Lite performance assessment

**In Progress**
- [ ] **Eval 6 KB expansion** (Biggie)
  - KB v2.0 received from Claude Yoon, registered in Prompt Factory ([T5 Concept Pipeline](https://www.notion.so/hpcnt/T5-Concept-Pipeline-25-Photo-Coachable-Features-VLM-Tabular-340ce00cd35481bfba3dc5a3a3885006))
  - blocker: injecting full 25-concept KB adds 13+ atoms/profile → LLM load too high; KB usage logic under review
- [ ] **Prompt v5.0.0 restructure** (Biggie)
  - full rewrite for Eval 6: remove duplications/contradictions, redefine output schema
  - depends on KB logic decision; Biggie owns both concurrently while Juniper supports other work
- [ ] **DLP compliance** (Parker, verbal agreement for 5/6) — [DLP Overview doc](https://docs.google.com/document/d/1hBVeDfT2gm0marF5f0E1ukic1sJw7tRMOiV6Z5Fh6Ow)
  - Parker on vacation; dry-run complete; ~470 active users retained
  - risk: eval pool will shrink ~160 → ~140 after execution (not a blocker for Eval 6)

**To Do**
- [ ] Eval 6 run (target: week of 5/5)
  - blocked by: KB injection logic (WI-03) + prompt v5.0.0 (WI-04), both Biggie in-progress
- [ ] Workshop findings summary (Juniper, [LP-611])
- [ ] Copy/tone alignment for Eval 6 baseline (Juniper, [LP-610])
- [ ] Team communication norms doc (Biggie, follow-up from 4/21 sync)

**Discussion (internal)**
- MVP LLM API: Gemini Pro kept as a scope-adjustment fallback for test region — not a signal of Flash-Lite failure. Ben Shin raised Bedrock-via-OpenAI as an alternative; cost estimate in progress (Kevin).

---

## Mainstream 2: Serving Spec & Integration

**PoC:** Brandon Choi
**Tinder counterparts:** Trideep Rath (Staff SWE ML), Jaini Shah (PM), Evan Keys (Sr. SWE Backend, Canvas)
**SOUP (backend team):** Nathan Lamb (TL), Henry Au (SWE III); Ryan Burns (Sr. Eng Manager)

> Timeline: Decision Brief 4/29 → Mock server + ML server design doc 5/6 → ML Deployment & testing 5/11 week → SOUP Q2 capacity late May → **BE Release QA 6/1**

**Done**
- [x] **Serving option decision** (Brandon) — [Option 2 Kick-off Brief](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U)
  - Option 2 (On-demand Sync) selected over Option 1 (Batch Precompute)
  - latency: p50 17.5s, p95 25.0s (2,486 samples, 4/20 ~ 4/22); cost: ~$33K/mo vs. ~$66K/mo upper bound, 3.65M DAU, 7% trigger rate
  - Flash-Lite not confirmed for production; Gemini Pro kept as fallback; eng region 100% rollout deferred until post-optimization
- [x] **Option 2 Kick-off Decision Brief + SOUP review** (Brandon) — [doc](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U)
  - Track A (product decisions) + Track B (engineering decisions) documented
  - Track B review sent to SOUP (Ryan Burns, Nathan Lamb, Henry Au) and Evan Keys (Canvas)
  - Evan Keys: no blockers; direction and document assessed as clear
- [x] **Backend Contract** (Brandon) — `POST /v1/photo-review`, 5-element response (bottom_sheet_title, bottom_sheet_body, coach_title, coach_body, repetitive_clusters)
- [x] **PRD review Part I** (Juniper) — [Product Spec](https://gotinder.atlassian.net/wiki/spaces/SOUP/pages/1898905601), [Spec Kick-off notes](https://docs.google.com/document/d/1VxtA1qjkyZshECqtrLOL7NDe5G8CJiV6Mltmm6HHurw)
  - team-wide PRD review 4/28; engineering-side blockers resolved
- [x] **Engineering uncertainties cleared** (Biggie → Trideep, 4/16)
  - Gemini Batch lag: 1–6 hr for English regions; Qwen3-VL: quality proof required; cache benefit: negligible (~3% of context)

**Decisions (resolved this period)**
> Custom ML model descoped — AURA calls Gemini API directly

> Serving: Option 2 (On-demand Sync)

> Coaching trigger: photo add / delete / replace only; bio, interests, reorder excluded

> Cadence: new coaching next app open after photo change; unchanged photos keep existing

> Bottom sheet timing: next app open, not same session as generation (Evan Keys + Jaini, 4/24 ~ 4/25)

> Repetitive cluster model: Best-keep + One-cluster-at-a-time; highest-rSRR photo left unmarked (4/28, Jaini + Juniper + Trideep)

> Language scope: English-only for MVP (Evan Keys + Juniper, 4/28)

> AURA LLM skip signal: status = success / skip (Brandon, 4/28)

**In Progress**
- [ ] **SOUP effort estimate** (Ryan Burns + Henry Au) — [estimate sheet](https://docs.google.com/spreadsheets/d/1Hfuz19NhpCp02qL5PH_8zq3hqm_zKkKtykf0y2A6X8w)
  - working session 4/30; SOUP Q2 capacity confirms late May start
  - MG AI development schedule not yet planned — serving option was not finalized until this week
- [ ] **PRD review Part II** (Juniper) — [Product Spec](https://gotinder.atlassian.net/wiki/spaces/SOUP/pages/1898905601)
  - Track A final text pending; bottom sheet direction and Nancy confirmation outstanding
  - Juniper ran Part 2 kick-off from travel on 4/29; content may not be fully current

**To Do**
- [ ] **Mock server + ML server design doc** (Brandon, target 5/6) — [MVP Eng Doc](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U)
  - external doc: SOUP system context + contract; internal doc: ML server component design (RFC for MLSE review)
  - events logging (flagged by Trideep) and User ID (flagged by Mumu) to be included
  - blocked by: contract alignment completion this week
- [ ] **ML Deployment & integration testing** (Brandon, target 5/11 week)
  - ML-side testing can begin 5/11; full e2e with SOUP backend only after SOUP Q2 capacity (late May)
- [ ] Privacy by Design doc for AURA demo (Biggie)
- [ ] Prompt Factory ↔ AURA ActionPlan schema contract (Parker, [LP-574])
- [ ] QoS investigation, VertexAI real-time and batch paths (Brandon, [LP-621])
- [ ] AURA Demo × Prompt Factory cleanup; Fake User Profile Generation (Parker)

**Open decisions (external input needed)**
- [ ] Bottom sheet dismiss policy: per-insight vs. full opt-out — Jaini deferred 4/28; engineering preference is full opt-out for MVP
- [ ] Photo tie-breaker metric for repetitive cluster ranking — Tinder team input pending; SmartPhoto score and top impressions/likes both lack full user coverage
- [ ] Profile Restrictions overlap exclusion rule — SOUPD-129 not finalized
- [ ] Markdown rendering effort (client side) — flagged 4/28, follow-up pending

**Discussion (internal)**
- Optimization stream (Trideep): post-MVP, a parallel optimization track should prepare — not all features need to be LLM-generated (e.g., a simple classification step before LLM call)

---

## Substream: VLM / Concepts KB

**PoC:** Claude Yoon

> KB v2.0 handed off to Mainstream 1 (Biggie); description verification ongoing.

**Done**
- [x] **Lifestyle Concepts KB v2.0** — handed off to Biggie; [T5 Concept Pipeline](https://www.notion.so/hpcnt/T5-Concept-Pipeline-25-Photo-Coachable-Features-VLM-Tabular-340ce00cd35481bfba3dc5a3a3885006)
  - 25 photo-coachable concepts validated on 530K rSRR Predictor v1 dataset (425K M / 104K F)
  - Cohen's d + Bonferroni + gender × age stratification on 28,687 NYC profiles
- [x] **`ldm-models` organized** — PR #38 (training + causal inference pipeline), PR #43 (modularized inference, stacked on #38)
- [x] **Demo-to-ML logic conversion** — 28 coaching items with explicit and implicit logic specified
- [x] **Concept labeling dataset** — 4/16 snapshot, ~5,000 training points

**In Progress**
- [ ] LLM-based concept description verification — 10-persona multi-agent approach generating and cross-validating descriptions

---

## Substream: rSRR Predictor

**PoC:** Ken Lee; external delivery: Biggie Choi

> Post-MVP research track — not in MVP scope. Results delivered to Trideep on request; runs independently of MVP delivery.

**Done**
- [x] **Predictor V1 experiment round 1** (Ken) — notified Trideep 4/27, delivered 4/29
  - Qwen3-VL 8B + LoRA, raw rSRR regression; R², MAE, Pearson, Kendall/Spearman, profile-ranker comparison, SmartPhoto counterfactual alignment evaluated
  - Trideep requested comparison of lesson-learnt against SmartPhoto model
- [x] **Training dataset v5 cleanup** (Ken) — S3 deletion confirmed; no AURA Demo impact verified with Parker and Brandon before deletion
- [x] **Face close ratio investigation** (Ken) — YOLO face detection table (`tinder_events_delta.media_yolo`) identified as a photo-level data source and shared

**In Progress / Pending**
- [ ] External doc for Trideep — in progress
- [ ] `ldm-models` repo and branch cleanup ([LP-594]) — pending final PR sharing

---

## Exited this period

**Link Lee** (→ Tailor/Chemistry ~4/21): completed analysis, submitted Offloading experiment PR #69 to Prompt Factory with lessons-learned doc, confirmed rSRR Predictor v1 data path before transfer.
