**Sources:**
- Photo Review Status Update Apr 15–22 (Google Doc)
- Backend Contract Google Doc (Brandon Choi, last updated Mar 24)
- Serving Options Google Doc (Juniper Han, Brandon Choi, last updated Apr 15)
- Serving Options thread (Slack #ml-photo-review, Apr 16)
- Tinder Core × Rev Profile Coaching Workshop (Notion, Apr 27–29)
- Option 2 Kick-off Decision Brief Google Doc (Brandon Choi, last updated Apr 28)
- Jira LP project (hyperconnect.atlassian.net): 30 issues updated Apr 27–29
- Slack #mgai-aura: Apr 16–30
- Google Calendar (mumu.kim@match.com): Apr 16–23
- Notion: MGAI PM-TL Sync (Juniper) Apr 24, Eval 4 feedback & required update, Eval 5 Prompt 해부 세션

---

# AURA Squad — Photo Review Status Update (Apr 16 – Apr 30, 2026)

## 1. Project Management
PoC: Biggie Choi

**Done**
- Shared rSRR predictor results with Trideep Rath
- Updated prompt to v4.6.x: added one-cluster-at-a-time UI logic for repetitive photos (surface lowest-RSRR cluster first; excluded representative photo from replace targets); corrected internal contradiction where replace/swap action could previously target representative photos

**In Progress**
- Lifestyle 25-concept KB report v2.0: reviewing statistical validation results and determining coaching priority (joint work with Claude Yoon)
- Eval 5 prompt review in progress with Juniper Han and Parker Cho

**To Do**
- Team communication norms document (follow-up from Apr 21 team sync)
- Privacy by Design document for AURA demo on profile coaching
- AURA Hub MVP exploration

---

## 2. Product / PM
PoC: Juniper Han; Tinder PM counterpart: Jaini Shah

**Done**
- Eval 3 results finalized and shared with Jaini Shah
  - Diversity category restructured: Interests & Lifestyle merged into one; Activities kept as a separate category
  - Repetitiveness definition refined to capture framing repetition in close-up selfies
- Apr 15–22 status update doc published for Tinder alignment
- Eval 4 conducted (week of Apr 22): Gemini Flash-Lite vs Pro comparison
  - Repetitive detection accuracy issues observed in both models: some repetitive photos missed; some non-repetitive photos incorrectly flagged
- Eval 4 feedback & required update Notion doc published; go/no-go criteria for Flash-Lite defined for Eval 5 decision
- Confluence PRD updated
- Pending product decisions resolved to unblock engineering sync (serving direction, coaching trigger rules, skip rules, bottom sheet timing)
- Attended Tinder Core × Rev Profile Coaching Workshop in Palo Alto (Apr 27–29)
  - Workshop covered coaching strategy, free vs. paid framework, user journey mapping; broader than Photo Review MVP scope

**In Progress**
- Eval 5 preparation: coordinating Knowledge Base update, prompt update, and bug fixes with Brandon and Parker (three tracks running in parallel)

**To Do**
- Workshop findings summary and team share-out
- Photo Review copy/tone alignment (polished baseline for Eval 6)
- Feature product rationale build-out

---

## 3. AURA Demo
PoC: Brandon Choi; Advisor: Mumu Kim

**Done**
- Serving options doc completed: Option 2 (On-demand Sync) recommended over Option 1 (Batch Precompute)
  - Measured Gemini Flash-Lite sync latency: p50 = 17.5s, p95 = 25.0s (2,486 samples, Apr 20–22 from Prompt Factory)
  - Gemini Batch dry-run turnaround: 16.8 min median (25 requests; not guaranteed at scale)
  - Cost estimate: ~$33K/mo Option 1 vs. ~$66K/mo upper bound Option 2, test region (USA/CAN/AUS, 3.65M DAU, 7% trigger rate)
- Backend contract defined: POST /v1/photo-review; 5-element response: bottom_sheet_title, bottom_sheet_body, coach_title, coach_body, repetitive_clusters
- Prompt logical contradictions identified and documented (e.g., "ALWAYS cite" vs. "cite only when backed" instructions)
- Option 2 Kick-off Decision Brief drafted; preliminary Spec Kick-off session held Apr 28 resolving required product and engineering decisions

**In Progress / Pending**
- Formal Option 2 kick-off meeting with SOUP backend (LP-606 Pending)
- Dummy ML server deployment for backend contract testing

**To Do**
- QoS investigation for VertexAI real-time and batch paths

**Key decisions resolved this period**
- Custom ML model descoped from MVP; AURA calls Gemini API directly
- Serving direction confirmed: Option 2 (On-demand Sync)
- Coaching trigger: app open only; photo add/delete/replace are the only refresh signals (bio/interest/reorder excluded from MVP)
- Coaching cadence: new coaching on next app open after photo change; existing coaching kept when photos unchanged
- Bottom sheet timing: appears at next app open, not in same session as generation (Evan + Jaini, Apr 24–25)
- Repetitive cluster model: Best-keep + One-cluster-at-a-time; highest-RSR photo in cluster left unmarked (confirmed at Apr 28 Spec Kick-off with Jaini, Juniper, Trideep)
- MVP language scope: English-only (Evan + Juniper, Apr 28)

---

## 4. Prompt Factory
PoC: Parker Cho

**Done**
- Prompt Factory test set workflow simplified and documented
- Cross-team knowledge sharing on Prompt Factory conducted for the Chemistry team
- Eval 4 support: provided evaluation session URLs; directed evaluators to Prompt Factory Human Eval page for Diversity Poses fail case review

**In Progress**
- Eval 5 iteration prep
- DLP compliance for AURA demo data

**To Do**
- Prompt Factory ↔ AURA ActionPlan schema contract (prevents silent prompt-version drift)
- Aura Demo × Prompt Factory cleanup
- Fake User Profile Generation using image generation model

---

## 5. Model / VLM
PoC: Claude Yoon

**Done**
- T5 Photo-Coachable Concepts KB report upgraded to v2.0: statistical validation added using 530K rSRR Predictor v1 dataset (M 425K / F 104K); covers 25 photo-coachable concepts; includes Cohen's d, Bonferroni correction, and gender × age stratified analysis on NYC users (28,687 profiles)
- Training and inference code organized in ldm-models: PR #38 (training + causal inference pipeline) and PR #43 (modularized inference, stacked on top of #38)
- Demo logic to ML logic conversion document completed: 28 coaching items with explicit and implicit logic specified
- Concept labeling dataset produced (Apr 16 snapshot, ~5,000 training points)

**In Progress**
- LLM-based concept description verification: 10-persona multi-agent approach generating and cross-validating concept descriptions

---

## 6. Evaluation
PoC: Ken Lee

**Done**
- rSRR Predictor V1 additional experiment round 1 completed
- Training dataset v5 cleaned up
- Face close ratio investigation: identified YOLO face detection table (tinder_events_delta.media_yolo) as a photo-level data source; shared with team for potential use in AURA demo pipeline

**In Progress**
- rSRR predictor documentation for external stakeholders (Trideep Rath)

**Pending**
- Repository and branch cleanup; final PR sharing

---

## 7. Analysis
PoC: Link Lee (transferred to Tailor / Chemistry mid-period, ~Apr 21)

**Done (before transfer)**
- Completed assigned analysis work
- Submitted Offloading experiment to Prompt Factory (PR #69 in prompt-factory); included lesson-learned document for team reference
- Provided rSRR-predictor v1 data path confirmation to team

Link Lee's AURA contributions ended upon transfer. No AURA work assigned after approximately Apr 21.

---

## 8. Tinder / SOUP Alignment
PoC: Trideep Rath (Staff SWE ML, Tinder); PM counterpart: Jaini Shah

**Done**
- Apr 16 Slack thread (Mumu → Trideep, Ben Shin): reviewed three open engineering uncertainties
  - Gemini Batch lag: 1–6 hours for English regions confirmed (based on prior 10K-request backfill benchmark)
  - Qwen3-VL step: team asked to prove quality uplift before adding as an intermediate step
  - Cache cost reduction: challenged — system prompt is only ~3% of context; cache benefit assessed as negligible
- Reviewed backend contract (co-reviewer alongside Mumu)
- Apr 24–25: agreed with Evan (Tinder) that bottom sheet appears at next app open, not same session
- Apr 28 Spec Kick-off: confirmed repetitive cluster model with Jaini and Juniper
- Received rSRR predictor report from Biggie
- Apr 27 workshop: attended Tinder Core × Rev Profile Coaching Workshop in Palo Alto

**In Progress / Pending**
- Formal Option 2 kick-off with SOUP backend engineers (Henry Au, Xincen Hao)
- Profile Restrictions experiment overlap exclusion rule (pending finalization via SOUP ticket)

**Open items requiring Tinder input**
- Bottom sheet dismiss policy: per-insight dismiss vs. full opt-out (explicitly logged as open question by Jaini at Apr 28 Spec Kick-off; engineering suggestion is full opt-out for MVP, pending PM ratification)
- Universal photo tie-breaker metric for repetitive cluster ranking: requested from Tinder team (current options — SmartPhoto score, top impressions/likes — lack full user coverage)
- Markdown rendering effort estimate for client side (Trideep/Evan, flagged at Apr 28 discussion)
