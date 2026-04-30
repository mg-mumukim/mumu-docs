**Sources:**
- Photo Review Status Update Apr 15–22 (Google Doc)
- Backend Contract Google Doc (Brandon Choi, last updated Mar 24)
- Serving Options Google Doc (Juniper Han, Brandon Choi, last updated Apr 15)
- Serving Options thread (Slack #ml-photo-review, Apr 16)
- Tinder Core × Rev Profile Coaching Workshop (Notion, Apr 27–29)
- Option 2 Kick-off Decision Brief Google Doc (Brandon Choi, last updated Apr 28)
- Jira LP project (hyperconnect.atlassian.net): 30 issues updated Apr 27–29
- Slack #mgai-aura: Apr 16–30
- Slack #ml-photo-review: Apr 16–30 (via Glean)
- Google Calendar (mumu.kim@match.com): Apr 16–23
- Notion: MGAI PM-TL Sync (Juniper) Apr 24, Eval 4 feedback & required update, Eval 5 Prompt 해부 세션

---

**Changes from v2**
- Restructured from 8 role-based sections into 2 core streams + 2 parallel streams; exited-member work consolidated at bottom
- Squad lead / Tinder counterpart distinction applied; Tinder/SOUP section removed (Trideep Rath is a counterpart, not a squad PoC)
- Decisions extracted into a dedicated subsection per stream; open items kept with stream owner
- Routine documentation and comms items grouped into single bullets

---

# AURA Squad — Photo Review Status Update (Apr 16 – Apr 30, 2026)

---

## Core Stream 1: Coaching Quality Loop

**Squad lead:** Biggie Choi (prompt), Juniper Han (eval)
**Contributors:** Parker Cho (Prompt Factory), Ken Lee (eval support)

This period closed Eval 3, completed Eval 4, and launched Eval 5 prep across three parallel tracks (KB review, prompt draft, DLP compliance). The prompt reached v4.6.x and is now undergoing Eval 5 review.

**Done**
- [x] Prompt updated to v4.6.x: one-cluster-at-a-time UI logic added (surface lowest-rSRR cluster first; representative photo excluded from replace targets), correcting a prior contradiction where replace/swap could target the representative photo
- [x] Eval 3 finalized and delivered to Jaini Shah: Diversity restructured (Interests & Lifestyle merged; Activities kept separate); repetitiveness definition refined to capture framing repetition in close-up selfies
- [x] Eval 4 run (week of Apr 22, Gemini Flash-Lite vs. Pro): accuracy issues documented in both models; go/no-go criteria for Flash-Lite defined for Eval 5 decision; feedback doc published
- [x] Documentation and alignment updates: Apr 15–22 status update published, Confluence PRD updated, Prompt Factory test set workflow simplified, Chemistry team PF knowledge share delivered
- [x] Tinder Core × Rev Profile Coaching Workshop attended (Palo Alto, Apr 27–29): coaching strategy, free vs. paid framework, user journey mapping

**In Progress**
- [ ] Eval 5 prep — three tracks running in parallel:
  - Prompt review: Biggie + Juniper + Parker iterating on draft
  - KB: Biggie reviewing Lifestyle KB v2.0 statistical validation (joint with Claude Yoon)
  - DLP: Parker resolving AURA demo data compliance requirement before Eval 5 can run

**To Do**
- [ ] Eval 5 execution and results (target: week of Apr 28)
- [ ] Workshop findings summary and team share-out (Juniper)
- [ ] Copy/tone alignment for Eval 6 baseline (Juniper)
- [ ] Team communication norms doc (follow-up from Apr 21 sync)

---

## Core Stream 2: Engineering Spec & Integration

**Squad lead:** Brandon Choi
**Tinder counterparts:** Trideep Rath (Staff SWE ML), Jaini Shah (PM), Evan (engineer), Henry Au / Xincen Hao (SOUP backend)

This period moved from option analysis to a finalized backend contract: serving direction selected, eight product and engineering decisions resolved, and a preliminary Spec Kick-off held with Tinder on Apr 28. Formal integration work begins with the SOUP backend next.

**Done**
- [x] Serving options analysis completed: Option 2 (On-demand Sync) selected over Option 1 (Batch Precompute); Flash-Lite sync latency benchmarked at p50 17.5s / p95 25.0s (2,486 samples, Apr 20–22); cost estimated at ~$33K/mo (Option 1) vs. ~$66K/mo upper bound (Option 2) for 3.65M DAU at 7% trigger rate
- [x] Backend contract finalized: `POST /v1/photo-review` with 5-element response (bottom_sheet_title, bottom_sheet_body, coach_title, coach_body, repetitive_clusters)
- [x] Prompt logical contradictions identified and documented; three open engineering questions clarified with Tinder (Gemini Batch lag confirmed at 1–6 hr for English; Qwen3-VL requires quality proof before adding; cache benefit assessed as negligible at ~3% of context)
- [x] Option 2 Kick-off Decision Brief drafted; Spec Kick-off session held Apr 28 with Tinder — product and engineering decisions recorded
- [x] Trideep shared Photo Suppression V2 proposal in #ml-photo-review on Apr 20, tagging AURA team for awareness

**Decisions (resolved this period)**
> Custom ML model descoped from MVP — AURA calls Gemini API directly

> Serving: Option 2 (On-demand Sync)

> Coaching trigger: photo add / delete / replace only; bio, interests, and reorder changes excluded from MVP

> Cadence: new coaching on next app open after photo change; unchanged photos keep existing coaching

> Bottom sheet timing: next app open, not in same session as generation (Evan + Jaini, Apr 24–25)

> Repetitive cluster model: Best-keep + One-cluster-at-a-time; highest-rSRR photo in cluster left unmarked (confirmed Apr 28 with Jaini, Juniper, Trideep)

> Language scope: English-only for MVP (Evan + Juniper, Apr 28)

> AURA LLM skip signal: status = success / skip (Brandon, Apr 28)

**In Progress / Pending**
- [ ] Formal Option 2 kick-off meeting with SOUP backend — preliminary alignment reached at Apr 28 Spec Kick-off; formal meeting not yet scheduled
- [ ] Dummy ML server for backend contract validation

**To Do**
- [ ] QoS investigation for VertexAI real-time and batch paths
- [ ] Privacy by Design doc for AURA demo
- [ ] PF ↔ AURA ActionPlan schema contract (prevents silent prompt-version drift)
- [ ] Aura Demo × Prompt Factory cleanup; Fake User Profile Generation

**Open decisions (external input needed)**
- [ ] Bottom sheet dismiss policy: per-insight dismiss vs. full opt-out — Jaini explicitly deferred at Apr 28 Spec Kick-off; engineering preference is full opt-out for MVP, awaiting PM ratification
- [ ] Photo tie-breaker metric for repetitive cluster ranking — Tinder team input pending; current candidates (SmartPhoto score, top impressions/likes) lack full user coverage
- [ ] Profile Restrictions overlap exclusion rule — pending SOUP ticket finalization
- [ ] Markdown rendering effort for client side — flagged at Apr 28; follow-up pending from Trideep/Evan

---

## Parallel Stream: VLM Foundation

**Lead:** Claude Yoon

**Done**
- [x] Lifestyle Concepts KB upgraded to v2.0: 25 photo-coachable concepts validated on 530K rSRR Predictor v1 dataset (425K M / 104K F); analysis includes Cohen's d, Bonferroni correction, and gender × age stratification on 28,687 NYC profiles
- [x] `ldm-models` training and inference code organized: PR #38 (training + causal inference pipeline), PR #43 (modularized inference, stacked on #38)
- [x] Demo-to-ML logic conversion completed: 28 coaching items specified with explicit and implicit logic
- [x] Concept labeling dataset produced (Apr 16 snapshot, ~5,000 training points)

**In Progress**
- [ ] LLM-based concept description verification: 10-persona multi-agent approach generating and cross-validating concept descriptions

---

## Parallel Stream: rSRR Predictor

**Lead:** Ken Lee; external delivery: Biggie Choi

**Done**
- [x] Predictor V1 experiment round 1 completed; results delivered to Trideep Rath (notified Apr 27, delivered Apr 29)
- [x] Training dataset v5 cleaned up; face close ratio data investigated — YOLO face detection table (`tinder_events_delta.media_yolo`) identified as a photo-level data source and shared with team

**In Progress / Pending**
- [ ] External documentation for Trideep Rath — in progress
- [ ] Repo and branch cleanup — pending final PR sharing

---

## Exited this period

**Link Lee** (transferred to Tailor/Chemistry ~Apr 21): completed assigned analysis work, submitted Offloading experiment PR #69 to prompt-factory with a lessons-learned doc, and confirmed rSRR-predictor v1 data path for the team before transfer.
