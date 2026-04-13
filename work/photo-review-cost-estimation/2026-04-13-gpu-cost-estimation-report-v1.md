# AURA Demo — GPU Node Cost Investigation

> **Sources:**
> - [LP-486](https://hyperconnect.atlassian.net/browse/LP-486) — Cost estimation of LLM call - GPU node / benchmark
> - [LP-493](https://hyperconnect.atlassian.net/browse/LP-493) — LLM+GPU Cost calculator dashboard update
> - `matchgroup-ai/aura-demo` repo: `shared.ts`, `GpuSection.tsx`, `api/routes/cost.py`, `docs/cost-estimation.md`
> - `note/source/2026-04-10-profle-ranker-h100-benchmark.md` (origin: [ldm-models/.../h100_full/REPORT.md](https://github.com/matchgroup-ai/ldm-models/blob/main/profile-ranker-face/results/h100_full/REPORT.md), Biggie Choi, 2026-04-08)
> - Slack #mgai-aura: [Biggie benchmark thread (2026-04-07)](https://tinder.slack.com/archives/C06FB511ZRR/p1775558549452379)
> - Slack #smart-photos-v4: [GPU instances cost thread (2026-01-28)](https://tinder.slack.com/archives/C08PC5KCX0C/p1769618237251849)
> - Slack #cloud-infra-operations: [Jessica Hickey g6e cost estimate (2025-12-30)](https://tinder.slack.com/archives/C097BRC83/p1764900145847879)
> - `note/source/2026-02-03-crs-v3-pii-redaction-cost-estimation.md` (origin: [CRS Chemistry Camera Roll V3 ML Cost Estimates](https://docs.google.com/document/d/1iS9IoJvbHzywiIZalkvSsEyRBcB4cDyc1iJQ8zHxoSk/edit?tab=t.62lfvapg9dfz), Owen Lee, Feb 2026)
> - Slack #mgai-chemistry: [Diego tech report thread (2025-12-30)](https://tinder.slack.com/archives/C0711BSAC03/p1767079310854599)
> - `note/source/2026-02-20-crs-v3-camerarollphotosuggestion-cost-estimation.md` (origin: [revive.mlinfra/.../camerarollphotosuggestion/docs/cost-estimation.md](https://github.com/TinderBackend/revive.mlinfra/blob/main/service/camerarollphotosuggestion/docs/cost-estimation.md))

---

## 1. Conclusion

**Q: How does the aura-demo cost dashboard calculate GPU node cost, and are those numbers reliable for production planning?**

The dashboard's GPU cost arithmetic is correct, but the numbers are not usable for production cost planning. The dashboard models ~$107/month for steady-state GPU inference; a production-realistic estimate using the same deployment pattern as Tinder's existing CRS Photo Suggestion service is ~$3,870/month — roughly 36x higher. The gap comes from three modeling choices that do not match how Tinder actually operates GPU infrastructure: pay-per-use billing on dedicated nodes, no HA redundancy, and a hypothetical RI discount that Tinder does not purchase.

For LP-486, the cost estimation document should use CSP-discounted, 24/7, 4-pod HA figures (~$129/day) rather than the dashboard's default output.

---

## 2. Key Findings

### 2.1. The dashboard understates steady-state cost by ~36x

| What the dashboard says | Production-realistic estimate | Gap |
|-------------------------|------------------------------|-----|
| $107/month (1 GPU, 3yr RI, 4.4h/day) | ~$3,870/month (4 g6e.xlarge, 24/7, CSP) | ~36x |
| $51 one-time backfill | $51 one-time backfill | none |

### 2.2. Three root causes explain the gap

**RI pricing is hypothetical.** The dashboard defaults to 3yr RI ($0.804/hr, 57% off). Tinder uses Compute Savings Plan (CSP), not RI. CSP discount is ~24% for g6e, yielding $1.861 x 0.76 = $1.414/hr — 1.76x higher than the RI rate. No evidence of GPU RI purchases was found.

**Billing model assumes spin-up/down.** `calcGpuCost` charges only for hours of actual processing (4.4h/day at steady state). Jessica Hickey's guidance: "assume your cost estimate is 24/7 usage since the workload requires pre-provisioned dedicated nodes." Both CRS services cost 24h/day regardless of utilization.

**No HA minimum.** The dashboard's minimum is 1 GPU pod. Tinder production standard is 1 pod/AZ x 4 AZs = 4 pods minimum, as seen in both CRS services.

### 2.3. CRS Photo Suggestion is the best comparison target

Three GPU-serving systems compared:

| Aspect | CRS PII Redaction | CRS Photo Suggestion | AURA (dashboard) |
|--------|-------------------|---------------------|------------------|
| Architecture | 3 microservices | Single service | Single vLLM process |
| GPU instance | g5.2xlarge (A10G) | g5.2xlarge (A10G) | g6e.xlarge (L40S) |
| Min cluster | 4 CPU + 8 GPU pods | 4 GPU pods | 1 GPU pod |
| Serving mode | Always-on 24/7 | Always-on 24/7 | Batch (1-24h) |
| Discount | 21% CSP | 21% CSP | 3yr RI (hypothetical) |
| Monthly baseline | $6,270 | $2,760 | $107 |

Photo Suggestion's single-service 4-pod architecture is the closest match to how AURA would deploy. Adjusting from g5 ($1.212/hr) to g6e ($1.861/hr): 4 pods x $1.861 x 24h x 0.76 = $129/day ($3,870/month).

### 2.4. Throughput figure uses full-forward, not production-truncated numbers

The dashboard uses `usersPerSecPerGpu: 79`, which corresponds to Qwen3-VL-8B **full-forward** vLLM at B=16 on H100 (236 users/s / 3.0 = ~79). However, Biggie's benchmark report explicitly recommends using **production-truncated** numbers for sizing. The production SAE concept-extraction path reads hidden states at layer 18 (0-indexed) out of 36 total layers, so only 19 layers need to run — the remaining 17 are wasted compute in full-forward mode. Truncation to 19 layers (trunc=19) yields 280 users/s on H100, extrapolating to **~94 users/s** on L40S — 19% higher than the dashboard's 79.

H100 measured throughput (vLLM, 5-seed mean) and L40S extrapolation (x3.0 blended):

| Model | Mode | H100 B=1 | H100 B=16 | L40S B=1 (est.) | L40S B=16 (est.) |
|-------|------|----------|-----------|-----------------|------------------|
| Qwen3-VL-8B | full-forward | 67 users/s | 236 users/s | ~22 users/s | ~79 users/s |
| Qwen3-VL-8B | trunc=19 (production) | 101 users/s | 280 users/s | ~34 users/s | ~94 users/s |
| Qwen3-VL-32B | trunc=44 (production) | 43 users/s | 92 users/s (B=4) | ~14 users/s | ~31 users/s (B=4) |
| Qwen3-VL-4B | full-forward | 97 users/s | 309 users/s (B=32) | ~32 users/s | ~103 users/s (B=32) |
| Qwen3.5-VL-9B | full-forward | ~30 users/s | ~43 users/s | ~10 users/s | ~14 users/s |

The 3.0x factor is a blended midpoint of compute-bound 2.73x (H100 989 TFLOPS / L40S 362 TFLOPS) and memory-bound 3.88x (H100 3.35 TB/s / L40S 864 GB/s). Direct L40S measurement has not been performed.

The dashboard's 79 users/s is conservative (safe), but a production deployment using truncated inference would be ~19% cheaper per user than the dashboard predicts.

### 2.5. Additional dashboard issues

- **Backfill ignores market coverage**: `GpuSection.tsx:20` always uses raw DAU (18M) for backfill, even when "Experiment US 5%" is selected (should be ~900K).
- **GPU vs API comparison is apples-to-oranges**: "Nx cheaper" compares 8B fine-tuned model (88.2% accuracy) against frontier API models without accounting for quality.
- **No infrastructure overhead**: EBS storage, data transfer, Karpenter startup latency, and CPU orchestration are not modeled.

---

## 3. CRS Production Cost Details

### 3.1. CRS PII Redaction

| Component | Pods | Instance | Daily cost |
|-----------|------|----------|-----------|
| camerarollpiiredaction (CPU) | 4 | m7g.4xlarge | $31.30 |
| mlyoloface (GPU) | 4 | g5.2xlarge | $116.35 |
| mltextdetection (GPU) | 4 | g5.2xlarge | $116.35 |
| Total (before discount) | 12 | | $264/day |
| Total (after 21% CSP) | | | $209/day ($6,270/month) |

Scaling:

| Scenario | Daily users | Daily cost (after CSP) |
|----------|-------------|----------------------|
| Baseline (min cluster) | 0-950K | $209 |
| USA 100% (50% adoption) | 1.3M | $313 |
| AUS+CAN+USA 100% | 1.7M | $418 |
| Global 100% | 20M | $4,180 |

Optimization by Diego Shin and Harvey Ko: 2.8-3.6x capacity gain through GPU batch processing on g5.2xlarge. M1->M2: photos per user 600->100, resolution to 256x256, throughput from 140 to 700 TPM on same cluster.

### 3.2. CRS Photo Suggestion

| Component | Pods | Instance | Daily cost |
|-----------|------|----------|-----------|
| camerarollphotosuggestion (GPU) | 4 | g5.2xlarge | $116.35 |
| Total (after 21% CSP) | | | $92/day ($2,760/month) |

Per-pod benchmark (g5.2xlarge, concurrency 30): 521 TPM (8.69 RPS), P99 3,828 ms, GPU utilization 66% avg / 100% max.

Scaling:

| Scenario | Daily users | Daily cost (after CSP) |
|----------|-------------|----------------------|
| Baseline (min cluster) | 0-2M | $92 |
| Full North America | 1.7M | $92 (no scale-out) |
| Global 100% | 20M | $617 |

---

## 4. Dashboard Implementation Reference

The cost dashboard lives in `matchgroup-ai/aura-demo`. API cost is server-side (`api/routes/cost.py`); GPU cost is frontend-only (`shared.ts`).

Self-hosted model constants:

| Parameter | Value | Source |
|-----------|-------|--------|
| Model | Qwen3-VL-8B SFT LoRA | `SELF_HOSTED.model` |
| Accuracy | 88.2% | `SELF_HOSTED.accuracy` |
| Throughput | 79 users/sec/GPU (full-forward; production truncated = ~94) | `SELF_HOSTED.usersPerSecPerGpu` |
| VRAM | 16 GB (FP16) | `SELF_HOSTED.vramGb` |
| GPU | NVIDIA L40S (48 GB) | `SELF_HOSTED.gpu` |

GPU instance pricing (us-east-1, April 2026):

| Instance | GPU/inst | On-Demand | Spot | 1yr RI | 3yr RI |
|----------|----------|-----------|------|--------|--------|
| g6e.xlarge | 1 | $1.861 | $0.914 | $1.172 | $0.804 |
| g6e.2xlarge | 1 | $2.242 | $1.100 | $1.410 | $0.970 |
| g6e.4xlarge | 1 | $3.004 | $1.480 | $1.890 | $1.300 |
| g6e.12xlarge | 4 | $10.493 | $4.542 | $6.610 | $4.533 |
| g6e.48xlarge | 8 | $30.131 | $9.662 | $18.983 | $13.020 |

Cost formula (`calcGpuCost`):

```
perGpuPerHr = 79 x 3600 = 284,400 users/hr/GPU
gpusNeeded  = ceil(dailyCalls / (perGpuPerHr x batchWindowHrs))
hrsPerGpu   = dailyCalls / (gpusNeeded x perGpuPerHr)
dailyCost   = gpusNeeded x hrsPerGpu x ratePerGpuHr
monthlyCost = dailyCost x 30
```

Traffic presets:

| Scenario | DAU | Trigger Rate | Market Coverage | Notes |
|----------|-----|-------------|----------------|-------|
| Day 0 (Backfill) | 18M | 100% | 100% | One-time |
| Steady State | 18M | 7% | 100% | 3% new logins + 4% profile changes |
| Photo-Only | 18M | 2% | 100% | Photo add/delete only |
| Experiment (US 5%) | 18M | 7% | 5% | US-only |

Ramp-down: `trigger_rate(day) = 4% + 96% x 0.7^day`. Day 0 = 100%, converges to ~7% by week 3.

Arithmetic verification — all worked examples match source code:

| Scenario | gpusNeeded | hrsPerGpu | dailyCost | monthlyCost |
|----------|-----------|-----------|-----------|-------------|
| Steady State (3yr RI, 24h) | 1 | 4.43h | $3.56 | $106.84 |
| Day 0 Backfill (3yr RI, 24h) | 3 | 21.10h | $50.89 | n/a (one-time) |

Infrastructure context:
- Karpenter-managed g6e nodes (xlarge / 2xlarge / 12xlarge), us-east-1a
- Compute Savings Plan: ~24% off on-demand for g6e (Jessica Hickey)
- Ben Shin estimate for 100 GPUs: $1.861 x 100 x 24h x 365 x 0.76 = ~$1.2M/year
- Known issue: GPU node UnexpectedAdmissionError (INC-278) — nvml.Init failures on new nodes
