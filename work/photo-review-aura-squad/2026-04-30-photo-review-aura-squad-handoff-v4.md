# Photo Review AURA Squad — Handoff v4

**Session date:** 2026-04-30
**Coverage:** 2026-04-16 → 2026-04-30

**Changes from v3**
- People table: Role and Org columns added; lead attribution corrected (Biggie = Lead, Mumu = TL)
- Domain Model: Milestones section added with Apr 30 PM-TL Sync timeline
- WorkItems: Eval 5 status corrected to Skipped; Eval 6 KB expansion added; SOUP estimate and PRD Part II added

---

## Stage 1 Anchors

| Source | Anchor |
|---|---|
| Slack | #mgai-aura (C06FB511ZRR); #ml-photo-review (C0AMBBQGU6R — accessible via Glean, not via claude.ai MCP or bot token) |
| Jira | LP (hyperconnect.atlassian.net) |
| GitHub | matchgroup-ai/aura-demo, matchgroup-ai/ldm-models, matchgroup-ai/prompt-factory |
| Notion | HPCNT workspace — search by "AURA photo review"; MGAI PM-TL Sync (Apr 30) |
| Google Calendar | mumu.kim@match.com |

---

## Domain Model Instance

### Terminology

| Term | Definition |
|---|---|
| Photo Review | The product feature: AI-generated photo coaching delivered via a bottom sheet in the Tinder app |
| Eval N | Evaluation round (Eval 3, 4, 5, 6): structured human evaluation of Photo Review Prompt output quality |
| Prompt | The LLM prompt used by AURA to generate photo coaching; versioned (e.g., v4.6.1) |
| KB (Knowledge Base) | Curated concept descriptions used in the AURA Prompt; currently 25 lifestyle concepts |
| rSRR | Relative Send/Receive Ratio: photo quality signal used for ranking and the rSRR Predictor target |
| rSRR Predictor | The ML model predicting rSRR from photo features; not in MVP scope |
| VLM | Vision Language Model: Gemini Flash-Lite or Pro, used for photo concept inference |
| Prompt Factory | Internal tool for Prompt management, versioning, test sets, and human evaluation |
| Bottom sheet | The mobile UI component surfacing photo coaching insights |
| Repetitive cluster | A group of photos flagged as compositionally repetitive; subject to Best-keep + One-cluster-at-a-time model |
| Option 2 (On-demand Sync) | Selected serving direction: Photo Review computed on-demand at app open |
| Backend Contract | The API spec between AURA ML and the Tinder app: `POST /v1/photo-review` |
| Spec Kick-off | Engineering alignment meeting for Option 2 implementation; preliminary session held Apr 28 |
| SOUP | Tinder backend team responsible for serving integration (Henry Au, Xincen Hao) |
| AURA Demo | The internal demo application used for Prompt iteration and Eval execution |
| Mainstream | A primary work stream: highest member coupling, most work items |
| Substream | A smaller, more independent work track running in parallel to the mainstreams |

---

### People

| Name | Role | Org | Email |
|---|---|---|---|
| Biggie Choi | Lead | internal | biggie.choi@match.com |
| Mumu Kim | TL | internal | mumu.kim@match.com |
| Juniper Han | PM | internal | juniper.han@match.com |
| Brandon Choi | MLSE | internal | brandon.choi@match.com |
| Parker Cho | MLSE | internal | parker.cho@match.com |
| Claude Yoon | MLE | internal | claude.yoon@match.com |
| Ken Lee | MLE | internal | ken.lee@match.com |
| Link Lee | MLE | internal (transferred to Tailor ~4/21) | link.lee@match.com |
| Trideep Rath | Staff-SWE | external (Tinder) | trideep.rath@gotinder.com |
| Jaini Shah | PM | external (Tinder) | jaini.shah@gotinder.com |
| Evan Keys | SWE | external (Tinder) | — |
| Henry Au | SWE | external (SOUP) | — |
| Xincen Hao | SWE | external (SOUP) | — |
| Nancy Wu | Designer | external (Tinder) | — |

---

### Milestones

| Description | Date | Status | Owner |
|---|---|---|---|
| Option 2 Decision Brief | 2026-04-29 | Met | Brandon |
| Mock server + ML server design doc | 2026-05-06 | Planned | Brandon |
| ML Deployment & integration testing | 2026-05-11 (week of) | Planned | Brandon |
| SOUP Q2 capacity available | 2026-05 (late May) | Planned | SOUP |
| BE Release QA | 2026-06-01 | Planned | — |

---

### WorkItems (current status as of 2026-04-30)

| ID | Title | Owner | Status |
|---|---|---|---|
| WI-01 | rSRR Predictor report shared with Trideep (notified Apr 27, delivered Apr 29) | Biggie | Done |
| WI-02 | Prompt v4.6.1 — Eval 4 feedback reflected (repetition cluster fix, diversity lever, blank photo handling) | Biggie + Parker | Done |
| WI-03 | Eval 6 KB expansion — 25-concept KB injection logic review; prompt v5.0.0 restructure | Biggie | In Progress |
| WI-04 | Eval 5 — **Skipped** (KB needs more time; rushing would produce inaccurate Flash-Lite assessment) | Juniper | Skipped |
| WI-05 | Eval 6 prompt review | Biggie, Juniper, Parker | To Do |
| WI-06 | Team communication norms doc | Biggie | To Do |
| WI-07 | Privacy by Design doc for AURA demo | Biggie | To Do |
| WI-08 | Eval 3 results & Eval 4 execution | Juniper | Done |
| WI-09 | Apr 15–22 status update doc | Juniper | Done |
| WI-10 | Confluence PRD update | Juniper | Done |
| WI-11 | Pending product decisions resolved (LP-613) | Juniper | Done |
| WI-12 | Workshop attendance (Palo Alto, Apr 27–29) | Juniper | Done |
| WI-13 | PRD Review Part II (Track A final text; bottom sheet direction) | Juniper | In Progress |
| WI-14 | Workshop findings summary | Juniper | To Do |
| WI-15 | Copy/tone alignment for Eval 6 (LP-610) | Juniper | To Do |
| WI-16 | Serving options doc — Option 2 selected (LP-515) | Brandon | Done |
| WI-17 | Backend Contract `POST /v1/photo-review` | Brandon | Done |
| WI-18 | Prompt logical contradictions analysis | Brandon | Done |
| WI-19 | Gemini Flash-Lite latency benchmarking | Brandon | Done |
| WI-20 | Option 2 Kick-off Decision Brief (LP-606) | Brandon | Done |
| WI-21 | SOUP engineering review request — Evan Keys: no blockers; Henry Au: pending | Brandon | Done |
| WI-22 | Mock server + ML server design doc | Brandon | To Do (target 5/6) |
| WI-23 | ML Deployment & integration testing | Brandon | To Do (target 5/11 week) |
| WI-24 | Dummy ML server for Backend Contract validation | Brandon | To Do |
| WI-25 | QoS investigation VertexAI (LP-621) | Brandon | To Do |
| WI-26 | SOUP effort estimate (Ryan Burns + Henry Au) | SOUP | In Progress |
| WI-27 | Prompt Factory test set workflow simplification | Parker | Done |
| WI-28 | Prompt Factory knowledge sharing for Chemistry | Parker | Done |
| WI-29 | Eval 4 evaluation support | Parker | Done |
| WI-30 | Prompt Factory eval infra (caching bug, testset workflow, compare screen) | Parker | Done |
| WI-31 | DLP compliance for AURA demo data (LP-617) — dry-run complete; 470 active users retained | Parker | In Progress (execution 5/6) |
| WI-32 | Prompt Factory ↔ AURA ActionPlan schema contract (LP-574) | Parker | To Do |
| WI-33 | AURA Demo × Prompt Factory cleanup (LP-591) | Parker | To Do |
| WI-34 | Fake User Profile Generation (LP-602) | Parker | To Do |
| WI-35 | T5 25-concept KB report v2.0 (PR #260) | Claude Yoon | Done |
| WI-36 | ldm-models training/inference code (PR #38, #43) | Claude Yoon | Done |
| WI-37 | Demo → ML logic conversion doc (28 items) | Claude Yoon | Done |
| WI-38 | Concept labeling dataset (~5,000 training points) | Claude Yoon | Done |
| WI-39 | LLM-based concept description verification | Claude Yoon | In Progress |
| WI-40 | rSRR Predictor V1 experiment round 1 (LP-607) | Ken | Done |
| WI-41 | Training dataset v5 cleanup (LP-565) | Ken | Done |
| WI-42 | Face close ratio data investigation | Ken | Done |
| WI-43 | rSRR Predictor external doc for Trideep (LP-616) | Ken | In Progress |
| WI-44 | Repo/branch cleanup (LP-594) | Ken | Pending |
| WI-45 | LP-570 analysis work | Link | Done (before transfer) |
| WI-46 | Offloading experiment PR #69 in Prompt Factory | Link | Done (before transfer) |
| WI-47 | Photo Suppression V2 proposal shared in #ml-photo-review (Apr 20) | Trideep | Done |

---

### Decisions

| Decision | By | Date | Status |
|---|---|---|---|
| Custom ML model descoped from MVP | Biggie, Mumu | 4/22 | Resolved |
| Serving direction = Option 2 (On-demand Sync) | Juniper, Jaini, Mumu | 4/24 | Resolved |
| Bottom sheet timing = next app open | Evan Keys, Jaini | 4/24–25 | Resolved |
| Coaching trigger = photo add/delete/replace only | Brandon, Evan Keys | 4/28 | Resolved |
| Cadence: next coaching on photo change, keep when unchanged | Juniper, Jaini, Brandon | 4/28 | Resolved |
| Repetitive cluster model: Best-keep + One-cluster-at-a-time | Jaini, Juniper, Trideep | 4/28 | Resolved |
| MVP language scope = English-only | Evan Keys, Juniper | 4/28 | Resolved |
| AURA LLM skip signal (status = success/skip) | Brandon | 4/28 | Resolved |
| Eval 5 skipped; next iteration = Eval 6 | Juniper | ~4/29 | Resolved |
| Gemini Pro kept as MVP fallback option | Biggie (+ 4/29 discussion) | 4/29 | Resolved |
| Bottom sheet dismiss policy: per-insight vs. full opt-out | Jaini | 4/28 | **Unresolved — Jaini deferred; engineering preference is full opt-out** |
| Photo tie-breaker metric for repetitive cluster ranking | Tinder team | — | **Unresolved — Tinder team input pending** |
| Profile Restrictions overlap exclusion rule | SOUP (SOUPD-129) | — | **Unresolved — pending SOUP finalization** |
| Markdown rendering effort (client side) | Trideep, Evan Keys | 4/28 | **Unresolved — follow-up pending** |
| OpenAI via Bedrock as LLM API alternative | Ben Shin (raised) | 4/29 | **Unresolved — cost estimate in progress (Kevin)** |

---

## Unresolved Items (carry into Stage 3 of next session)

1. **Bottom sheet dismiss policy**: Jaini deferred at Apr 28 Spec Kick-off; engineering preference full opt-out for MVP; PM ratification pending.
2. **Photo tie-breaker metric**: Tinder team has not responded. SmartPhoto score and top impressions/likes both lack full user coverage.
3. **Profile Restrictions overlap exclusion**: SOUPD-129 not finalized.
4. **SOUP effort estimate**: Ryan Burns and Henry Au working session scheduled Apr 30; MG AI side schedule not yet planned.
5. **Eval 6 KB injection logic**: Injecting full 25-concept KB exceeds feasible LLM load; KB usage logic and prompt v5.0.0 restructure in progress.
6. **MVP LLM API**: Gemini Pro kept as fallback; Bedrock/OpenAI option raised by Ben Shin; cost estimate in progress.

---

## Coverage

2026-04-16 → 2026-04-30
