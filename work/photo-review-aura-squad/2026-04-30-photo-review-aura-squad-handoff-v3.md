# Photo Review AURA Squad — Handoff v3

**Session date:** 2026-04-30
**Coverage:** 2026-04-16 → 2026-04-30

**Changes from v2**
- Domain Model: Terminology section added
- Update document: stream labels revised to Mainstream / Substream with project-vocabulary names

---

## Stage 1 Anchors

| Source | Anchor |
|---|---|
| Slack | #mgai-aura (C06FB511ZRR); #ml-photo-review (C0AMBBQGU6R — accessible via Glean, not via claude.ai MCP or bot token) |
| Jira | LP (hyperconnect.atlassian.net) |
| GitHub | matchgroup-ai/aura-demo, matchgroup-ai/ldm-models, matchgroup-ai/prompt-factory |
| Notion | HPCNT workspace — search by "AURA photo review" |
| Google Calendar | mumu.kim@match.com |

---

## Domain Model Instance

### Terminology

| Term | Definition |
|---|---|
| Photo Review | The product feature: AI-generated photo coaching delivered via a bottom sheet in the Tinder app |
| Eval N | Evaluation round (Eval 3, 4, 5, 6): structured human evaluation of Photo Review Prompt output quality |
| Prompt | The LLM prompt used by AURA to generate photo coaching; versioned (e.g., v4.6.x) |
| KB (Knowledge Base) | Curated concept descriptions used in the AURA Prompt; currently 25 lifestyle concepts |
| rSRR | Relative Send/Receive Ratio: photo quality signal used for ranking and the rSRR Predictor target |
| rSRR Predictor | The ML model predicting rSRR from photo features; developed by Ken Lee |
| VLM | Vision Language Model: Gemini Flash-Lite or Pro, used for photo concept inference |
| Prompt Factory | Internal tool for Prompt management, versioning, test sets, and human evaluation |
| Bottom sheet | The mobile UI component surfacing photo coaching insights |
| Repetitive cluster | A group of photos flagged as compositionally repetitive; subject to Best-keep + One-cluster-at-a-time model |
| Serving | The AURA serving architecture decision (Option 1 Batch Precompute vs. Option 2 On-demand Sync) |
| Option 2 (On-demand Sync) | Selected serving direction: Photo Review computed on-demand at app open |
| Backend Contract | The API spec between AURA ML and the Tinder app: `POST /v1/photo-review` |
| Spec Kick-off | Engineering alignment meeting for Option 2 implementation; preliminary session held Apr 28 |
| SOUP | Tinder backend team responsible for serving integration (Henry Au, Xincen Hao) |
| AURA Demo | The internal demo application used for Prompt iteration and Eval execution |
| Mainstream | A primary work stream: highest member coupling, most work items |
| Substream | A smaller, more independent work track running in parallel to the mainstreams |

---

### People

| Name | Role | Email |
|---|---|---|
| Biggie Choi | ML Lead / Project Lead | biggie.choi@match.com |
| Juniper Han | PM | juniper.han@match.com |
| Brandon Choi | MLSE (AURA Demo lead) | brandon.choi@match.com |
| Parker Cho | MLSE (Prompt Factory) | parker.cho@match.com |
| Claude Yoon | MLE (VLM / Concepts KB) | claude.yoon@match.com |
| Ken Lee | MLE (Evaluation, rSRR Predictor) | ken.lee@match.com |
| Link Lee | MLE (Analysis) — transferred to Tailor ~4/21 | link.lee@match.com |
| Mumu Kim | TL / Advisor | mumu.kim@match.com |
| Trideep Rath | Staff SWE ML, Tinder | trideep.rath@gotinder.com |
| Jaini Shah | Principal PM, Tinder | jaini.shah@gotinder.com |
| Evan (Tinder) | Tinder engineer (no last name confirmed) | — |
| Henry Au | SOUP backend engineer | — |
| Xincen Hao | SOUP backend engineer | — |
| Nancy Wu | Designer (Tinder, proposed one-cluster-at-a-time) | — |

---

### WorkItems (current status as of 2026-04-30)

| ID | Title | Owner | Status |
|---|---|---|---|
| WI-01 | rSRR Predictor report shared with Trideep (notified Apr 27, delivered Apr 29) | Biggie | Done |
| WI-02 | Prompt v4.6.x — repetitive cluster UI and prompt fix | Biggie | Done |
| WI-03 | Lifestyle 25-concept KB report v2.0 review | Biggie | In Progress |
| WI-04 | Eval 5 prompt review | Biggie, Juniper, Parker | In Progress |
| WI-05 | Team communication norms doc | Biggie | To Do |
| WI-06 | Privacy by Design doc for AURA demo | Biggie | To Do |
| WI-07 | Eval 3 results & Eval 4 execution | Juniper | Done |
| WI-08 | Apr 15–22 status update doc | Juniper | Done |
| WI-09 | Confluence PRD update | Juniper | Done |
| WI-10 | Pending product decisions resolved (LP-613) | Juniper | Done |
| WI-11 | Workshop attendance (Palo Alto, Apr 27–29) | Juniper | Done |
| WI-12 | Eval 5 preparation (KB, Prompt, bug fix) | Juniper, Brandon, Parker | In Progress |
| WI-13 | Workshop findings summary | Juniper | To Do |
| WI-14 | Copy/tone alignment for Eval 6 | Juniper | To Do |
| WI-15 | Serving options doc (LP-515) | Brandon | Done |
| WI-16 | Backend Contract (POST /v1/photo-review) | Brandon | Done |
| WI-17 | Prompt logical contradictions analysis | Brandon | Done |
| WI-18 | Gemini Flash-Lite latency benchmarking | Brandon | Done |
| WI-19 | Option 2 Spec Kick-off (LP-606) | Brandon | Pending (preliminary session held 4/28; formal meeting pending) |
| WI-20 | Dummy ML server for Backend Contract | Brandon | To Do |
| WI-21 | QoS investigation VertexAI | Brandon | To Do |
| WI-22 | Prompt Factory test set workflow simplification | Parker | Done |
| WI-23 | Prompt Factory knowledge sharing for CRS | Parker | Done |
| WI-24 | Eval 4 evaluation support | Parker | Done |
| WI-25 | Eval 5 iteration prep (LP-614) | Parker | In Progress |
| WI-26 | DLP compliance for AURA demo data (LP-617) | Parker | In Progress |
| WI-27 | Prompt Factory ↔ AURA ActionPlan schema contract (LP-574) | Parker | To Do |
| WI-28 | AURA Demo × Prompt Factory cleanup (LP-591) | Parker | To Do |
| WI-29 | Fake User Profile Generation (LP-602) | Parker | To Do |
| WI-30 | T5 25-concept KB report v2.0 (PR #260) | Claude Yoon | Done |
| WI-31 | ldm-models training/inference code (PR #38, #43) | Claude Yoon | Done |
| WI-32 | Demo → ML logic conversion doc (28 items) | Claude Yoon | Done |
| WI-33 | LLM-based concept description verification | Claude Yoon | In Progress |
| WI-34 | rSRR Predictor V1 experiment round 1 (LP-607) | Ken | Done |
| WI-35 | Dataset v5 cleanup (LP-565) | Ken | Done |
| WI-36 | Face close ratio data investigation | Ken | Done |
| WI-37 | rSRR Predictor external doc (LP-616) | Ken | In Progress |
| WI-38 | Repo/branch cleanup (LP-594) | Ken | Pending |
| WI-39 | LP-570 (analysis work) | Link | Done (before transfer) |
| WI-40 | Offloading experiment PR #69 in Prompt Factory | Link | Done (before transfer) |
| WI-41 | Photo Suppression V2 proposal shared in #ml-photo-review (Apr 20) | Trideep | Done |

---

### Decisions

| Decision | By | Date | Status |
|---|---|---|---|
| Custom ML model descoped from MVP | Biggie, Mumu | 4/22 | Resolved |
| Serving direction = Option 2 (On-demand Sync) | Juniper, Jaini, Mumu | 4/24 | Resolved |
| Bottom sheet timing = next app open | Evan, Jaini | 4/24–25 | Resolved |
| Coaching trigger = photo add/delete/replace only | Brandon, Evan | 4/28 | Resolved |
| Cadence: next-day new coaching on photo change, keep when unchanged | Juniper, Jaini, Brandon | 4/28 | Resolved |
| Repetitive cluster model: Best-keep + One-cluster-at-a-time | Jaini, Juniper, Trideep | 4/28 | Resolved |
| MVP language scope = English-only | Evan, Juniper | 4/28 | Resolved |
| AURA LLM skip signal (status = success/skip) | Brandon | 4/28 | Resolved |
| Bottom sheet dismiss policy: per-insight vs. full opt-out | Jaini | 4/28 | **Unresolved — Jaini explicitly deferred; engineering suggests full opt-out for MVP** |
| Photo tie-breaker metric for repetitive cluster ranking | Tinder team | — | **Unresolved — Tinder team input pending** |
| Profile Restrictions overlap exclusion rule | SOUP (SOUPD-129) | — | **Unresolved — pending SOUP finalization** |
| Markdown rendering effort (client side) | Trideep, Evan | 4/28 | **Unresolved — flagged as follow-up** |

---

## Unresolved Items (carry into Stage 3 of next session)

1. **Bottom sheet dismiss policy**: Jaini logged as open question at Apr 28 Spec Kick-off. Engineering suggestion is full opt-out for MVP; PM ratification pending from Jaini/Juniper.
2. **Photo tie-breaker metric**: Tinder team has not responded to request for a universal photo-level quality metric for ranking within repetitive clusters.
3. **Profile Restrictions overlap exclusion**: SOUPD-129 not yet finalized; Photo Review will exclude users in active Profile Restrictions experiment geo/cohort/assignment.
4. **Formal Option 2 Spec Kick-off outcome**: Preliminary session held Apr 28 with partial agreements recorded; formal meeting (LP-606) pending.
5. **Eval 5 results**: Target week of Apr 28; prep in progress at end of coverage period. Results not yet available.

---

## Coverage

2026-04-16 → 2026-04-30
