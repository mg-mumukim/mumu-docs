<!-- sources: same as update-v11 -->

# Photo Review · AURA Squad · Session Handoff

**Date:** 2026-04-30
**Coverage:** 4/16 → 4/30

---

## Stage 1 Anchors

| Source | Anchor |
|---|---|
| Slack | #mgai-aura (C06FB511ZRR) |
| Jira | LP (hyperconnect.atlassian.net) |
| GitHub | matchgroup-ai/aura-demo, matchgroup-ai/ldm-models |
| Notion | MG AI Meeting Notes (collection://f68f73ca-0ddb-4030-a9e9-35a10c408374) |

---

## People

| Name | Role | Org |
|---|---|---|
| Biggie Choi | Lead | internal |
| Mumu Kim | Advisor | internal |
| Juniper Han | PM | external (Tinder) |
| Brandon Choi | MLSE | internal |
| Parker Cho | MLSE (demo/infra) | internal |
| Ken Lee | MLE | internal |
| Claude Yoon | MLE | internal |
| Trideep Rath | Staff SWE ML | external (Tinder ML/Gen AI) |
| Jaini Shah | Counterpart (PM) | external (Tinder) |
| Henry Au | Counterpart (SOUP eng) | external (Tinder) |
| Ryan Burns | Counterpart (SOUP eng) | external (Tinder) |
| Nathan Lamb | Counterpart (SOUP TL) | external (Tinder) |
| Evan Keys | Counterpart (Canvas eng) | external (Tinder) |
| Kevin Celustka | Counterpart (data) | external (Tinder) |
| Ben Shin | Counterpart (infra/platform) | external (Tinder) |
| Nancy Wu | Counterpart (design) | external (Tinder) |

Exited this period: **Link Lee** (~4/21, transferred to Tailor/Chemistry)

---

## Domain Model Instance

### WorkItems

**Mainstream 1: Engineering Kickoff**

| WorkItem | Owner | Status | Key Events |
|---|---|---|---|
| Serving option decision | Brandon | Done | ~4/22 DecisionMade: Option 2 adopted after Trideep sync |
| Option 2 Kick-off Decision Brief | Brandon | Done | 4/29 StatusChanged(→Done): Track A + Track B; SOUP review initiated |
| SOUP Track B review | SOUP eng | InProgress | Evan Keys done (no blockers); Henry Au pending as of 4/30 |
| SOUP effort estimation | Ryan Burns | InProgress | 4/29-30 started; Evan+Henry working session 4/30 |
| PRD Part I (team-wide review) | Juniper | Done | 4/28 Done; engineering sync blockers cleared (LP-613) |
| Spec kick-off meetings | Jaini Shah | Done | 4/27 Part 1 + 4/29 Part 2; product spec decisions resolved (see Decisions); Nancy Wu items pending |
| PRD Part II (Track A final text) | Juniper | InProgress | Bottom sheet direction aligned; Nancy Wu confirmation pending on: bottom sheet image approach and fallback copy approach |
| Mock server + ML server design doc | Brandon | NotStarted | Target 5/6 PT morning; events logging + user ID gaps to address |
| MGAI dev + deployment schedule | Mumu | NotStarted | Not yet planned; needs scheduling week of 4/30 |

**Mainstream 2: Demo Eval**

| WorkItem | Owner | Status | Key Events |
|---|---|---|---|
| Eval 4 (Flash-Lite vs Pro) | Juniper | Done | ~4/22-25 Done; repetitive detection quality issues found; prompt v4.6.1 + UI fixes applied |
| Eval 5 (KB v1.0 coaching eval) | Juniper | Skipped | 4/30 SkippedEvent: KB quality needs more time; reason confirmed |
| Eval 6 (KB expansion + copy/tone) | Biggie / Juniper | InProgress | KB v2.0 stats validated; diversity taxonomy finalized (5 dims); prompt v5.0.0 restructuring started; target week 5/4 |
| Photo Review copy/tone alignment | Juniper | InProgress | LP-610; polished baseline for Eval 6 copy eval |
| MVP LLM API model selection | Juniper / Mumu | Unresolved | Pro kept as fallback; Flash-Lite current default; cost estimate pending (Kevin); OpenAI via Bedrock suggested (Ben Shin) |
| Prompt v4.6.1 deployment | Biggie + Parker | Done | Repetition cluster fix, diversity lever simplification, blank photo handling, UI priority logic |
| KB v2.0 (25-concept lifestyle) | Biggie / Claude Yoon | Done (artifact) | Cohen's d, Bonferroni, gender×age stratification validated; registered in Prompt Factory; T5 PHOTO_COACHABLE metadata bug fixed; diversity taxonomy: 5 dims (Skills/Activities, Social Life, Lifestyle/Interests, Diverse Locations, Diverse Poses) |
| Prompt Factory infra (caching, multi-pod, human eval) | Parker | Done | Coaching snapshot caching bug fixed; LP-564 (test set workflow) done; human eval page functional |

**Substream: Manual DLP**

| WorkItem | Owner | Status | Key Events |
|---|---|---|---|
| Dataset v5 deletion (S3) | Ken | Done | LP-565; no AURA demo impact confirmed before deletion |
| ldm-models repo cleanup | Ken | InProgress | LP-594 Pending; branch cleanup + PR sharing |
| AURA demo data DLP | Parker | InProgress | LP-617; dry run done; ~470 active users preserved; eval pool 160→140; target 5/6 |

**Substream: rSRR Predictor (post-MVP research)**

| WorkItem | Owner | Status | Key Events |
|---|---|---|---|
| v1 training + evaluation | Ken | Done | LP-607; Qwen3-VL 8B + LoRA; pairwise ranking ≥ profile-ranker baseline |
| Trideep external brief | Ken | InProgress | LP-616; Trideep requested lesson-learned and SmartPhoto alignment context |

---

### Decisions

| Statement | By | Status | Rationale |
|---|---|---|---|
| Option 2 (on-demand, ~30s) as MVP serving path | Brandon + Trideep | Adopted (~4/22) | Universal coverage (no batch-miss), ephemeral inference (no privacy/retention complexity), single endpoint (simpler build). Trade-off: nudge arrives ~30s after app open when user is mid-swipe. |
| Custom ML model excluded from MVP scope | Team | Adopted (~4/22) | Quality not stable enough within timeline; no significant quality benefit vs. Gemini Pro; waiting would block serving and engineering decisions. ML research findings still feed into Knowledge Base. |
| Eval 5 skipped | Juniper | Adopted (4/30) | KB needs more time; rushing would produce an inaccurate evaluation of Flash-Lite overall performance, not just KB quality. |
| MVP LLM API model (Flash-Lite vs Pro) | Juniper / Mumu | Unresolved | 4/29 meeting kept Pro open as a scope lever for test region rollout; cost estimate in progress. Not a quality failure signal for Flash-Lite. |
| Production LLM model not finalized | Mumu | Unresolved | Flash-Lite current demo default; Pro and OpenAI via Bedrock both under consideration; eng-region 100% rollout gated on optimization completion. |
| Max insights cap = 3 per user | Jaini Shah | Adopted (4/29) | Prevents experience from feeling repetitive; cooldown between insights provides additional relief. |
| Cooldown: 1 day for Segment 1 (6+ photos); 0 days for Segment 2 (<6 photos) | Jaini Shah | Adopted (4/29) | Both backend configurable. Segment 2 excluded from cooldown so users can receive an updated insight immediately after adding a photo toward their count goal. |
| No true opt-out for MVP | Jaini Shah | Adopted (4/27) | Users can dismiss; max cap + cooldown mitigates annoyance; full opt-out deferred post-MVP. Experiment targeted at safe geos only. |
| Experiment structure: Control + Variant 1 with levers randomized | Jaini Shah | Adopted (4/29) | Control: new profile home, no photo review. Variant 1: new profile home + photo review, benefit and title levers randomized. Specific lever combinations adjustable post-launch via MGAI/ML side without a new lever. |

---

## Milestones

| Description | Date | Status | Owner |
|---|---|---|---|
| Option 2 Decision Brief | 4/29 | Met | Brandon |
| Mock server + ML server design doc | 5/6 | Planned | Brandon |
| AURA demo DLP execution | 5/6 | Planned | Parker |
| Eval 6 | ~5/4 week | Planned | Biggie / Juniper |
| ML deployment & integration testing | ~5/11 week | Planned | Brandon |
| SOUP Q2 engineering capacity | late May | Planned | Ryan Burns (SOUP) |
| BE Release QA | 6/1 | Planned | — |

---

## Unresolved Items (carry to next session)

1. **MVP LLM API model**: Flash-Lite vs Pro vs OpenAI via Bedrock; Kevin's cost estimate not yet complete
2. **MGAI development + deployment schedule**: not yet planned; Mumu flagged as needed for the week of 4/30
3. **Henry Au Track B review**: not confirmed complete as of 4/30
4. **Profile Home view frequency data**: needed for actual Option 2 cost estimate; data ask pending with Tinder data team
5. **Gemini sync QPS quota** on Tinder's Vertex account: not yet confirmed
6. **Privacy/Legal review on stored inferences**: Option 2 is ephemeral, but review still open per serving options doc — Blake ask from Mumu
7. **Bottom sheet image approach**: Nancy Wu to decide — Option 1 (catch-all image, works for both add and replace CTAs) vs Option 2 (separate image per primary action); unresolved as of 4/29
8. **Fallback copy approach**: Nancy Wu action item; 3 options: (1) control generic copy, (2) functional copy, (3) custom copy per scenario; unresolved as of 4/29
9. **Non-LLM copy finalization timeline**: when hardcoded/template copy must be ready; no date set; required to plan parallel asset preparation
10. **Engineering sync timing with SOUP**: when to open dedicated engineering sync; raised in spec kick-off 4/29, not yet scheduled

---

## Coverage

4/16 → 4/30
