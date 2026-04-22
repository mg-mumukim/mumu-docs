| Key           | Value                          |
| ------------- | ------------------------------ |
| Source        | `https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg` |
| Downloaded at | `2026-04-22 15:44:25`        |

---

# Serving Options

Maintainer [Juniper Han](mailto:juniper.han@match.com) [Brandon Choi](mailto:brandon.choi@match.com)  
Reviewer [Mumu Kim](mailto:mumu.kim@match.com)  
Last updated 2026-04-15  
---

[About this document](#heading=h.ujax50se8oh0)

[Cost Dashboard](#heading=h.e52by4wh8l9)

[Summary](#heading=h.wf38ei3d67j2)

[Constraints](#heading=h.xehxwgb5x9me)

[Options](#heading=h.w7f2wvwjlhvv)

[Remaining Questions](#heading=h.733hmlwd1p9q)

---

# About this document

**2026-04-21 update**: We have committed to serving Photo Coaching **without real-time GPU serving or any self-hosted ML model**. All inference runs through the Gemini API (external vendor) on a single path. Option 3 (True Realtime, 0-5s) is eliminated for MVP scope — rationale below.

The open decision is now narrowed to **Option 1 (Gemini Batch precompute) vs Option 2 (On-demand Gemini sync, \~30s allow)**. This document outlines the remaining trade-offs, assumptions, and open questions needed to close that decision.

### Why True Realtime option (under 5-second generation) was eliminated

**Context**: Without self-hosted GPU serving, concept extraction and coaching text generation are merged into a single Gemini API call. The payload is 9 photos (\~2,300 vision tokens) \+ system prompt \+ KB \+ bio \+ output \= \~8,600+ multimodal tokens.

| \# | Reason | Detail |
| :---- | :---- | :---- |
| 1 | **No latency SLA from Google** | Gemini API SLA covers availability only. No committed response-time SLO exists. A 5s product SLA cannot be built on an un-SLA'd external service. |
| 2 | **9-photo vision payload is structurally too heavy for 5s** | Internal measurements: text-only Flash Lite \~1s (Tailor PII redaction); AURA 9-photo full pipeline \~1 min (measured 2026-04-13). Vision tokens are computationally heavier than text — linear extrapolation from text-only does not hold. |
| 3 | **Gemini latency variance is large** | CRS team monitors a Gemini Latency Heatmap (hour-of-day buckets) and p99 latency as a separate alert. Even if p50 were 3s, p95/p99 can reach 15-45s — a 5s hard SLA would cause mass failures. |
| 4 | **Option 3 was originally designed for self-hosted ML** | The original spec assumed a self-hosted concept extractor (Qwen3-VL, \<1s) doing the heavy lifting, with the LLM handling only lightweight text generation. Without that GPU layer the entire pipeline falls on one external API call — a fundamentally different cost structure. |
| 5 | **No room for fallback within 5s** | On Gemini timeout or error there is no retry budget. Option 2 (30s) allows one retry; Option 3 (5s) does not. |

**Revival conditions (for reference)**: (a) reintroduce a self-hosted GPU concept extractor — reverses the no-GPU decision, or (b) Google guarantees \< 3s p95 for 9-photo vision calls — not currently offered.

# Flow diagram

The two remaining options share the same components but differ in synchronization pattern.

## Components

## ![image1](images/69f87799e057.png)

```
graph LR
    Client["Tinder Client<br/>iOS / Android"]
    SOUP["SOUP Backend<br/>Tinder"]
    AURA["AURA ML API<br/>Python, MGAI"]
    Gemini["Gemini API<br/>external vendor"]
    DB[("Coaching Store<br/>Option 1 only")]
    S3[("Profile photos<br/>existing Tinder S3")]

    Client <-->|HTTPS| SOUP
    SOUP -->|POST /v1/photo-review| AURA
    AURA -->|Gemini SDK| Gemini
    AURA -.->|read photos| S3
    AURA -.->|write| DB
    SOUP -.->|read| DB
```

| Component | Owner | Stack | Role |
| :---- | :---- | :---- | :---- |
| **Tinder Client** | Tinder | iOS / Android native | Renders Profile Home and the coaching bottom sheet; no ML logic on device. |
| **SOUP Backend** | Tinder | Kotlin / Go | Owns user profile data and Profile Home entry. Decides when to request coaching and handles the user-facing response. |
| **AURA ML API** | MGAI | Python (FastAPI) | Accepts `POST /v1/photo-review` from SOUP, builds the Gemini payload, returns coaching JSON. Stateless request handler. |
| **Gemini API** | Google (vendor) | external HTTPS | Executes the vision \+ language call. Supports a sync path (real-time) and a Batch path (discounted, longer turnaround). |
| **Coaching Store** | MGAI (owned by AURA ML API) | DynamoDB / Redis / Postgres (TBD) | **Option 1 only.** Stores precomputed coaching keyed by `(user_id, profile_version)`. Accessed exclusively through AURA ML API — SOUP does not read/write directly. |
| **Profile photos (S3)** | Tinder (existing) | S3 | Source of the 9-photo payload. Already available; no new storage to provision. |

## Option 1 — Precompute \+ Push

![image2](images/1ee5398b44fa.png)

```
sequenceDiagram
    participant Sched as "Scheduler (cron or event)"
    participant SOUP as "SOUP Backend"
    participant AURA as "AURA ML API"
    participant Gemini as "Gemini Batch"
    participant DB as "Coaching Store"
    participant Client as "Tinder Client"

    rect rgb(240, 248, 255)
    Note over Sched,DB: Batch precompute (cadence TBD - hourly or daily)
    Sched->>SOUP: trigger batch job
    SOUP->>SOUP: select eligible users (DAU + profile-changed)
    SOUP->>AURA: POST /v1/photo-review/batch (N payloads)
    AURA->>Gemini: submit Batch job
    Note right of Gemini: turnaround pending (P0 #3)
    Gemini-->>AURA: batch results
    AURA->>DB: write coaching per (user_id, profile_version)
    AURA-->>SOUP: batch complete
    end

    rect rgb(245, 255, 245)
    Note over Client,AURA: User-facing read path (same contract as Option 2)
    Client->>SOUP: open Profile Home
    SOUP->>AURA: POST /v1/photo-review (same contract)
    AURA->>DB: lookup (user_id, profile_version)
    alt cache hit and fresh
        DB-->>AURA: coaching JSON
        AURA-->>SOUP: cached response (~100ms)
        SOUP-->>Client: Profile Home + bottom sheet
    else cache miss or stale
        AURA-->>SOUP: no coaching available
        SOUP-->>Client: Profile Home (no coaching)
        Note over AURA: hybrid fallback option: trigger Gemini sync here instead of returning empty
    end
    end
```

**Synchronization points**

- **SOUP always calls the same `POST /v1/photo-review` endpoint** — cache hit vs Gemini call is decided inside AURA, invisible to SOUP. This keeps the backend contract identical for Option 1 and Option 2 and enables hybrid fallback (precompute miss → on-demand generation) without SOUP changes.  
- **Eligible-user selection** — SOUP queries its own user DB; no AURA dependency for cohort building  
- **Batch job lifecycle** — Gemini Batch is asynchronous; AURA must persist `job_id` and reconcile via poll or webhook  
- **Profile change invalidation** — on profile edit, AURA invalidates the cached coaching (keyed by profile\_version). SOUP does not need to interact with the Coaching Store directly. Policy choice (mark stale vs event-driven rerun) tracked as open question P1 \#5 in Remaining Questions below  
- **Coaching Store ownership** — AURA ML API owns both writes (from batch) and reads (from user requests). SOUP never touches the store directly, reducing cross-team coupling

## Option 2 — On-demand (\~30s allow)

![image3](images/d9a228a3e258.png)

```
sequenceDiagram
    participant Client as "Tinder Client"
    participant SOUP as "SOUP Backend"
    participant AURA as "AURA ML API"
    participant Gemini as "Gemini sync"

    Client->>SOUP: open Profile Home
    SOUP-->>Client: render Profile Home skeleton (no coaching yet)
    SOUP->>AURA: POST /v1/photo-review (9 photos + bio + descriptors)
    AURA->>Gemini: vision + text call
    Note over AURA,Gemini: p50/p95/p99 pending (P0 #1)
    Gemini-->>AURA: coaching JSON
    AURA-->>SOUP: response (or 30s timeout)
    alt within 30s
        SOUP-->>Client: push bottom sheet in-session
    else timeout
        SOUP-->>Client: no coaching this session
    end
```

**Synchronization points**

- **No pre-stored state** — every Profile Home open that qualifies triggers a fresh call; no DB writes  
    
- **In-session wait** — Client must hold a "coaching pending" state for up to \~30s; UX must handle the user switching away mid-wait  
    
- **Timeout policy** — hard 30s? one retry? silent fallback? Listed as engineering decision in the Options table. Determined once P0 \#1 latency distribution is known  
    
- [ ] [Brandon Choi](mailto:brandon.choi@match.com) — diagrams drafted 2026-04-21, pending review by Mumu / Henry Au (SOUP backend lead)

# Options in one table

- [ ] [Juniper Han](mailto:juniper.han@match.com) **WIP**

|  | Option 1 (Precompute \+ Push) | Option 2 (On-demand, \~30s allow) |
| :---- | :---- | :---- |
| **Approach Explained** | Gemini Batch API. Compute coaching results ahead of time for a selected (eligible) user cohort via batch jobs. Experience is instant at view time, but availability is non-deterministic. | Gemini sync API. Start generating results only when the user opens Profile Home; surface result as soon as ready within a \~30s UX budget. |
| **Ideal trigger** | Scheduled batch (cadence TBD — hourly vs daily depends on staleness tolerance) | App-open → Profile Home entry |
| **Latency to result** | \~0s at view time (pre-populated); stale or missing for users outside the precompute set | **p50 \= 17.5s, p95 \= 25.0s, p99 \= 30.5s** (PF database, N=2,486, prompt v4+). Fits within 30s budget at p95; \~1% timeout at p99. |
| **Nudge to Profile Home** | Eligible user immediately gets the bottom sheet once they enter the app (only if precomputed result exists) | Eligible user gets the bottom sheet once the coaching message is ready while they are active on the app |
| **Notification timing** | Immediate (only if result exists). Users who create a new profile or change a photo between batches do not get coaching until the next batch. | Delayed (up to \~30s after app entry) |
| **Dependency** | **Batch pipeline, scheduler, storage for precomputed results. Batch Completion SLA** — `[P0 #3 pending]` actual Gemini Batch turnaround (1k profile test) determines feasible cadence. **Gemini Batch stability** — must support large-scale batch inference without stalling or throttling. | **Gemini sync quota / rate limit** — `[P1 #6 pending]`. Must handle peak QPS derived from Profile Home entry rate. **Bottom sheet UX** — disruptive timing and short session length can drop completion; may need "show on next app entrance" fallback. |
| **UX consistency** | Hit or miss (depends on precompute coverage) | Consistent but with \~30s loading |
| **Cost (monthly, steady-state, test region)** | Gemini Batch \~50% discount × N\_precompute | Standard Gemini × N\_profile\_home\_views. `[P0 #2 pending]` N\_views from Tinder data team. |
| **Pros Summary** | Zero latency at access time; most cost-efficient at scale; enables proactive surfacing without waiting | Less wasted computation; always fresh results (no staleness); simpler backend (no storage/push) |
| **Cons Summary** | Non-deterministic availability; wasted compute on users who never view results; inference storage triggers new privacy/retention review | Noticeable latency breaks user flow and may reduce engagement; cost scales linearly with view rate |
| **Engineering decision required** | (Eng) Batch SLA validation — `[P0 #3]`. (Eng) Backend decision: how to handle users missed in batch, newly eligible mid-cycle, partial batch failures. (Eng) Push channel (pre-populated fetch vs notification vs lazy load) — `[P1 #7 pending, Henry Au / Xincen Hao (SOUP backend)]`. | (Eng) Real-time latency validation — `[P0 #1]`. (Eng) Gemini sync QPS / quota confirmation — `[P1 #6]`. (Eng) Timeout / retry / fallback behavior for calls exceeding 30s. |
| **Product decision required** | Precompute set criteria — all eligible daily / DAU only / profile-change event-driven (`[P1 #5]`). Staleness tolerance (`[P1 #9]`). | Degraded fallback allowed? (`[P2 #12]`) What does user see on timeout? |
| **Privacy/Legal** | Stores coaching inference per user → retention, access/correction/deletion obligations apply (`[P1 #8 pending, Blake]`) | Ephemeral — coaching lives only in the response |
| **Suggestion** | `[To be filled after P0/P1 data arrives, target 4/23 EOD]` | `[To be filled after P0/P1 data arrives, target 4/23 EOD]` |
| **Caveat: we can't move forward with this option if** | Batch turnaround exceeds staleness tolerance, or precompute coverage drops below a PM-defined minimum | Gemini sync p95 exceeds 30s, or sync QPS quota cannot support peak load |

# Decision criteria

P0 \#1 latency data is now available. **Gemini Flash-Lite p95 \= 25.0s (under 30s) — Option 2 is technically viable.** The decision now comes down to tradeoffs, not feasibility gates.

## What the latency data tells us

- p95 \= 25.0s → 95% of requests complete within 30s. Option 2 works.  
- p99 \= 30.5s → \~1% of requests will timeout at 30s. Need graceful fallback.  
- p50 \= 17.5s → typical user waits \~18s. PM must accept this UX.

## Remaining decision factors

The choice between Option 1 and Option 2 depends on which axis stakeholders prioritize:

| Priority | Favors | Reasoning |
| :---- | :---- | :---- |
| **Ship faster** | Option 2 | 4-5 weeks vs 6-8 weeks. No batch infra, no storage, no scheduler. |
| **Lower cost** | Option 1 | Batch API 50% discount ($0.0043 vs $0.0086/call). Bounded compute (only eligible users, not every view). |
| **Full coverage** | Option 2 | Every user gets coaching on every visit. No cold-start gap. |
| **UX (no wait)** | Option 1 | 0ms at view time (when cached). But non-deterministic — some users get nothing. |
| **UX (consistent)** | Option 2 | Every user gets the same experience (\~18s wait). No "why did my friend get coaching but I didn't?" |
| **Privacy simplicity** | Option 2 | Ephemeral — no stored inferences, no retention obligations, no deletion pipeline. |
| **Infra simplicity** | Option 2 | Single request-response endpoint. No batch pipeline, scheduler, coaching store, or cache invalidation. |

## Open questions that will sharpen the decision

| Question | If answer is... | Then... |
| :---- | :---- | :---- |
| PM staleness tolerance (P1 \#9) | \< 1 hour | Option 1 batch cadence must be hourly — higher compute cost, reducing its cost advantage |
| PM staleness tolerance (P1 \#9) | \> 24 hours or "acceptable" | Option 1 daily batch is cheap and simple |
| Monthly budget cap (P2 \#11) | \< $33K/mo (test region) | Only Option 1 (batch) fits |
| Monthly budget cap (P2 \#11) | \> $66K/mo (test region) | Both options fit; cost is not the deciding factor |
| PM accepts \~18s loading UX? | No | Option 1 or Hybrid |
| PM accepts \~18s loading UX? | Yes | Option 2 is the simplest path |
| Hybrid fallback allowed? (P2 \#12) | Yes | Option 1 \+ Option 2 fallback gives best coverage at highest complexity |

# Timeline comparison

Estimated weeks from decision to production-ready, per option. Based on SOUP Q2 Engineering Capacity (Ryan Burns, 2026-04-02): Photo Review Backend \= 6 weeks at 75% confidence.

## Common prerequisites (both options)

| Step | Owner | Est. weeks | Status |
| :---- | :---- | :---- | :---- |
| Profile Home v4 ships (Photo Review card surface) | SOUP client \+ backend | In progress | Prerequisite — Photo Review slots into v4 Profile Home |
| Eval rounds complete \+ coaching copy frozen | MGAI (Juniper, Biggie, Parker) | \~1 wk remaining | Eval 4 fishfooding done 4/20, final polish in progress |
| Privacy by Design sign-off | Mumu → Blake / Legal | \~1 wk | PbD docs drafted, 4/9 legal sync done, awaiting final approval |
| Backend contract finalized | Mumu \+ Henry Au | Done | `POST /v1/photo-review` reviewed by both sides |

## Option 1 — Batch precompute

| Step | Owner | Est. weeks | Notes |
| :---- | :---- | :---- | :---- |
| AURA batch endpoint (`/v1/photo-review/batch`) | Brandon (MGAI) | 1-2 | Extends existing demo pipeline to accept batch payloads |
| Gemini Batch API integration \+ job lifecycle | Brandon (MGAI) | 1 | Submit/poll/reconcile pattern |
| Coaching Store setup (schema, write path) | Brandon (MGAI) | 1 | DynamoDB or Redis; keyed by (user\_id, profile\_version) |
| AURA read path (cache lookup in existing endpoint) | Brandon (MGAI) | 0.5 | `POST /v1/photo-review` checks store before calling Gemini |
| Batch scheduler (cron trigger \+ eligible user query) | Brandon \+ SOUP backend | 1 | Depends on how SOUP exposes eligible user list |
| SOUP integration (call existing endpoint, handle empty response) | Henry Au (SOUP backend) | 1-2 | Minimal — same contract, but needs "no coaching" UX path |
| E2E test \+ staging | Both | 1 |  |
| **Total** |  | **6-8 weeks** | Aligns with Ryan's 6-week estimate (75% confidence) |

## Option 2 — On-demand sync

| Step | Owner | Est. weeks | Notes |
| :---- | :---- | :---- | :---- |
| AURA sync endpoint hardening (error handling, timeout, retries) | Brandon (MGAI) | 1 | Demo already works; production-grade error handling needed |
| SOUP integration (call endpoint, manage 30s loading state, timeout UX) | Henry Au (SOUP backend) \+ client | 2-3 | Loading state is new UX — client needs spinner/skeleton \+ timeout fallback |
| Gemini QPS quota validation \+ monitoring | Brandon \+ Mumu | 0.5 | Confirm quota, set up alerts |
| E2E test \+ staging | Both | 1 |  |
| **Total** |  | **4-5 weeks** | Faster because no batch infra, no storage, no scheduler |

## Option 1+2 Hybrid

| Step | Owner | Est. weeks | Notes |
| :---- | :---- | :---- | :---- |
| All of Option 1 | see above | 6-8 |  |
| \+ sync fallback path inside AURA (cache miss → Gemini sync) | Brandon (MGAI) | 0.5 | Minimal addition if Option 2 endpoint already works |
| \+ SOUP timeout handling for fallback case | Henry Au (SOUP) | 0.5 | Same as Option 2's loading UX |
| **Total** |  | **7-9 weeks** | Most complete coverage but longest timeline |

**Summary**: Option 2 ships \~2-3 weeks faster than Option 1\. Hybrid adds \~1 week on top of Option 1\. If timeline is the top priority and latency data supports it, Option 2 wins.

# Risk comparison

Top risks per option, ranked by likelihood × impact.

## Option 1 — Batch precompute

| \# | Risk | Likelihood | Impact | Mitigation |
| :---- | :---- | :---- | :---- | :---- |
| 1 | **Gemini Batch turnaround exceeds staleness tolerance** — Google's batch SLA is best-effort (no committed turnaround). If batches take 6-12h instead of 1-2h, coaching is stale for most users. | Medium | High | Measure actual turnaround (P0 \#3). If unacceptable, fall back to Option 2 or use sync API for precompute (loses 50% discount). |
| 2 | **Privacy/retention obligations for stored inferences** — Coaching results stored per-user trigger GDPR/CCPA access, correction, and deletion rights. Legal may require audit logging, retention limits, and user-facing disclosure. | Medium | Medium | Blake review pending (P1 \#8). If obligations are heavy, this adds engineering scope (deletion pipeline, audit log) and delays launch. |
| 3 | **Wasted compute on users who never view Profile Home** — Batch precomputes coaching for N\_precompute users; only a fraction will actually open Profile Home before the next batch cycle. | Medium | Low | Start with conservative precompute set (DAU-only or profile-change event-driven, P1 \#5). Measure view rate post-launch and adjust. |
| 4 | **Cold-start users get no coaching** — Users who join or reactivate between batch cycles have no precomputed result. | Low | Medium | Hybrid fallback (sync for cache miss). Or accept no coaching for new users until next batch — PM decision (P2 \#12). |

## Option 2 — On-demand sync

| \# | Risk | Likelihood | Impact | Mitigation |
| :---- | :---- | :---- | :---- | :---- |
| 1 | **Gemini sync p95 exceeds 30s** — External API latency is unbounded and varies by time of day. If p95 is 40-60s, a large fraction of users see timeout with no coaching. | Medium-High | High | P0 \#1 benchmark is the go/no-go gate. If p95 \> 30s, Option 2 is dead as standalone. |
| 2 | **QPS throttling at peak** — Profile Home entry rate × Gemini sync calls may exceed quota. Throttled requests \= users with no coaching during peak hours. | Medium | High | Validate quota (P1 \#6). Request quota increase from Google if needed. Implement client-side rate limiting / staggered retry. |
| 3 | **30s loading UX reduces engagement** — Users waiting 30s for a bottom sheet may lose interest, leave Profile Home, or develop negative perception of the feature. | Medium | Medium | A/B test loading UX. "Show on next app entrance" fallback if user navigates away. PM acceptance is gating (P1 \#9). |
| 4 | **Cost scales linearly with views, not users** — A power user who opens Profile Home 5x/day triggers 5 Gemini calls. No caching layer means no cost amortization. | Low | Medium | Add response-level TTL cache in AURA (e.g. same profile\_version within 1h → return cached response without calling Gemini). Reduces cost without full Option 1 infra. |

## Side-by-side summary

|  | Option 1 (Batch) | Option 2 (Sync) | Hybrid |
| :---- | :---- | :---- | :---- |
| **Biggest risk** | Batch turnaround SLA | Latency p95 \> 30s | Combines both risk sets |
| **Risk that kills the option** | Turnaround \> staleness tolerance | p95 \> 30s | Neither (fallback exists) |
| **Timeline risk** | Longer (6-8 wk) | Shorter (4-5 wk) | Longest (7-9 wk) |
| **Privacy risk** | Higher (stored inferences) | Lower (ephemeral) | Same as Option 1 |
| **Cost risk** | Lower (batch discount, bounded compute) | Higher (scales with views) | Medium (batch \+ sync fallback) |

# Remaining product questions

- [ ] [Juniper Han](mailto:juniper.han@match.com)

image1

| Questions | Status | Owner | Target | Note |
| :---- | :---- | :---- | :---- | :---- |
| **Related to cost or coaching serving logic** |  |  |  |  |
| What was the adoption rate used in the estimation? 20%? | Open | Juniper / Jaini |  |  |
| For people who did **not** change their profile photo, do we want to leave the same coaching message on until they dismiss/ take an action **OR** do we want to be switching to a different lever message until they dismiss coaching? | Open | Juniper / Jaini |  |  |
| For people who changed their profile photo, what would be the cut-off time for profile updates to be included in the next round of coaching? (Assuming this would be low activity hour) | Open | Juniper / Jaini |  | Related to Option 1 staleness tolerance |
| **Option 1 vs 2 decision — open questions** |  |  |  |  |
| Staleness tolerance — how many hours/days can coaching stay fresh before it must be regenerated? (P1 \#9) | Open | Juniper \+ Jaini | 4/23 | Determines Option 1 batch cadence (hourly vs daily) |
| Precompute set criteria for Option 1 — A: all eligible daily / B: DAU only daily / C: profile-change event-driven (P1 \#5) | Open | Juniper \+ Jaini | 4/23 | Drives monthly cost ceiling for Option 1 |
| Degraded fallback — if Option 1 precompute miss, fall back to Option 2 (on-demand) or show no coaching? (P2 \#12) | Open | Juniper \+ Jaini |  | Enables a hybrid approach |
| Non–Profile-Home entry points planned (notification, email, push)? (P2 \#10) | Open | Juniper |  | Option 1 gains more inherent advantage if yes |
| **Others** |  |  |  |  |
| **True or False:** Photo Coaching bottom sheet will only appear on Day 0\. Once the user sees it (whether dismiss or accept), it will never appear again. | Open | Juniper |  |  |

# Engineering evidences

## Gemini Flash-Lite latency (P0 \#1)

Source: Prompt Factory PostgreSQL `generation_snapshots` table. Model: `gemini/gemini-3.1-flash-lite-preview`. Prompt version \>= v4 (excluding eval-tagged runs). Output token outliers removed (kept 1,500-5,000 range, 93.9% of data).

### N=2,486 coaching generations (2026-04-20 to 2026-04-22)

| Metric | Value |
| :---- | :---- |
| **p50** | **17,468ms (17.5s)** |
| **p90** | **22,445ms (22.4s)** |
| **p95** | **25,045ms (25.0s)** |
| **p99** | **30,507ms (30.5s)** |
| avg input\_tokens | 18,451 |
| avg output\_tokens | 3,384 |

Output token distribution: p5=2,065 p25=2,901 p50=3,352 p75=3,836 p95=4,735 (std=882, tight distribution).

### Conclusion

1. **Option 2 (30s budget): feasible but tight.** p95 \= 25.0s passes; p99 \= 30.5s is on the boundary. \~1% of requests may exceed 30s.  
2. **Option 3 (5s budget): infeasible.** p50 \= 17.5s — no path to 5s without self-hosted ML.  
3. **Timeout handling required.** Production must enforce a hard 30s timeout with graceful fallback (no coaching for that session). Expect \~1-5% timeout rate at p99 boundary.  
- [x] P0 \#1 — collected 2026-04-22 from PF database (N=2,486 coaching generations, prompt v4+, outlier-excluded).

## Gemini Batch API turnaround (P0 \#3)

- [ ] [Brandon Choi](mailto:brandon.choi@match.com) — target 4/23

Submit → retrieve end-to-end time for a 1k-profile batch. Determines Option 1 batch cadence feasibility (hourly vs daily).

No existing data — Gemini Batch API has not been used in AURA yet. Need a dedicated test run.

## Gemini sync API rate limit / quota (P1 \#6)

- [ ] [Brandon Choi](mailto:brandon.choi@match.com) / [Mumu Kim](mailto:mumu.kim@match.com) — target 4/22

Check Google Cloud Console for Tinder-account Gemini sync quota (RPM, TPM). Cross-reference with estimated peak QPS from Profile Home entry rate (P0 \#2).

Reference: CRS v2 noted prior concerns — "NanoBanana struggling at 1 QPS" (from Serving Options doc). Confirm whether this was resolved or if quota has been increased since.

## Profile Home entry rate (P0 \#2)

- [ ] Juniper / Harriet → Tinder data team — target 4/23

Average and p95 `profile_home_view` events per DAU. Drives the `N_profile_home_views` term in Option 2 monthly cost.

Reference from Engineering Docs: \~4% DAU change profile daily; steady-state trigger rate \~7% of DAU/day. But this is profile *change* rate, not Profile Home *view* rate — view rate is likely much higher.

## Profile edit → Profile Home re-view conversion (P0 \#4)

- [ ] Jaini → Tinder data team — target 4/23

% of users who re-enter Profile Home within N minutes after editing their profile. Quantifies Option 1 staleness impact.

## All numbers needed for decision — collection plan

| \# | Number | Current status | How to get | Owner | Target |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Latency** |  |  |  |  |  |
| P0 \#1 | Gemini Flash-Lite sync latency | **Done: p50=17.5s, p95=25.0s, p99=30.5s** (N=2,486, PF database, prompt v4+, outlier-excluded) | PF PostgreSQL `generation_snapshots` | Brandon | Done |
| P0 \#3 | Gemini Batch API turnaround (submit → retrieve) | **Not yet tested** | Submit 1k-profile batch via Batch API, measure wall-clock time to completion | Brandon | 4/23 |
| **Traffic / user behavior** |  |  |  |  |  |
| P0 \#2 | Profile Home view rate (views/DAU/day) | **Unknown** | Tinder Mode/Databricks query on `profile_home_view` events | Juniper → Tinder data | 4/23 |
| P0 \#4 | Profile edit → Profile Home re-view conversion | **Unknown** | Same data source, event sequence analysis | Jaini → Tinder data | 4/23 |
| **Cost** |  |  |  |  |  |
| — | Per-call cost (Flash-Lite sync) | **Known: $0.0086/call** | From Section 1 of this doc (Mumu) | — | Done |
| — | Per-call cost (Flash-Lite batch) | **Known: $0.0043/call** (50% discount) | Gemini Batch API pricing | — | Done |
| — | Option 1 monthly (test region steady-state) | **Known: \~$33K/mo** | 7.67M calls × $0.0043 | — | Done (from Section 4\) |
| — | Option 2 monthly (test region steady-state) | **Partially known: $66K/mo IF every trigger \= view** | 7.67M calls × $0.0086. But actual N\_views may differ from N\_triggers — need P0 \#2 to confirm | Brandon | After P0 \#2 |
| — | Monthly budget cap | **Unknown** | Ask Kevin Celustka / Juniper | Juniper | 4/23 |
| **Infra / quota** |  |  |  |  |  |
| P1 \#6 | Gemini sync QPS quota (Tinder account) | **Unknown** | Google Cloud Console → Vertex AI quotas page | Brandon / Mumu | 4/22 |
| P1 \#7 | SOUP backend LOE per option | **Unknown** | Async ask to Henry Au / Xincen Hao | Brandon | 4/24 |
| **Product / PM decisions** |  |  |  |  |  |
| P1 \#5 | Precompute set criteria (all eligible vs DAU vs event) | **PM decision needed** | Juniper \+ Jaini | Juniper | 4/23 |
| P1 \#8 | Privacy/Legal: stored inference obligations | **Under review** | Blake / Legal | Mumu | 4/24 |
| P1 \#9 | Staleness tolerance (hours/days) | **PM decision needed** | Juniper \+ Jaini | Juniper | 4/23 |
| P2 \#11 | Monthly budget cap | **Unknown** | Kevin Celustka | Juniper | 4/23 |
| P2 \#12 | Hybrid fallback allowed? | **PM decision needed** | Juniper \+ Jaini | Juniper | — |

**Numbers already available (no action needed):**

- Per-call cost: $0.0086 (sync), $0.0043 (batch) — from Mumu's LLM Cost section  
- Test region DAU: 3.65M (USA+CAN+AUS) — from Kevin's traffic model  
- Steady-state trigger rate: \~7% of DAU/day — from Kevin's analysis  
- Monthly call volume (test region): 7.67M — from Section 3.3  
- Day-1 cold start volume: 3.65M (test region) — one-time  
- Ramp-down to steady state: \~21 days — from Kevin's Mode report

---

### Appendix — Historical evidence (no longer load-bearing after 2026-04-21 decision)

- ~~Qwen3-VL performance~~ — self-hosted ML removed from scope  
- ~~Cost savings effect of Qwen3-VL~~ — self-hosted ML removed from scope





