# SmartPhoto V4 — Engineering Report

> **Sources:**
> - [Men's Smart Photo v4 (Confluence, SOUP)](https://gotinder.atlassian.net/wiki/spaces/SOUP/pages/1333133359) — Requirements doc
> - [Smart Photo Service (Confluence, ENG)](https://gotinder.atlassian.net/wiki/spaces/ENG/pages/27947519) — V2/V3 architecture
> - [smartphotosvllm Runbook (GitHub)](https://github.com/TinderBackend/revive.mlinfra/blob/main/service/smartphotosvllm/Runbook.md) — vLLM service
> - [SmartPhoto Worker Runbook (GitHub)](https://github.com/TinderBackend/revive.discovery/blob/main/worker/smartphoto/RUNBOOK.md) — Worker architecture
> - [PR #2175 — Smart Photos V4 LLM Interface and Ranking](https://github.com/TinderBackend/revive.mlinfra/pull/2175) — Decision Engine ranking module
> - [PR #989 — Migrate smartphoto-backfill to Spring Kafka](https://github.com/TinderBackend/revive.discovery/pull/989) — Backfill worker rewrite
> - [PR #925 — Kafka fetch smoothing](https://github.com/TinderBackend/revive.discovery/pull/925) — Traffic burst mitigation
> - [PR #907 — Country expansion](https://github.com/TinderBackend/revive.discovery/pull/907) — Global rollout
> - [ML-1307 (Jira, Tinder)](https://gotinder.atlassian.net/browse/ML-1307) — Epic: SmartPhoto V4 and FU enhancement of Decision Engine
> - [LP-309 (Jira, Hyperconnect)](https://hyperconnect.atlassian.net/browse/LP-309) — Cost-per-inference analysis
> - Slack #smart-photos-v4: backfill ramp, code freeze discussion, DE errors, country expansion, Kafka lag
> - Slack #ml-team-mlinfra: DynamoDB caching deployment (Ankush, Dec 2025)
> - Slack #mgai-aura: GPU cost context, Trideep on SmartPhoto v4 infra
> - Source code: `revive.mlinfra/mlplatform/decisionengine/ranking/smart_photos_ranking/` (Go) — ranking logic, LLM calls, prompt, preprocessing, traffic split
> - Source code: `revive.discovery/worker/smartphoto/` and `worker/smartphoto-backfill/` (Java) — workers, orchestrator

**Changes from v2:**
- Added rationale for pairwise comparison approach to finding 2.2 (Ben Shin, Slack #core-growth-engagement)
- Added ranking aggregation algorithm detail to finding 2.2 (win-count + probability-weighted scoring from ranker.go)
- Added Predibase→vLLM migration context to finding 2.8 (Ankush comparison, Feb 2026)
- Added finding 2.13: serving path (mediamodification service applies final ordering with face/verified heuristics)
- Added finding 2.14: tech blog published Apr 8, 2026
- Added Evidence 3.13 (ranking algorithm) and 3.14 (serving path)
- Added section 5: Unverified Items

**Changes from v1:**
- Added findings 2.8-2.12 from source code and worker runbook
- Added Evidence 3.9-3.12
- Updated Reference with full file inventory from GitHub

---

## 1. Conclusion

**Q: What is the full engineering picture of SmartPhoto v4 — architecture, ML pipeline, infrastructure, key decisions, and team roles?**

SmartPhoto v4 replaces the 2020-era EfficientNet image ranker (v3) with a VLM-based pairwise photo comparator built on Qwen2.5-VL 7B. The system has four main engineering components: (1) two Kafka-driven Java workers that detect photo changes and trigger ranking, (2) a Go Decision Engine that orchestrates pairwise comparisons with DynamoDB caching and profile-aware prompting, (3) a dual serving backend (Predibase hosted + self-hosted vLLM) with etcd-controlled traffic split, and (4) a Phoenix-based experimentation layer. V4 ranking currently applies to male users only. The project was built across two repos (`revive.discovery` for workers, `revive.mlinfra` for ML infra), involved 6 engineers across 4 functions, and rolled out globally between Dec 2025 and Mar 2026.

---

## 2. Key Findings

### 2.1. The system has four core components with clear ownership boundaries

The architecture splits into workers (Java, Gabriela), Decision Engine + vLLM (Go/Python, Ankush + Trideep/Skanda), and experimentation (Phoenix, Kevin). See Evidence 3.1 for the full data flow.

### 2.2. The ML model is a Qwen2.5-VL 7B VLM doing pairwise photo comparison

Instead of scoring each photo independently (v3 approach), v4 receives image pairs and returns which is the better Tinder primary photo. Ben Shin explained the rationale: "there's a reason why we chose the pairwise comparison approach from ML perspective" — pairwise comparison has a well-established connection to rSRR prediction accuracy (Slack #core-growth-engagement, Feb 2026). The model runs on vLLM with prefix caching enabled and a custom Qwen2-VL chat template. See Evidence 3.2 for performance numbers.

Results are aggregated using a win-count + probability-weighted scoring algorithm: each photo accumulates wins and logprob-derived scores across all its pairwise comparisons, then photos are sorted by win count (descending), with average score as tiebreaker. See Evidence 3.13.

### 2.3. DynamoDB caching was the key cost optimization

Before caching, every photo event recomputed all pairwise scores for a user's photos (up to 9 photos = 36 pairs). Ankush added a DynamoDB caching layer (Dec 2025) that reuses previously computed pairwise scores when only one photo changes, yielding 30-40% GPU savings (~$20K/month). See Evidence 3.3 for the caching design.

### 2.4. Backfill was the critical-path prerequisite for experiment launch

V4 cannot run an experiment until enough users have precomputed v4 scores. The backfill worker consumed `user-active-time-updates-with-photos` from Kafka and processed existing users over a 2-4 week period. The team ran backfill during the Dec 2025 holiday code freeze to start the experiment in Jan 2026. At 100% backfill, the system ran at ~100 LLM QPS on 70 GPUs (capacity: ~350 QPS). See Evidence 3.4.

### 2.5. Country-based progressive rollout was used for global expansion

Base countries (US, CA, VN, MX, GB) were unconditionally allowed. Two expansion waves added 26 more countries behind independent etcd flags: expansion 1 (AR, AU, CL, PH, TW, VE, DZ, AE, BO, GT) and expansion 2 (IN, NZ, QA, KH, HN, LB, IQ, MN, NP, UG, NI, TT, SN, MU, MD, MM). Each wave could be ramped independently. See Evidence 3.5.

### 2.6. The experiment tested four variants on the swipee side

Control (V3), Variant 1 (V4), Variant 2 (V4 + Shirtless downranking), and Variant 3 (User ordering). A separate swiper-side test added a V4 + Faces variant. The shirtless heuristic uses an existing T&S model to identify and deprioritize the highest-ranked shirtless photo. See Evidence 3.6.

### 2.7. Two significant operational incidents occurred during rollout

(1) Feb 2026: Decision Engine failure rate spiked to ~43%, requiring backfill and additional country traffic to be shut off. (2) Apr 2026: Kafka consumer lag reached 1.38M messages with a growing gap of ~169 msgs/sec, prompting partition increase to 38 for the smartphoto worker. See Evidence 3.7.

### 2.8. The serving infrastructure migrated from Predibase to self-hosted vLLM

The system originally ran on Predibase (managed LLM hosting). In Feb 2026, Ankush ran a controlled comparison on the same 41K eval dataset with identical hardware (2 GPUs, 20 concurrent). vLLM showed 76.26% accuracy vs Predibase 75.06%, and 8.50 rps vs 7.66 rps throughput. Trideep noted the accuracy difference was not meaningful for a dedicated experiment, but the throughput improvement and cost control justified migration. The traffic split is controlled by etcd flag (`SMART_PHOTOS_VLLM_TRAFFIC_RATE`, 0-100%), allowing gradual migration. See Evidence 3.10.

### 2.9. Prompts include profile features for personalized ranking

The LLM prompt includes user profile context — age (bucketed), country, match gender preference, and relationship goal — so the model can judge photos relative to the user's audience. Cache is invalidated when target genders or relationship intent change. See Evidence 3.11.

### 2.10. V4 ranking is male-only

The Decision Engine processor is only called for male users (gender 0). Female users continue on V3 (ML Image Ranking with separate male/female EfficientNet models). The worker runbook states: "DecisionEngineProcessor is only called for male users when enabled."

### 2.11. The worker uses a four-stage orchestrator pipeline

Both workers share a `smartphoto-orchestrator` library with four processors in sequence: MediaUserReader → MLImageRankerProcessor → DecisionEngineProcessor → MediaUserUpdater. See Evidence 3.12.

### 2.12. Ranked photo ordering is served through the mediamodification service

The Decision Engine computes and caches rankings, but the final photo order shown to users goes through a separate serving path. The `mediamodification` service (owned by SOUP/Browse) reads `smartphoto_rank["v4"].rank` from the media-users DynamoDB table and applies additional heuristics: verified photos first, then single-face photos, then multi-face photos. Different experiment cohorts can be served different orderings. As of Mar 2026, there is no endpoint that returns the exact served order for a given user — this was flagged as a gap by both Recs Viewer and analytics teams. See Evidence 3.14.

### 2.13. A tech blog was published in Apr 2026

Trideep published a tech blog on Apr 8, 2026: "An inside look at how Tinder's VLM-based system reframes photo ranking through pairwise comparisons to improve matches, likes, and meaningful connections." (Slack #smart-photos-v4)

### 2.14. Team structure

Six engineers across four functions worked on v4. See Evidence 3.8 for the full roster and role mapping.

---

## 3. Evidence / Details

### 3.1. System data flow

```mermaid
flowchart LR
    subgraph Kafka Topics
        T1[media-updates]
        T2[user-active-time-updates-with-photos]
    end

    subgraph SmartPhoto Worker
        direction LR
        MUR[MediaUserReader]
        MLR[MLImageRankerProcessor]
        DEP[DecisionEngineProcessor]
        MUU[MediaUserUpdater]
    end

    subgraph External Services
        MDC[(MediaDataClient)]
        MLC_M[MLImageRankingClient\nglobalMaleClient]
        MLC_F[MLImageRankingClient\nglobalFemaleClient]
        DEC[DecisionEngineClient]
    end

    T1 --> MUR
    T2 --> MUR
    MUR <--> MDC
    MUR --> MLR
    MLR <--> MLC_M
    MLR <--> MLC_F
    MLR --> DEP
    DEP <--> DEC
    DEP --> MUU
    MUU --> MDC
```

Within the Decision Engine, the ranking flow for each request:

1. DynamoDB cache fetch and S3 image downloads run **in parallel** (DynamoDB has a 3s timeout — if exceeded, cache is skipped gracefully)
2. Profile features are preprocessed while waiting
3. For each model ID (processed sequentially): determine cached vs uncached pairs, run uncached pairs through LLM **in parallel**, combine results, run Ranker aggregation
4. Save updated cache to DynamoDB asynchronously (5s timeout, background context)
5. Return ranked photo ordering

### 3.2. vLLM service specifications

| Parameter | Value |
|-----------|-------|
| Model | Qwen2.5-VL 7B (fine-tuned) |
| Serving engine | vLLM with prefix caching |
| Tensor parallel size | 2 GPUs per replica |
| GPU type | A10G (or equivalent) |
| Prod pods | 12-59 (scaled over time) |
| Total GPUs in production | Up to 70 |
| LLM QPS capacity | ~350 |
| Accuracy | 76.26% |
| Throughput | 8.50 req/s per replica (2 GPUs, 20 concurrent) |
| Avg latency | 940 ms |
| Chat template | Custom Qwen2-VL jinja template |
| Model weights path | /spotlight/smart_photos_v4_merged |

### 3.3. DynamoDB caching design

- Table: `smartphotoranking`
- Key: user-level; stores pairwise photo scores
- Max photos per user: 9 (max 36 pairwise comparisons)
- On photo event: read existing scores, recompute only new/changed pairs, write back
- GPU savings: 30-40% reduction in LLM calls
- Cost savings: ~$20K/month in GPU costs
- Deployed during Dec 2025 code freeze with exception approval

### 3.4. Backfill timeline

| Date | Event |
|------|-------|
| Dec 19, 2025 | Code freeze exception approved for backfill levers |
| Dec 23, 2025 | DynamoDB caching layer deployed |
| Jan 5, 2026 | Backfill at ~45% capacity, ramping to 100% |
| Jan 5, 2026 | LLM QPS at ~40, stable on 70 GPUs |
| Jan 21, 2026 | Target experiment start (after backfill complete) |
| Feb 10, 2026 | DE failure spike, backfill paused |
| Mar 2026 | Global rollout in progress |

### 3.5. Rollout etcd flags

| Flag | Purpose |
|------|---------|
| smartphoto_decision_engine_enabled | Master switch for v4 ranking |
| smartphoto_decision_engine_country_filter_enabled | Restrict v4 to allowed countries |
| smartphoto_worker_backfill_active_time_process_enabled | Enable backfill worker |
| smartphoto_worker_backfill_country_filter_enabled | Country filter for backfill |
| smartphoto_decision_engine_country_filter_photo_touchup_expansion | Expansion wave 1 (10 countries) |
| smartphoto_decision_engine_country_filter_photo_touchup_expansion_2 | Expansion wave 2 (16 countries) |

### 3.6. Experiment variants

Swipee-side test:

| Variant | Description |
|---------|-------------|
| Control | Smart Photo V3 (EfficientNet, no face prioritization) |
| Variant 1 | Smart Photo V4 (VLM ranking) |
| Variant 2 | V4 + Shirtless (VLM + T&S shirtless downranking) |
| Variant 3 | User ordering (user-set photo order) |

Swiper-side test (additional):

| Variant | Description |
|---------|-------------|
| Variant 4 | V4 + Faces (VLM + T&S face detection prioritization) |

Shirtless heuristic: order photos by v4 score, then swap the first non-shirtless photo into slot 1.

### 3.7. Incidents

**Feb 10, 2026 — Decision Engine failure spike**
- Failure rate: ~43% of DE requests
- Cause: cluster update on Predibase, near-zero LLM requests getting through
- Mitigation: turned off backfill worker and additional country processing
- Impact: lost scores for unprocessed users (no retry logic at the time)

**Apr 12, 2026 — Kafka consumer lag**
- Lag: 1.38M messages, growing ~100 msgs/sec
- Gap: producer outpacing consumer by ~169 msgs/sec
- Downstream: smartphotosvllm healthy (376/sec success, 0.05% error), 59 pods
- Root cause: higher-than-expected photo update volume
- Mitigation: increase partitions to 38

### 3.8. Team roster

| Role | Name | Responsibilities |
|------|------|-----------------|
| PM | Aaron Silvers-Lamas | Requirements, experiment design |
| EM | Xincen Hao | Engineering management, incident response |
| EM | Ryan Burns | Engineering management |
| Backend | Gabriela Rodriguez-Florido | Workers (online + backfill), Kafka, country rollout |
| Backend | Jason Chang | Backend support |
| ML | Skanda Suresh | V4 model, experiment design, traffic control |
| ML | Trideep Rath | V4 model, LLM service monitoring, cost optimization |
| ML Infra | Ankush Ransiwal | Decision Engine, vLLM serving, DynamoDB caching, ranking pipeline |
| Analytics | Kevin Celustka | Metrics, analytics rollups, experiment analysis |

### 3.9. Prompt design

System message:
> You are a top-tier expert in evaluating Tinder profile photos, with deep knowledge of dating app culture, user psychology, human behavior, and image quality. Your task is to decide which photo would perform better as a primary profile picture on Tinder, based on visual appeal and social dynamics.

User message: two images + instruction "Respond with either 'first' or 'second' depending on which one is better as a Tinder primary photo." + optional profile context.

LLM parameters: temperature=0.0, max_tokens=100, logprobs=true. Response is parsed for "first" or "second" prefix; anything else is an error.

Image selection: `PickMediaURL` selects the URL closest to 640x800 resolution from available sizes. Images are downloaded from S3 or HTTP, validated (magic bytes check for JPEG/PNG/GIF/WebP), and base64-encoded.

### 3.10. Traffic split mechanism

| Config | Source | Purpose |
|--------|--------|---------|
| SMART_PHOTOS_VLLM_TRAFFIC_RATE | etcd | Percentage of traffic routed to vLLM (0-100) |
| VLLM_TRAFFIC_PERCENT | config JSON | Fallback if etcd unavailable |
| VLLM_SMART_PHOTOS_URL | config | Default: `http://smartphotosvllm.svc.tinder.local/v1/chat/completions` |
| VLLM_SMART_PHOTOS_MODEL | config | Default: `/spotlight/smart_photos_v4_merged` |
| PREDIBASE_SMART_PHOTOS_ADAPTER | config | Default: `Smart_Photos/2` |

When rate is between 0 and 100, each request is randomly sampled. Both backends use the same prompt structure but different client libraries (`vllmclient` vs `llmclient`).

### 3.11. Profile preprocessing features

| Feature | Source field | Processing | Example output |
|---------|-------------|------------|----------------|
| age_years | ProfileInfo.Age | Clipped to max 75, "Unknown" if empty | "28" |
| age_bucket | Derived from age_years | 18-25, 25-35, 35-50, 50-75, 75+ | "25-35" |
| country | ProfileInfo.Country | 3-letter code → full name via lookup table | "United States of America" |
| match_gender_preference | ProfileInfo.TargetGenders | Sorted combo mapped to string | "Women" |
| relationship_goal | ProfileInfo.RelationshipIntent | descriptor_choice_id → label | "Long-term partner" |

Cache invalidation: if target genders or relationship intent differ from the cached record, all pairwise scores for that user+model are recomputed.

`SMART_PHOTOS_IGNORE_RELATIONSHIP_INTENT` flag: when set to 1, relationship intent is always "Not Specified" for cache consistency during migration.

### 3.12. Orchestrator pipeline

| Stage | Class | Purpose |
|-------|-------|---------|
| 1 | MediaUserReader | Read photo data from Media Data Service |
| 2 | MLImageRankerProcessor | Call ML Image Ranking (v3, separate male/female EfficientNet models) |
| 3 | DecisionEngineProcessor | Call Decision Engine for v4 ranking (male users only, when enabled) |
| 4 | MediaUserUpdater | Write updated rankings to Media Data Service |

Shared library: `libraries/smartphoto-orchestrator/` in `revive.discovery`. Both workers depend on this.

### 3.13. Ranking aggregation algorithm (ranker.go)

For each photo, the ranker tracks three values across all its pairwise comparisons:

| Metric | How it accumulates |
|--------|--------------------|
| win_count | +1 for each comparison won |
| total_score | Winner gets +probability (from logprobs); loser gets +(1 - probability) |
| comparison_count | +1 for each comparison participated in |

Sorting: primary by win_count descending, secondary by average_score (total_score / comparison_count) descending.

This is a simple Copeland-like method enhanced with logprob confidence. It does not use Bradley-Terry or Elo. All pairwise comparisons must succeed for a valid ranking — partial failures cause the entire model request to fail (but partial results are still cached for faster retry).

### 3.14. Serving path

The Decision Engine writes ranked photo scores to the `media-users` DynamoDB table under `smartphoto_rank["v4"].rank`. The final photo order shown to users is determined at serving time by the `mediamodification` service (in `revive.recs` or `revive.content`), which:

1. Reads the stored v4 ranking (for male users in v4 experiment cohort)
2. Applies rule-based heuristics: verified photo first > single-face photo > multi-face photo > smart photo rank
3. Returns the final ordering to recs/profile for display

Different experiment cohorts (v3, v4, v4+shirtless, user ordering) are served different orderings at this layer. As of Mar 2026, there is no API endpoint that returns the exact served order for a given user — Recs Viewer implemented a stop-gap using `smartphoto_rank["v4"].rank` directly (REC-5388).

---

## 5. Unverified Items

The following items could not be verified due to access restrictions or insufficient evidence:

| Item | Reason |
|------|--------|
| SmartphotosV4 ML System Design (GDrive, Ankush) | Permission denied — not accessible via Glean or direct GDrive |
| SmartPhotosV4 spec (GDrive, Trideep) | Permission denied |
| SmartPhotosV4 original doc (GDrive, Trideep) | Permission denied |
| V4 model fine-tuning details (training data, method, dataset size) | Not found in any accessible source; likely in the GDrive docs above |
| Current Predibase→vLLM traffic split ratio | Not visible from code (etcd runtime value); Feb 2026 comparison suggests migration in progress |
| Experiment outcome metrics (rSRR lifts, revenue impact) | Partial: Slack #ml-shoutout mentions +0.4% total cash lift vs V3, ~$2.6M annualized extrapolation. Full analysis not found |
| V5 scope and relationship to V4 | Mentioned in Slack (Ben: "a teaser of SmartPhoto v5", Mar 2026) and a GDrive doc "Smart Photos V5: ML Proposal" exists but was not read |
| Tech blog URL | Trideep posted "our tech blog is live" (Apr 8, 2026) but the URL was not included in the Slack message |

---

## 4. Reference

### Repositories

| Repo | Language | Components |
|------|----------|------------|
| TinderBackend/revive.discovery | Java | worker/smartphoto (online), worker/smartphoto-backfill, libraries/smartphoto-orchestrator, libraries/smartphoto-consumer-lib |
| TinderBackend/revive.mlinfra | Go, Python | mlplatform/decisionengine (Go), service/smartphotosvllm (Python/vLLM), docker/smartphotosvllm |
| TinderBackend/scaffold | YAML | service/revive.mlinfra/smartphotosvllm (deployment config) |

### Decision Engine ranking module files

| File | Size | Purpose |
|------|------|---------|
| ranking/smart_photos_ranking/smartphotosranking.go | 20.9KB | Main runner: parallel DynamoDB+S3 fetch, cache logic, per-model sequential processing |
| ranking/smart_photos_ranking/call_llm.go | 11.5KB | Predibase LLM client, image download/encode, format detection, URL selection |
| ranking/smart_photos_ranking/preprocessor.go | 10.4KB | Profile feature extraction (age, country, gender pref, relationship intent) |
| ranking/smart_photos_ranking/traffic_split.go | 7.7KB | Predibase/vLLM traffic routing, vLLM client, etcd flag reader |
| ranking/smart_photos_ranking/metrics.go | 6.0KB | Prometheus metric definitions |
| ranking/smart_photos_ranking/ranker.go | 3.5KB | Pairwise results → final ranking aggregation |
| ranking/smart_photos_ranking/generate_prompt.go | 3.1KB | Chat completion request construction with Qwen2-VL template |
| ranking/smart_photos_ranking/logging.go | 2.7KB | Structured event logging |
| rpci/decision_engine.go | - | RPC interface, DynamoDB client init |
| rpci/smart_photos_ranking.go | - | gRPC endpoint for SmartPhotos ranking |

### Monitoring

| Dashboard | URL pattern |
|-----------|------------|
| smartphotosvllm service (SD) | grafana.ue1az.tinderops.net/d/.../smartphotosvllm |
| Revive worker metrics | grafana.ue1az.tinderops.net/d/v31otsh4k/smart-photo-revive-worker |
| Backfill worker metrics | grafana.ue1az.tinderops.net/d/v31otsh4k1/smart-photo-backfill-worker |
| Revive worker standard | grafana.ue1az.tinderops.net/d/.../smartphotoreviveworker |
| Backfill worker standard | grafana.ue1az.tinderops.net/d/.../smartphotobackfillworker |
| Predibase LLM monitoring | predibase dashboard (LLM QPS, GPU utilization) |

### Alerts

- QPS 200s: missing successful requests
- QPS 500s: error rate threshold
- Latency: per-endpoint, triggered at ~2x timeout (~60s)
- Container restarts: threshold-based
- Alert channel: #ml-infra-alert
- Oncall: mlinfra escalation path on incident.io

### Key Prometheus metrics (Decision Engine)

| Metric | Purpose |
|--------|---------|
| de_smart_photos_ranking_latency | End-to-end ranking latency |
| de_smart_photos_cache_hits / cache_misses | DynamoDB cache effectiveness |
| de_smart_photos_llm_calls_saved | LLM calls avoided by caching |
| de_smart_photos_traffic_routing_total | Predibase vs vLLM routing decisions |
| de_smart_photos_vllm_inference_latency_seconds | vLLM call latency |
