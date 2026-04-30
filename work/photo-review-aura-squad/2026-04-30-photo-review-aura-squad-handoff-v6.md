# Photo Review AURA Squad — Handoff v6

**Session date:** 2026-04-30
**Coverage:** 4/16 ~ 4/30

**Changes from v5**
- VLM/Concepts KB: KB v2.0 handed off to Biggie (Mainstream 1); KB injection logic is M1 internal. Description verification (WI-37) still in progress.
- WI-06 Eval 6 run: DLP (WI-31) removed from blockedBy; riskFactor note added instead (pool may shrink 160→140, not a hard gate)
- rSRR Predictor: no operational dependency on Mainstream 2 (post-MVP scope, separate track)
- Date format: M/D throughout

---

## Stage 1 Anchors

| Source | Anchor |
|---|---|
| Slack | #mgai-aura (C06FB511ZRR); #ml-photo-review (C0AMBBQGU6R — via Glean only) |
| Jira | LP (hyperconnect.atlassian.net) |
| GitHub | matchgroup-ai/aura-demo, matchgroup-ai/ldm-models, matchgroup-ai/prompt-factory |
| Notion | HPCNT workspace — search "AURA photo review"; MGAI PM-TL Sync 4/30 |
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
| rSRR Predictor | ML model predicting rSRR from photo features; post-MVP research track, not in MVP scope |
| VLM | Vision Language Model: Gemini Flash-Lite or Pro, used for photo concept inference |
| Prompt Factory | Internal tool for Prompt management, versioning, test sets, and human evaluation |
| Bottom sheet | The mobile UI component surfacing photo coaching insights |
| Repetitive cluster | A group of photos flagged as compositionally repetitive; subject to Best-keep + One-cluster-at-a-time model |
| Option 2 (On-demand Sync) | Selected serving direction: Photo Review computed on-demand at app open |
| Backend Contract | The API spec between AURA ML and the Tinder app: `POST /v1/photo-review` |
| Spec Kick-off | Engineering alignment meeting; Part 1 4/27, Part 2 4/29 |
| SOUP | Tinder backend team responsible for serving integration (TL: Nathan Lamb; dev: Henry Au; manager: Ryan Burns) |
| AURA Demo | The internal demo application used for Prompt iteration and Eval execution |
| Mainstream | A primary work stream: highest member coupling, most work items |
| Substream | A smaller, more independent work track running in parallel to the mainstreams |

---

### People

| Name | Role | Org | Email |
|---|---|---|---|
| Biggie Choi | Lead | internal | biggie.choi@match.com |
| Mumu Kim | Advisor | internal | mumu.kim@match.com |
| Juniper Han | PM | internal | juniper.han@match.com |
| Brandon Choi | MLSE | internal | brandon.choi@match.com |
| Parker Cho | MLSE | internal | parker.cho@match.com |
| Claude Yoon | MLE | internal | claude.yoon@match.com |
| Ken Lee | MLE | internal | ken.lee@match.com |
| Link Lee | MLE | internal (transferred to Tailor ~4/21) | link.lee@match.com |
| Trideep Rath | Staff-SWE | external (Tinder ML) | trideep.rath@gotinder.com |
| Jaini Shah | PM | external (Tinder) | jaini.shah@gotinder.com |
| Ryan Burns | Sr. Eng Manager | external (Tinder SOUP) | ryan.burns@gotinder.com |
| Nathan Lamb | TL | external (Tinder SOUP) | nathan.lamb@gotinder.com |
| Henry Au | SWE III Backend | external (Tinder SOUP) | henry.au@gotinder.com |
| Evan Keys | Sr. SWE Backend | external (Tinder Canvas team) | evan.keys@gotinder.com |
| Xincen Hao | Director, Engineering | external (Tinder) | xincen.hao@gotinder.com |
| Nancy Wu | Designer | external (Tinder) | — |

---

### Milestones

| Description | Date | Status | Owner |
|---|---|---|---|
| Option 2 Decision Brief | 4/29 | Met | Brandon |
| Mock server + ML server design doc | 5/6 | Planned | Brandon |
| ML Deployment & integration testing | 5/11 (week of) | Planned | Brandon |
| SOUP Q2 capacity available | late May | Planned | SOUP |
| BE Release QA | 6/1 | Planned | — |

---

### WorkItems (current status as of 4/30)

| ID | Title | Owner | Status | blockedBy | riskFactor |
|---|---|---|---|---|---|
| WI-01 | rSRR Predictor report delivered to Trideep (notified 4/27, delivered 4/29) | Biggie | Done | — | — |
| WI-02 | Prompt v4.6.1 — Eval 4 feedback reflected | Biggie + Parker | Done | — | — |
| WI-03 | Eval 6 KB expansion — injection logic review (KB handed off from Claude Yoon) | Biggie | In Progress | — | — |
| WI-04 | Prompt v5.0.0 restructure for Eval 6 | Biggie | In Progress | WI-03 (KB logic decision first) | — |
| WI-05 | Eval 5 | Juniper | Skipped | — | — |
| WI-06 | Eval 6 run | Biggie, Juniper, Parker | To Do | WI-03, WI-04 | DLP (WI-31) execution on 5/6 may reduce eval pool 160→140 |
| WI-07 | Team communication norms doc | Biggie | To Do | — | — |
| WI-08 | Privacy by Design doc for AURA demo | Biggie | To Do | — | — |
| WI-09 | Eval 3 results & Eval 4 execution | Juniper | Done | — | — |
| WI-10 | Apr 15–22 status update doc | Juniper | Done | — | — |
| WI-11 | Confluence PRD update | Juniper | Done | — | — |
| WI-12 | Pending product decisions resolved | Juniper | Done | — | — |
| WI-13 | Workshop attendance (Palo Alto, 4/27 ~ 4/29) | Juniper | Done | — | — |
| WI-14 | PRD Review Part II — Track A final text + bottom sheet | Juniper | In Progress | — | — |
| WI-15 | Workshop findings summary | Juniper | To Do | — | — |
| WI-16 | Copy/tone alignment for Eval 6 baseline | Juniper | To Do | — | — |
| WI-17 | Serving options doc — Option 2 selected | Brandon | Done | — | — |
| WI-18 | Backend Contract `POST /v1/photo-review` | Brandon | Done | — | — |
| WI-19 | Prompt logical contradictions analysis | Brandon | Done | — | — |
| WI-20 | Gemini Flash-Lite latency benchmarking | Brandon | Done | — | — |
| WI-21 | Option 2 Kick-off Decision Brief | Brandon | Done | — | — |
| WI-22 | Mock server + ML server design doc | Brandon | To Do (5/6) | contract alignment this week | — |
| WI-23 | ML Deployment & integration testing | Brandon | To Do (5/11 wk) | WI-22 | Full e2e with SOUP only after SOUP Q2 (late May) |
| WI-24 | SOUP effort estimate | Ryan Burns, Henry Au | In Progress | — | — |
| WI-25 | QoS investigation VertexAI | Brandon | To Do | — | — |
| WI-26 | AURA Demo × Prompt Factory cleanup | Parker | To Do | — | — |
| WI-27 | Fake User Profile Generation | Parker | To Do | — | — |
| WI-28 | Prompt Factory test set workflow simplification | Parker | Done | — | — |
| WI-29 | Prompt Factory knowledge share (Chemistry) | Parker | Done | — | — |
| WI-30 | Eval 4 evaluation support | Parker | Done | — | — |
| WI-31 | DLP compliance — AURA demo data; Parker on vacation, verbal agreement for 5/6; dry-run complete, 470 users retained | Parker | In Progress (5/6) | — | Eval pool shrinks 160→140 after execution |
| WI-32 | Prompt Factory ↔ AURA ActionPlan schema contract | Parker | To Do | — | — |
| WI-33 | Lifestyle Concepts KB v2.0 — handed off to Biggie (M1) | Claude Yoon | Done | — | — |
| WI-34 | ldm-models training/inference code (PR #38, #43) | Claude Yoon | Done | — | — |
| WI-35 | Demo → ML logic conversion doc (28 items) | Claude Yoon | Done | — | — |
| WI-36 | Concept labeling dataset (~5,000 training points) | Claude Yoon | Done | — | — |
| WI-37 | LLM-based concept description verification | Claude Yoon | In Progress | — | — |
| WI-38 | rSRR Predictor V1 experiment round 1 | Ken | Done | — | — |
| WI-39 | Training dataset v5 cleanup | Ken | Done | — | — |
| WI-40 | Face close ratio data investigation | Ken | Done | — | — |
| WI-41 | rSRR Predictor external doc for Trideep | Ken | In Progress | — | — |
| WI-42 | Repo/branch cleanup | Ken | Pending | — | — |
| WI-43 | Eval 4 feedback → AURA Demo (prompt v4.6.1 + UI fixes) | Parker | Done | — | — |
| WI-44 | Prompt Factory eval infra (caching bug, testset compare, dataset lineage) | Parker | Done | — | — |
| WI-45 | T5 KB injection fix | Parker | Done | — | — |
| WI-46 | Photo Suppression V2 proposal shared in #ml-photo-review | Trideep | Done | — | — |
| WI-47 | LP-570 analysis work | Link | Done (before transfer) | — | — |
| WI-48 | Offloading experiment PR #69 in Prompt Factory | Link | Done (before transfer) | — | — |

---

### Decisions

| Decision | By | Date | Status |
|---|---|---|---|
| Custom ML model descoped from MVP | Biggie, Mumu | 4/22 | Resolved |
| Serving direction = Option 2 (On-demand Sync) | Juniper, Jaini, Mumu | 4/24 | Resolved |
| Bottom sheet timing = next app open | Evan Keys, Jaini | 4/24 ~ 4/25 | Resolved |
| Coaching trigger = photo add/delete/replace only | Brandon, Evan Keys | 4/28 | Resolved |
| Cadence: next coaching on photo change, keep when unchanged | Juniper, Jaini, Brandon | 4/28 | Resolved |
| Repetitive cluster model: Best-keep + One-cluster-at-a-time | Jaini, Juniper, Trideep | 4/28 | Resolved |
| MVP language scope = English-only | Evan Keys, Juniper | 4/28 | Resolved |
| AURA LLM skip signal (status = success/skip) | Brandon | 4/28 | Resolved |
| Eval 5 skipped; next iteration = Eval 6 | Juniper | ~4/29 | Resolved |
| Gemini Pro kept as MVP fallback option | Biggie (+ 4/29 discussion) | 4/29 | Resolved |
| Bottom sheet dismiss policy: per-insight vs. full opt-out | Jaini | 4/28 | **Unresolved** — engineering preference: full opt-out |
| Photo tie-breaker metric for repetitive cluster ranking | Tinder team | — | **Unresolved** — input pending |
| Profile Restrictions overlap exclusion rule | SOUP (SOUPD-129) | — | **Unresolved** — pending SOUP finalization |
| Markdown rendering effort (client side) | Trideep, Evan Keys | 4/28 | **Unresolved** — follow-up pending |
| OpenAI via Bedrock as LLM API alternative | Ben Shin (raised) | 4/29 | **Unresolved** — cost estimate in progress (Kevin) |

---

## Unresolved Items (carry into Stage 3 of next session)

1. **Bottom sheet dismiss policy**: Jaini deferred 4/28; engineering preference full opt-out; PM ratification pending.
2. **Photo tie-breaker metric**: Tinder team input pending. SmartPhoto and top impressions/likes lack full user coverage.
3. **Profile Restrictions overlap exclusion**: SOUPD-129 not finalized.
4. **SOUP effort estimate**: Ryan Burns + Henry Au working session 4/30; MG AI schedule not yet planned.
5. **Eval 6 KB injection logic**: Injecting full 25-concept KB adds 13+ atoms/profile. KB usage logic + prompt v5.0.0 restructure both block Eval 6 run (WI-03, WI-04).
6. **MVP LLM API**: Gemini Pro as fallback; Bedrock/OpenAI option raised by Ben Shin; cost estimate in progress (Kevin).

---

## Coverage

4/16 ~ 4/30
