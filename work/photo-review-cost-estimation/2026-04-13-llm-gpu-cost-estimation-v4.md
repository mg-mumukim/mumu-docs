# Photo Review — LLM + GPU Cost Estimation

> **Sources**:
> - `work/photo-review-cost-estimation/2026-04-10-llm-cost-estimation-v2.md`
> - `work/photo-review-cost-estimation/2026-04-13-gpu-cost-estimation-report-v1.md`
> - `note/source/2026-04-10-profle-ranker-h100-benchmark.md`
> - `note/source/2026-02-03-crs-v3-pii-redaction-cost-estimation.md`
> - `note/source/2026-02-20-crs-v3-camerarollphotosuggestion-cost-estimation.md`

---

## TL;DR

Photo Review runs a two-stage pipeline per request: (1) self-hosted GPU inference for concept prediction (e.g., Qwen3-VL-8B on L40S), then (2) LLM API call to generate coaching output based on the concepts predicted (e.g., Gemini 3.1 Flash-Lite). Both costs are additive.

GPU cost depends on the serving mode: real-time requires an always-on 4-pod HA cluster ($4.1K/month), while batch only pays for GPU-hours needed to process the daily batch within 1 hour ($38-247/month).

| Scope | Serving Mode | API Cost | GPU Cost | Total | Within $50K? |
|-------|-------------|--------:|---------:|------:|:------------:|
| Test regions (USA+CAN+AUS) | Batch (GPU on-demand) | $33K | $50 | $33K | Yes |
| Test regions | Batch (GPU CSP) | $33K | $38 | $33K | Yes |
| Test regions | Real-time (GPU HA) | $66K | $4K | $70K | No |
| Global | Batch (GPU on-demand) | $163K | $247 | $163K | No |
| Global | Batch (GPU CSP) | $163K | $188 | $163K | No |
| Global | Real-time (GPU HA) | $325K | $4K | $329K | No |

Only test-region + batch fits within $50K/month. In batch mode, GPU cost is negligible compared to LLM API cost.

---

## 1. LLM API Cost per Request

Baseline model: Gemini 3.1 Flash-Lite Preview (`gemini/gemini-3.1-flash-lite-preview`).

| #Requests | Cost |
|----------:|-----:|
| 1 | $0.0086 |
| 1M | $8,600 |

This per-request cost is an all-in number that accounts for the typical token usage of a Photo Review call (system prompt, user profile text, multi-image vision tokens, and output coaching text). See the [AURA Cost Dashboard](https://ai-demo.dev.matchgroupcentral.net/aurademo/demo/cost-dashboard) for the detailed breakdown of input/output token parameters or other models.

> Reference: [Gemini 3.1 Flash-Lite: Built for intelligence at scale](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-1-flash-lite/)

## 2. GPU Self-Hosted Cost

Baseline model: Qwen3-VL-8B SFT LoRA on NVIDIA L40S (g6e.xlarge, us-east-1).

### Per-Node Pricing

| Pricing Model | Rate ($/hr) | Monthly (1 pod, 24/7) |
|---------------|------------:|----------------------:|
| On-Demand | $1.861 | $1,340 |
| CSP (24% discount) | $1.414 | $1,018 |
| 3yr RI (hypothetical) | $0.804 | $579 |

Tinder uses Compute Savings Plan (CSP), not Reserved Instances. The dashboard defaults to 3yr RI, which is not a pricing option Tinder has purchased.

### Three GPU Billing Modes

GPU cost structure differs depending on the serving mode:

**Mode A: Always-on HA (real-time serving)**

Real-time serving requires the GPU cluster to be available for immediate inference when a user opens the app. Production minimum: 4 pods (1 pod/AZ x 4 AZs) running 24/7 per Jessica Hickey's guidance.

Monthly cost = 4 x $1.414/hr x 24h x 30d = $4,073 (CSP)

Note: the gpu-cost-estimation-report-v1 states $3,870/month ($129/day), but 4 x $1.861 x 24 x 0.76 = $135.78/day = $4,073/month. The report's figure implies ~28% discount rather than the stated 24%. This document uses the arithmetic-consistent figure of $4,073.

**Mode B: On-demand burst (batch serving)**

Batch serving collects eligible users and processes them in a scheduled GPU batch. GPUs are spun up, process the batch within 1 hour, and spun down. No HA minimum required.

GPUs needed = ceil(daily_calls / 284,400)

| Scope | Daily Calls | GPUs Needed | GPU-Hours/Day | Monthly GPU-Hours | Monthly Cost (On-Demand) |
|-------|------------:|------------:|--------------:|------------------:|-------------------------:|
| Test regions | 255.5K | 1 | 0.9 | 27 | $50 |
| Global | 1.26M | 5 | 4.4 | 133 | $247 |

**Mode C: Savings Plan burst (batch serving)**

Same burst pattern as Mode B but with CSP discount (24% off on-demand).

| Scope | Monthly GPU-Hours | Monthly Cost (CSP) |
|-------|------------------:|-------------------:|
| Test regions | 27 | $38 |
| Global | 133 | $188 |

### Throughput

Throughput per GPU: 79 users/sec (full-forward) or ~94 users/sec (production-truncated, trunc=19). One GPU processes 284,400 users/hr (full-forward).

For always-on mode, each pod operates independently in its AZ, so total cluster throughput is 4x single-pod throughput. The 4-pod cluster handles up to 27.3M users/day — well above global steady-state (1.26M/day). Cross-AZ latency does not affect throughput because requests are routed to the local AZ pod.

For batch mode, multiple GPUs run in parallel. 5 GPUs process 1.26M users in ~53 minutes; 1 GPU processes 255.5K users in ~54 minutes.

## 3. Traffic Quantities

### 3.1 DAU Baseline

| Country | DAU |
|---------|----:|
| Global | 18M |
| USA | 2.8M |
| CAN | 450K |
| AUS | 400K |
| English regions total (USA+CAN+AUS) | 3.65M |

Source: databricks `published_prod.analytics_rollups.app_open_first_last_uid_day`

### 3.2 When Does a User Trigger a Call?

A user triggers exactly one Photo Review call per day, and only if one of the following conditions is met:

1. **First login after launch (cold start)** — The user has never received a Photo Review.
2. **Profile change** — After their initial score, a returning user triggers a new call only when they change their profile (add/delete a photo, change bio, change prompt, add/delete an interest or descriptor).

If a user opens the app but has already been scored and has not changed their profile, no call is made. Maximum 1 call per user per day.

### 3.3 Step-by-Step: How Many Calls per Day?

**Step 1 — Day 1 (Cold Start)**

On the very first day Photo Review is turned on, every user who opens the app triggers one call.

- Day-1 calls = DAU = 18M (global)

**Step 2 — Days 2 through ~21 (Ramp-Down)**

Starting on day 2, users already scored on day 1 do NOT trigger a new call (unless they changed their profile). Only two groups generate calls:

- Newly returning users who missed day 1 and open the app for the first time since launch.
- Profile changers who update their photos, bio, prompts, or interests.

Each day, the pool of "never-scored" users shrinks. Kevin's analysis shows this ramp-down takes about 3 weeks to reach steady state.

**Step 3 — Steady State (After ~3 Weeks)**

| Source | Rate (% of DAU) | Rationale |
|--------|----------------:|-----------|
| New/returning logins (first-time scores) | ~3% | New registrations + reactivated users who haven't been scored |
| Profile changes | ~4% | Kevin's sample of 1,000 users: ~4% make any profile change on a given day |
| Total steady-state rate | ~7% | Sum of above (with minor overlap, making this a slight overestimate) |

> Note on the 4% figure: Ben's notebook estimates 8.5% of DAU change photos daily, which is higher than Kevin's 4% for all profile changes combined. The two measurements likely differ in methodology or scope. In this document, we use Kevin's figure.

**Step 4 — Calculate Daily Calls at Steady State**

Steady-state daily calls = DAU x 7%

| Scope | DAU | Daily Calls |
|-------|----:|------------:|
| Global | 18M | 1.26M |
| USA | 2.8M | 196K |
| CAN | 450K | 31.5K |
| AUS | 400K | 28K |
| Test regions (USA+CAN+AUS) | 3.65M | 255.5K |

**Step 5 — Calculate Monthly Calls**

Monthly calls = Daily calls x 30

| Scope | Daily Calls | Monthly Calls |
|-------|------------:|--------------:|
| Global | 1.26M | 37.8M |
| Test regions | 255.5K | 7.67M |

## 4. LLM API Serving Scenarios

Two serving options for the LLM API stage. They differ in when the call happens and how much it costs. Both options require the GPU stage to run first.

### Option 1: Real-Time Review Generation

The LLM is called when a user opens the app (if they are eligible). Feedback is generated and shown on the same day.

- Latency: ~1 minute per generation.
- API pricing: Standard (non-batch) rate.
- Cost per request: $0.0086
- GPU mode: Always-on HA (Mode A)

### Option 2: Batched (24-Hour) Review Generation

Eligible users are collected throughout the day. GPU batch processes concept predictions within 1 hour. Results are then submitted to the Gemini Batch API. Final coaching output is stored and served from cache.

- Latency: Up to 24 hours delay.
- API pricing: Gemini Batch API at 50% discount.
- Cost per request: $0.0086 x 50% = $0.0043
- GPU mode: On-demand burst (Mode B) or Savings Plan burst (Mode C)

### LLM API Cost Summary

#### Day-1 (Cold Start) — One-Time Cost

| Scope | Calls | Real-Time Cost | Batch Cost |
|-------|------:|--------------:|-----------:|
| Global | 18M | $154.8K | $77.4K |
| Test regions | 3.65M | $31.4K | $15.7K |

#### Steady-State — Monthly Cost

| Scope | Monthly Calls | Real-Time ($/month) | Batch ($/month) |
|-------|-------------:|--------------------:|----------------:|
| Global | 37.8M | $325K | $163K |
| Test regions | 7.67M | $66K | $33K |

## 5. GPU Serving Scenarios

The GPU stage runs concept prediction before the LLM API call. Cost depends on the serving mode.

### Real-Time (Mode A: Always-on HA)

4-pod HA cluster running 24/7. Required for immediate inference when a user opens the app.

| Component | Monthly Cost (CSP) |
|-----------|-------------------:|
| 4x g6e.xlarge (24/7) | $4,073 |

### Batch (Mode B: On-Demand Burst)

GPUs spin up, process the daily batch within 1 hour, spin down. Pay on-demand rate for actual GPU-hours only.

| Scope | GPUs | GPU-Hours/Day | Monthly Cost (On-Demand) |
|-------|-----:|--------------:|-------------------------:|
| Test regions | 1 | 0.9 | $50 |
| Global | 5 | 4.4 | $247 |

### Batch (Mode C: Savings Plan Burst)

Same burst pattern as Mode B with CSP discount (24% off on-demand).

| Scope | Monthly GPU-Hours | Monthly Cost (CSP) |
|-------|------------------:|-------------------:|
| Test regions | 27 | $38 |
| Global | 133 | $188 |

### Day-1 Backfill GPU Cost

| Scope | Users | GPUs Needed (1h) | GPU-Hours | On-Demand | CSP |
|-------|------:|------------------:|----------:|----------:|----:|
| Global | 18M | 64 | 63.3 | $118 | $90 |
| Test regions | 3.65M | 13 | 12.8 | $24 | $18 |

For always-on mode (real-time), backfill is absorbed by the existing 4-pod cluster (takes ~16h with 4 GPUs for global). For batch mode, spin up the required GPUs for a one-time burst.

### CRS Production Comparison

| Service | Architecture | Pods | Instance | Monthly (after CSP) |
|---------|-------------|------|----------|--------------------:|
| CRS PII Redaction | 3 microservices | 4 CPU + 8 GPU | g5.2xlarge | $6,270 |
| CRS Photo Suggestion | Single service | 4 GPU | g5.2xlarge | $2,760 |
| AURA (always-on estimate) | Single service | 4 GPU | g6e.xlarge | $4,073 |

## 6. Combined Cost Summary

Total monthly cost = LLM API cost + GPU cost.

### Steady-State — Monthly Total

| Scope | Serving Mode | API Cost | GPU Cost | Total | Within $50K? |
|-------|-------------|--------:|---------:|------:|:------------:|
| Test regions | Batch + GPU on-demand | $33K | $50 | $33.1K | Yes |
| Test regions | Batch + GPU CSP | $33K | $38 | $33K | Yes |
| Test regions | Real-time + GPU HA | $66K | $4.1K | $70.1K | No |
| Global | Batch + GPU on-demand | $163K | $247 | $163.2K | No |
| Global | Batch + GPU CSP | $163K | $188 | $163.2K | No |
| Global | Real-time + GPU HA | $325K | $4.1K | $329.1K | No |

In batch mode, GPU cost is less than 1% of the total — LLM API dominates. In real-time mode, GPU adds ~$4.1K fixed overhead.

### Day-1 (Cold Start) — One-Time Total

| Scope | Serving Mode | API Cost | GPU Cost | Total |
|-------|-------------|--------:|---------:|------:|
| Test regions | Batch + GPU on-demand | $15.7K | $24 | $15.7K |
| Test regions | Real-time + GPU HA | $31.4K | $0 (absorbed) | $31.4K |
| Global | Batch + GPU on-demand | $77.4K | $118 | $77.5K |
| Global | Real-time + GPU HA | $154.8K | $0 (absorbed) | $154.8K |

## 7. Assumptions

### 7.1 Maximum One Call per User per Day

Each user triggers at most 1 call per day, regardless of how many times they open the app or change their profile.

- Value: Max 1 call/user/day
- Source: Kevin's traffic model and Ben's recommendation ("Batch per user, not per app open")

### 7.2 Profile Change Rate

~4% of DAU makes at least one profile change per day. A "profile change" includes: add/delete photo, change bio, change prompt, add/delete interest, add/delete descriptor.

- Value: 4% of DAU/day
- Source: Kevin's analysis of a 1,000-user sample
- Note: Ben's notebook reports ~8.5% for photo changes specifically. We use Kevin's figure per team convention.

### 7.3 Steady-State Rate

Total steady-state trigger rate is ~7% of DAU/day, combining ~3% new/returning logins and ~4% profile changes.

- Value: 7% of DAU/day
- Source: Kevin's ramp model ("stable state would be 7%")
- Note: Minor overlap (a new login who also changes their profile) makes 7% a slight overestimate.

### 7.4 Cold-Start Ramp-Down

After launch, it takes approximately 3 weeks for daily call volume to ramp down from 100% of DAU to 7% steady state.

- Value: ~21 days ramp-down
- Source: Kevin's Mode report ("take about 3 weeks to reach a steady state of 7%")

### 7.5 Batch API Discount

Gemini's Batch API offers a 50% discount on per-token pricing compared to standard (real-time) API calls.

- Value: 50% cost reduction
- Source: Gemini API pricing documentation

### 7.6 GPU Batch Window: 1 Hour

For batch serving, the GPU stage must complete processing within 1 hour. The number of GPUs needed is calculated as ceil(daily_calls / 284,400) where 284,400 = 79 users/sec x 3,600 sec.

- Value: 1-hour batch window
- Source: Planning assumption (not a system constraint)

### 7.7 GPU Pricing: CSP, Not RI

Tinder uses Compute Savings Plan for GPU instances. CSP discount for g6e is ~24% off on-demand. The aura-demo cost dashboard defaults to 3yr RI ($0.804/hr, 57% off), but Tinder has not purchased GPU RI. Using RI pricing would understate GPU cost by 1.76x.

- Value: $1.414/hr per g6e.xlarge (after 24% CSP)
- Source: Jessica Hickey (Slack #cloud-infra-operations, 2025-12-30)
- Note: gpu-cost-estimation-report-v1 states $129/day ($3,870/month) for 4-pod HA, but the arithmetic with 24% CSP yields $135.78/day ($4,073/month). The report's figure implies ~28% discount. This document uses the 24%-consistent figure.

### 7.8 GPU HA Minimum: 4 Pods, 24/7 (Real-Time Only)

Real-time GPU serving requires 1 pod per AZ x 4 AZs = 4 pods minimum, running 24/7. This does not apply to batch mode, where GPUs can be spun up on-demand.

- Value: 4 pods, 24/7 (real-time); variable (batch)
- Source: CRS PII Redaction and CRS Photo Suggestion deployment patterns; Jessica Hickey guidance

### 7.9 GPU Throughput: H100 to L40S Extrapolation

L40S throughput is estimated by dividing H100 measured throughput by 3.0x blended factor (compute-bound 2.73x + memory-bound 3.88x midpoint). Direct L40S measurement has not been performed.

- Value: 3.0x scaling factor
- Source: H100 989 TFLOPS / L40S 362 TFLOPS (compute), H100 3.35 TB/s / L40S 864 GB/s (memory)

### 7.10 Dashboard Backfill Ignores Market Coverage

`GpuSection.tsx:20` always uses raw DAU (18M) for backfill, even when "Experiment US 5%" is selected (should be ~900K). This overstates backfill cost for scoped experiments.

- Source: `matchgroup-ai/aura-demo` codebase

### 7.11 GPU vs API Quality Difference Not Modeled

The dashboard's "Nx cheaper" GPU-vs-API comparison compares an 8B fine-tuned model (88.2% accuracy) against frontier API models without accounting for the quality gap. This document does not model quality trade-offs.

- Source: `shared.ts` SELF_HOSTED.accuracy = 88.2%

### 7.12 Infrastructure Overhead Not Included

GPU cost figures do not include EBS storage, data transfer, Karpenter startup latency, or CPU orchestration overhead.

### 7.13 Eligible User Filtering (Not Applied)

Kevin noted: "We would also probably exclude some users who didn't meet the minimum profile criteria or already had good profiles." This would reduce call volume, but we do not apply this filter in the current estimate.
