# SAE + Qwen VL Inference Benchmark Report (H100, **single-user real data**, 5 seeds, HF + vLLM + SGLang)

**Date:** 2026-04-08
**Hardware:** NVIDIA H100 80GB HBM3 (hyperpod, slurm `h100` partition, 1 GPU)
**Branch:** `bench-sae-inference` (forked off `pr-34` / `origin/ken/eval-1k-expansion`)
**HF padded-batching run (5 seeds):** Qwen3-VL [bench_20260408T023050Z](bench_20260408T023050Z_NVIDIA_H100_80GB_HBM3.json), Qwen3.5-VL [bench_20260408T075611Z](bench_20260408T075611Z_NVIDIA_H100_80GB_HBM3.json)
**vLLM continuous-batching run (5 seeds):** Qwen3-VL [vllm_bench_20260408T023624Z](vllm_bench_20260408T023624Z_NVIDIA_H100_80GB_HBM3.json), Qwen3.5-VL 9B+27B [vllm_bench_20260408T083656Z](vllm_bench_20260408T083656Z_NVIDIA_H100_80GB_HBM3.json), Qwen3.5-VL 35B-A3B [vllm_bench_20260408T083858Z](vllm_bench_20260408T083858Z_NVIDIA_H100_80GB_HBM3.json)
**SGLang offline run (5 seeds):** Qwen3-VL [sglang_bench_20260408T024807Z](sglang_bench_20260408T024807Z_NVIDIA_H100_80GB_HBM3.json), Qwen3.5-VL [sglang_bench_20260408T082742Z](sglang_bench_20260408T082742Z_NVIDIA_H100_80GB_HBM3.json)

This report has three measured frameworks side by side on the same input distribution:
- **HF**: HuggingFace `transformers==5.3.0` `model.forward()` with padded batching, `attn_implementation="sdpa"` (PyTorch's built-in cuDNN flash attention dispatch on H100). Includes the SAE forward.
- **vLLM**: `vllm==0.19.0` offline `LLM.generate(max_tokens=1)` with continuous batching, paged attention, chunked prefill, FlashAttention 3 vit kernels. VLM forward only (vLLM doesn't expose hidden states; SAE not measured but is essentially free per the HF run).
- **SGLang**: `sglang==0.5.10` offline `Engine.generate(max_new_tokens=1)` with RadixAttention + continuous batching. Bench script defaults: `attention_backend=fa3`, `mm_attention_backend=fa3`, `mem_fraction_static=0.85` (see [sglang_bench_qwen3vl.py](../../sglang_bench_qwen3vl.py) `_build_parser`). Tuning attempts on top of these defaults hurt performance — see [SGLang section](#sglang-comparison-and-a-surprising-bottleneck). VLM forward only (same caveat).

> **Important:** earlier revisions of this report used **pairwise** inputs (two users per request), because that's the format the backbone Qwen3-VL was fine-tuned on. The **production SAE concept-extraction path is single-user** — the backbone is run on one user's profile + images, and the last-token hidden state is fed into the SAE to compute the concept activations for that one user. All numbers in this report are now single-user; the pair-based numbers from the earlier revisions are not directly comparable.

## TL;DR {#tldr}

**Question:** How fast is single-user TopKSAE + Qwen VL concept extraction on real production data, and what should we deploy on a single L40S (g6e.xlarge)?

**Two latency baselines, full-forward and production-realistic truncated.** The production SAE concept-extraction path attaches the SAE to a *mid-network* hidden layer — specifically, the output of the L-th transformer block (`outputs.hidden_states[L+1][:, -1, :]`). Every transformer layer *after* L is pure compute waste in production. The 8B production candidate reads the SAE off **layer 18** (0-indexed, 19 of 36 layers = 53%) and the 32B candidate reads off **layer 43** (44 of 64 layers = 69%). We measured **both** full-forward (every layer runs) and truncated (only layers 0..L run; the truncated-decoder wall time is the production-realistic number).

**Headline numbers — production truncated 8B and 32B** (5-seed mean throughput at the best batch size, real data, H100, **req/s = users/s**):

| framework                          | 4B (full)  | **8B trunc=19** | **32B trunc=44** | 9B-25 (full) | 27B-25 (full) | 35B-A3B-25 (full) |
|------------------------------------|-----------:|-----------------:|-----------------:|-------------:|--------------:|------------------:|
| HF padded batching (sdpa)          | 17.8 @B=2  |    **19.7** @B=2 |     **9.3** @B=1 |  14.0 @B=2   |   6.7 @B=2    |   3.3 @B=2        |
| **vLLM** (continuous batch, FA3)   | 309 @B=32  |    **280** @B=16 |     **92** @B=4  |   43 @B=16   |    14 @B=1    |    14 @B=2        |
| SGLang (offline, default config)\* | ~9.8 @anyB |          ~9.7\*\*|          ~8.8\*\*|  ~8.9 @anyB  |  ~7.1 @anyB   |  ~6.7 @anyB       |

\* SGLang's offline `Engine.generate()` numbers are surprisingly slow and don't reflect SGLang's true ceiling — see [SGLang section](#sglang-comparison-and-a-surprising-bottleneck). \*\* SGLang was **not re-measured under truncation** (per-request IPC overhead dominates its wall time regardless of model compute, so truncation wouldn't move the needle there).

**Full-forward 8B/32B numbers** for context: 8B full vLLM = **236** @B=16, 32B full vLLM = **70** @B=4. The truncated wall time is **16–33% lower** end-to-end and the truncated throughput is **18–62% higher** across all (framework × model × batch) cells. See [Production SAE early-exit measurements](#production-sae-early-exit-measurements) for every cell.

**Seven things to know:**

1. **The SAE is essentially free.** It adds **0.20–0.41 ms** (< 0.5% of VLM time) to any forward pass regardless of model size or truncation depth. All sizing decisions are about the VLM cost, not the SAE.
2. **Production SAE is mid-network, so truncating the decoder at the SAE layer is the *only* sizing number that matters.** On the Qwen3-VL-8B production candidate (SAE at 0-indexed L18, `num_hidden_layers=19`), truncating saves **17–31% wall time** and **37% VRAM at B=1** (18.4 GB → 11.7 GB). On the 32B production candidate (SAE at L43, `num_hidden_layers=44`), truncating saves **22–27% wall time** and drops VRAM from **67.9 GB → 48.3 GB at B=1**, which is right at the L40S 48 GB ceiling (see fit story below). The full-forward numbers elsewhere in this report are an **upper bound**, not the production cost.
3. **vLLM is 5–18× faster than naive HF padded batching** on the Qwen3-VL family. **The Qwen3.5-VL family gets less vLLM speedup** (~3–9×) because vLLM 0.19's GDN/Mamba SSM prefill kernel is chunked-recurrent rather than fully parallel — see [Why is Qwen3.5-VL slower today?](#why-is-qwen35-vl-slower-today).
4. **HF padded batching's throughput peaks at B=2 for almost every model** on single-user inputs: padding waste from variable image-count per user accumulates fast. The B=2 sweet spot for HF gives the best small-batch latency-vs-throughput trade-off. Truncation preserves this peak.
5. **L40S fit story flips for 32B with truncation**: full 32B is 66 GB of weights (does not fit 48 GB L40S BF16), but **32B trunc=44** loads only ~45 GB of weights (the vLLM loader reports 44.7 GiB on H100), which just barely fits L40S BF16 at B=1 with ~3 GB of KV-cache headroom. This is a candidate-flipping result: the full-32B-on-L40S story has always been "quantize or don't deploy", but the *truncated* 32B is BF16-deployable on a single L40S if you accept B=1 and minimal KV cache. Qwen3.5-VL-27B (~54 GB) and 35B-A3B (~70 GB) still do not fit even with truncation.
6. **The Qwen3.5-VL family is *not* a free upgrade for this workload.** 9B-25 is slower than 8B on every framework, 27B-25 is comparable to 32B, 35B-A3B-25 (MoE) shows essentially zero compute advantage. **Cause is vLLM 0.19's chunked-recurrent SSM prefill kernel** — framework/library maturity, not architecture. Qwen3.5-VL was **not re-measured under truncation** (it is not a production candidate).
7. **Inputs are real production single-user samples** (6.5 images and ~926 tokens average). The prompt mirrors the production [extract_per_profile_sharded.py](../../../experiments/cbm_probe/extract_per_profile_sharded.py) path: same system prompt + Profile A text + Profile A images + the original pair instruction kept verbatim.

**L40S production sizing recommendation — production-realistic (truncated) numbers** (single g6e.xlarge GPU, BF16, blended ×3.0 H100→L40S extrapolation):

- **Qwen3-VL-8B trunc=19 + vLLM**: **~34 users/s** (B=1 latency-sensitive) to **~94 users/s (B=16 throughput-sensitive)** — comfortably fits L40S at every measured batch size; **top pick if 8B accuracy is acceptable**
- **Qwen3-VL-32B trunc=44 + vLLM**: **~14 users/s** (B=1) to **~31 users/s (B=4)** — BF16 fit is tight (load ~48 GB, ~0–3 GB headroom); viable for deployment if you accept B=1 without KV growth, or quantize for batching headroom
- **Qwen3-VL-4B full + vLLM** (not a production candidate but the baseline upper bound): ~32–103 users/s
- Qwen3.5-VL family, Qwen3-VL-4B: not re-measured under truncation; not production candidates for this workload

Full-forward numbers for the same recommendations are in [Strong recommendation](#strong-recommendation) for reference.

**Caveats that matter for interpreting the numbers below:**

- **Image-count per user varies a lot** (range 1–11 images, mean 6.5), driving 5–25% CV at small batches. Single-seed measurements at small batches are unreliable to ±25%.
- **HF used `sdpa`, not standalone `flash-attn` package** (the latter wouldn't install due to ABI conflict). On H100 the two paths agree within ~5% — `sdpa` already dispatches to cuDNN flash attention.
- **L40S numbers are roofline extrapolations from H100 measurements** (×3.0 blended of compute-bound 2.73× and memory-bound 3.88×), not direct measurements.
- **SAE forward is measured in HF only**; vLLM and SGLang don't expose hidden states. The HF measurements show SAE is < 0.5% of VLM time so this is not a meaningful gap for sizing.

[Per-config HF table](#per-config-h100-measurements-real-data-mean-std-over-5-seeds) · [vLLM section](#vllm-comparison-the-production-realistic-ceiling) · [SGLang section](#sglang-comparison-and-a-surprising-bottleneck) · [L40S extrapolation](#l40s-extrapolation) · [Caveats & gaps](#caveats-known-gaps)

---

## Goal {#goal}

Measure latency, throughput, and peak VRAM of running TopKSAE on top of frozen Qwen3-VL hidden states **for the production single-user concept-extraction path**, using real samples from the v5 PairwiseRankingDataset. The goal is to size production inference cost on the deployment target — **L40S (g6e.xlarge, 48 GB)** — using H100 measurements as the baseline.

The backbone Qwen3-VL was fine-tuned on **pairwise** comparisons ("which of these two users gets more likes?") but the production task we're sizing is **single-user concept extraction**: feed one user's profile + images through the backbone, grab a hidden state, and pass it to the TopKSAE to get concept activations for that user. The pair format is an artifact of how the backbone was trained, not what production runs.

To stay close to the backbone's training distribution we keep the system prompt and the original pair instruction verbatim, but drop Profile B entirely:

- system prompt + Profile A text + Profile A images + original pair instruction (verbatim)
- no Profile B section, no second user

This matches the prompt structure used by the production extraction code at [experiments/cbm_probe/extract_per_profile_sharded.py:107](../../../experiments/cbm_probe/extract_per_profile_sharded.py#L107) (which builds a pair prompt with target user as A and a dummy other user as B); the only difference is that we drop the dummy B entirely so latency reflects single-user cost rather than `target + dummy_other`.

**Two latency baselines per model:**
1. **Full forward** — every transformer layer up to the final layer runs, and the SAE reads `outputs.hidden_states[-1][:, -1, :]`. This is the natural thing to do if you don't know where the SAE attaches, and it's the upper bound on inference cost. It also serves as an apples-to-apples comparison against published Qwen3-VL inference benchmarks that all measure full forward.
2. **Production-realistic truncated forward** — only layers `0..K` run, where `K` is the zero-indexed SAE attachment layer + 1 (so the truncated decoder's final hidden state equals the SAE's input). Layers `K..N-1` are not allocated and never run. This is what the production path actually costs.

The Qwen3-VL-8B production candidate has the SAE attached at **layer 18** (0-indexed, of 36 layers), so we truncate to `num_hidden_layers=19`. The Qwen3-VL-32B candidate has the SAE attached at **layer 43** (0-indexed, of 64 layers), so we truncate to `num_hidden_layers=44`. These match the cbm_probe extraction convention in [experiments/cbm_probe/extract_per_profile_sharded.py:46](../../../experiments/cbm_probe/extract_per_profile_sharded.py#L46) (where `DEFAULT_LAYERS = [15, 31, 47, 63]` uses 0-indexed layer output convention).

Both HF and vLLM are measured under both baselines. **The production sizing recommendation is based on the truncated numbers** — see [Production SAE early-exit measurements](#production-sae-early-exit-measurements). The full-forward sections below remain as an upper bound and as the comparison point for published Qwen3-VL benchmarks.

Real single-user samples:
- ~6.5 images per user (range 1–11)
- ~926 total tokens (range 219–1476): ~85% vision tokens, ~15% text tokens
- Variable shape across samples → padding waste during HF batching, no waste under continuous batching

## Setup {#setup}

### Software Stack {#software-stack}
- **Inference framework:** **HuggingFace `transformers==5.3.0`**, plain `model.forward()` call. **No vLLM, TensorRT-LLM, TGI, or `torch.compile`.**
- **PyTorch:** `2.6.0+cu124` (`torch==2.6.0`, CUDA 12.4)
- **Attention backend:** `sdpa` — PyTorch's built-in `scaled_dot_product_attention`, dispatches to cuDNN flash attention on H100. Production code in [profile-ranker-face/src/profile_ranker_face/training/runtime.py:67](../../src/profile_ranker_face/training/runtime.py#L67) pins `flash_attention_2` (the standalone `flash-attn` package); on H100 the two backends typically agree within 5–15%.
- **Precision:** BF16 throughout (model weights, activations, SAE)
- **Forward call:** `model(**inputs, output_hidden_states=True, return_dict=True)` — single prefill pass, no `generate()` / autoregressive decode.
- **Hidden state extraction:** `outputs.hidden_states[-1][:, -1, :]` → last token of last transformer layer → `[B, hidden_size]` → SAE.
- **Transient extras (not yet in pyproject):** `torchvision==0.21.0` (Qwen3-VL processor needs it for `Qwen3VLVideoProcessor`), `deltalake==1.5.0` (for reading the v5 delta-format pairwise tables).

### Models {#models}

| Key         | HF model id                  | hidden_size | approx params | architecture            | BF16 weights |
|------------:|------------------------------|------------:|--------------:|-------------------------|-------------:|
|  4B         | `Qwen/Qwen3-VL-4B-Instruct`  |        2560 |          ~4 B | Qwen3-VL (full attn)    |       ~8 GB  |
|  8B         | `Qwen/Qwen3-VL-8B-Instruct`  |        4096 |          ~8 B | Qwen3-VL (full attn)    |      ~17 GB  |
| 32B         | `Qwen/Qwen3-VL-32B-Instruct` |        5120 |         ~32 B | Qwen3-VL (full attn)    |      ~66 GB  |
| 9B-25       | `Qwen/Qwen3.5-9B`            |        4096 |          ~9 B | Qwen3.5-VL (hybrid lin+full) | ~18 GB  |
| 27B-25      | `Qwen/Qwen3.5-27B`           |        5120 |         ~27 B | Qwen3.5-VL (hybrid lin+full) | ~54 GB  |
| 35B-A3B-25  | `Qwen/Qwen3.5-35B-A3B`       |        2048 |   ~35 B (3B active) | Qwen3.5-VL MoE (256 experts, 8 active per token) | ~70 GB |

All loaded as base models (no LoRA, no checkpoints). Hardware compute is invariant to weight values, so untrained random-init models give the same forward-pass cost as production-trained ones.

The Qwen3.5-VL family uses **hybrid attention**: 3 Mamba-family SSM layers (Gated DeltaNet) followed by 1 full-attention layer, repeating. This was designed to make long-context inference cheaper because the SSM layers carry constant per-step state instead of a growing KV cache. The trade-off, which dominates our prefill-only workload, is that vLLM 0.19's GDN prefill kernel uses a chunked-recurrent algorithm rather than the fully parallel scan used by training frameworks — see [Why is Qwen3.5-VL slower today?](#why-is-qwen35-vl-slower-today). **The `-25` suffix in model keys is shorthand for "Qwen3.5"** (e.g., `9b25` = Qwen3.5-9B = `Qwen/Qwen3.5-9B`).

The HF model ids look text-only (`Qwen/Qwen3.5-9B`, no `-VL-` infix) but they ARE multimodal Qwen3.5-VL models — Qwen3.5's naming convention puts VL as the default. The `config.json` has `vision_config` and the architecture is `Qwen3_5ForConditionalGeneration` (or `Qwen3_5MoeForConditionalGeneration` for the MoE variant), both registered in vLLM 0.19's supported models list as multimodal `T+I+V`.

### SAE {#sae}
- **Architecture:** `TopKSAE` from [profile-ranker-face/sae_layer.py:16](../../sae_layer.py#L16) — encoder linear → TopK sparsity → decoder linear (~270 MFLOPs per sample at most).
- **`num_features`:** 16384, **`k`:** 32, **dtype:** BF16, **init:** random (never loaded from a checkpoint).

### Inputs (real single-user data) {#inputs-real-single-user-data}

#### What is one "sample"? {#what-is-one-sample}

**One sample = one user** (one Profile A only). The backbone gets a Qwen3-VL prompt with the system prompt, the user's profile metadata, the user's images, and the original pair instruction kept verbatim — no Profile B. The hidden state we feed into the SAE is the last-token hidden state of this single-user forward pass.

Concretely, the prompt looks like (from [bench_sae_runtime._build_single_user_sample](../../bench_sae_runtime.py)):

```
[system text]      "You are a dating app matching algorithm."

[Profile A text]   {gender, age, bio (≤500 chars), interests, verified,
                    has_school, has_job, looking_for}        ← target user's metadata
[Profile A images] <image><image>...                         ← all of the target user's images

[instruction]      "Based on your perspective as a dating app user, which person
                    gets more likes? Do not explain. Generate just one letter:
                    A or B."         ← kept verbatim from the pair training prompt
```

The instruction does not "make sense" without a Profile B, but we keep it verbatim because the backbone was fine-tuned with this exact instruction in its training prompts; dropping or changing it would push the input further from the training distribution. The `A`/`B` decision token at the end is irrelevant — we never generate, we only forward and read the last-token hidden state.

**Why this is the right input:** the production SAE concept-extraction path runs the backbone on a single user, not on a pair. The backbone was trained on pairs only because that was the labelled data available; the SAE consumes a per-user representation that should reflect that user alone. The earlier (now-superseded) revisions of this report used pair inputs and overstated the per-user latency by ~2×.

#### Source files {#source-files}

Samples come from the **v5 PairwiseRankingDataset eval split** (`/data/mgai/aura/profile-ranker/data/v5/eval_model_pairs`), but we only keep the left user from each row (post-shuffle), so each sample is one user:

- **Source:** [profile-ranker-face/src/profile_ranker_face/data_processing/dataset.py](../../src/profile_ranker_face/data_processing/dataset.py) `PairwiseRankingDataset` (we use only the left side)
- **Single-user prompt builder:** [profile-ranker-face/bench_sae_runtime.py](../../bench_sae_runtime.py) `_build_single_user_sample` (mirrors `PromptBuilder` but with Profile B dropped)
- **Image preprocessing:** all profile images resized to **320×400** (matches the v5 default prompt config)
- **Sample format:** one user's profile JSON metadata + that user's full image set

**5 sample windows are drawn from the eval split** with seeds `[1, 2, 3, 4, 5]`. Each seed gives a different 32-sample window of users. Within a seed, the same samples are used across all `(model, batch_size)` configs (so the per-seed comparison across batch sizes is fair). Across seeds, we compute mean ± std to get statistical confidence.

#### Per-sample token length distribution (n=160 samples, 5 seeds × 32 samples) {#per-sample-token-length-distribution}

Computed by [profile-ranker-face/analyze_sample_token_lengths.py](../../analyze_sample_token_lengths.py) running each pickled sample through the Qwen3-VL HF processor twice (text-only, then text+images) to get exact text and vision token counts. Raw output: [sample_token_lengths.json](sample_token_lengths.json).

| metric          |    mean |   min |   max |    p50 |    p95 |
|-----------------|--------:|------:|------:|-------:|-------:|
| `n_images`      |     6.5 |     1 |    11 |    6.0 |    9.0 |
| `text_tokens`   |   153.6 |   100 |   280 |  146.0 |  221.0 |
| `vision_tokens` |   772.8 |   119 |  1309 |  714.0 | 1071.0 |
| `total_tokens`  |   926.4 |   219 |  1476 |  883.5 | 1274.0 |

**Three things this reveals:**

1. **Vision tokens dominate.** Mean **82.1%** (range 41.5%–89.6%) of every sample's input is vision tokens. This is why the vision encoder is the dominant cost on real data and why vision-encoder optimizations (FA3 ViT kernel, ViT CUDA graph) are the biggest single factor in the vLLM speedup.
2. **Vision tokens per image is *exactly* 119, with zero variance.** Qwen3-VL's processor maps every 320×400 image to a fixed 119-token grid (no dynamic patching at this resolution). Total vision tokens = `n_images × 119` exactly. This means the input length distribution is fully determined by the image-count distribution per sample.
3. **Text tokens are tiny — mean 154, max 280.** The chat template + JSON profile metadata + instruction fits in well under 300 tokens, even at the longest. **Total token growth across batches is essentially driven by image count.**

### Measurement Protocol {#measurement-protocol}

**Warmup / measurement iterations per `(model, batch_size, seed)` cell.** Not all three frameworks use the same counts — vLLM/SGLang per-call overhead is much higher than HF, so fewer iters keep total runtime sane:

| framework | warmup | measure | source |
|-----------|-------:|--------:|--------|
| HF        |      3 |     10  | [bench_sae_inference.py](../../bench_sae_inference.py) `--warmup` / `--iters` defaults |
| vLLM      |      2 |      5  | [vllm_bench_qwen3vl.py](../../vllm_bench_qwen3vl.py) `WARMUP_ITERS` / `MEASURE_ITERS` |
| SGLang    |      3 |      5  | [sglang_bench_qwen3vl.py](../../sglang_bench_qwen3vl.py) `WARMUP_ITERS` / `MEASURE_ITERS` |

Warmup passes are discarded (let cuDNN heuristics / vLLM compile / SGLang radix cache stabilize). Within-seed jitter from this many measure iters is < 1 % at the p50 level (see [Sanity Checks](#sanity-checks) item 7), so the dominant noise source across all three frameworks is input-window variance across the 5 seeds, not within-seed timing jitter.

**Other protocol details:**

- **Timer:** `torch.cuda.Event(enable_timing=True)` (HF) or `time.perf_counter()` bracketed by `torch.cuda.synchronize()` (vLLM / SGLang) on the GPU stream
- **Synchronization:** `torch.cuda.synchronize()` before recording start AND after end events each iteration
- **VLM time bracket:** From `vlm_start.record()` through the last-token hidden slice (HF); end-to-end `LLM.generate(max_tokens=1)` / `Engine.generate(max_new_tokens=1)` (vLLM / SGLang)
- **SAE time bracket:** Just the SAE forward call on the extracted hidden vector (HF only — vLLM / SGLang don't expose hidden states; per HF this is < 0.5 % of VLM time so the gap is not material)
- **End-to-end:** `vlm_ms_mean + sae_ms_mean` (HF); `wall_ms_mean` (vLLM / SGLang)
- **VRAM:** `torch.cuda.reset_peak_memory_stats()` after warmup, `torch.cuda.max_memory_allocated()` after measurement (HF); driver-level `torch.cuda.mem_get_info()` after warmup (vLLM / SGLang, because the engine runs in a subprocess where `max_memory_allocated()` doesn't see across processes — note that this captures the engine's *reservation*, not just live model usage)
- **Statistics:** mean, p50, p95 from the per-iter sample list (10 for HF, 5 for vLLM / SGLang)

### Sweeps {#sweeps}

- **4B:** `[1, 2, 4, 8, 16, 32]`
- **8B:** `[1, 2, 4, 8, 16]`
- **32B:** `[1, 2, 4]`
- **9B-25:** `[1, 2, 4, 8, 16]`
- **27B-25:** `[1, 2, 4]`
- **35B-A3B-25:** `[1, 2]` (B=4 OOMs in HF; B=2 already at 72 GB)

All 24 unique `(model, batch)` configs ran to completion in HF (= 6+5+3+5+3+2 from the sweep above × 5 seeds = **120 measurements per metric**); vLLM also completed all 24 (35B-A3B required `gpu_memory_utilization=0.95` instead of the default `0.9` because the 70 GB weights leave very little room for KV cache). SGLang completed all 24.

## Results {#results}

### Per-config H100 measurements (real data, mean ± std over 5 seeds) {#per-config-h100-measurements-real-data-mean-std-over-5-seeds}

| model       | B  | n | vlm p50 (ms)    | sae p50 (ms)  | thru (users/s) | peak VRAM (GB) |
|------------:|---:|--:|----------------:|--------------:|---------------:|---------------:|
| 4B          |  1 | 5 |    77.0 ± 4.6   | 0.209 ± 0.004 |  12.99 ± 0.78  |   9.59 ± 0.11  |
| 4B          |  2 | 5 |   116.0 ± 25.6  | 0.223 ± 0.004 |  17.83 ± 3.54  |  10.49 ± 0.44  |
| 4B          |  4 | 5 |   232.4 ± 33.8  | 0.230 ± 0.002 |  17.50 ± 2.63  |  12.32 ± 0.50  |
| 4B          |  8 | 5 |   475.4 ± 32.0  | 0.237 ± 0.004 |  16.88 ± 1.05  |  15.96 ± 0.51  |
| 4B          | 16 | 5 |   936.7 ± 67.1  | 0.250 ± 0.003 |  17.15 ± 1.17  |  22.90 ± 0.97  |
| 4B          | 32 | 5 |  1918.4 ± 106.4 | 0.247 ± 0.006 |  16.69 ± 0.87  |  37.14 ± 1.73  |
| 8B          |  1 | 5 |    81.2 ± 3.5   | 0.244 ± 0.005 |  12.30 ± 0.53  |  18.38 ± 0.12  |
| 8B          |  2 | 5 |   145.1 ± 38.0  | 0.260 ± 0.003 |  14.46 ± 3.43  |  19.44 ± 0.52  |
| 8B          |  4 | 5 |   303.6 ± 44.1  | 0.268 ± 0.003 |  13.40 ± 2.07  |  21.59 ± 0.60  |
| 8B          |  8 | 5 |   624.9 ± 44.7  | 0.277 ± 0.004 |  12.85 ± 0.84  |  25.86 ± 0.61  |
| 8B          | 16 | 5 |  1245.0 ± 89.6  | 0.303 ± 0.006 |  12.89 ± 0.88  |  34.00 ± 1.14  |
| 32B         |  1 | 5 |   139.2 ± 22.9  | 0.270 ± 0.008 |   7.32 ± 1.21  |  67.94 ± 0.19  |
| 32B         |  2 | 5 |   373.7 ± 124.3 | 0.293 ± 0.003 |   5.81 ± 1.74  |  69.66 ± 0.84  |
| 32B         |  4 | 5 |   837.1 ± 141.6 | 0.297 ± 0.006 |   4.89 ± 0.86  |  73.16 ± 0.97  |
| 9B-25       |  1 | 5 |   101.2 ± 3.9   | 0.245 ± 0.003 |   9.84 ± 0.41  |  19.74 ± 0.12  |
| 9B-25       |  2 | 5 |   147.5 ± 32.5  | 0.260 ± 0.003 |  14.02 ± 2.79  |  20.93 ± 0.55  |
| 9B-25       |  4 | 5 |   300.6 ± 41.0  | 0.270 ± 0.002 |  13.50 ± 1.95  |  23.33 ± 0.63  |
| 9B-25       |  8 | 5 |   617.7 ± 39.6  | 0.286 ± 0.003 |  12.98 ± 0.77  |  28.06 ± 0.65  |
| 9B-25       | 16 | 5 |  1235.6 ± 78.8  | 0.314 ± 0.012 |  12.99 ± 0.78  |  37.15 ± 1.21  |
| 27B-25      |  1 | 5 |   170.1 ± 4.3   | 0.271 ± 0.004 |   5.88 ± 0.15  |  56.13 ± 0.18  |
| 27B-25      |  2 | 5 |   319.3 ± 92.4  | 0.295 ± 0.004 |   6.66 ± 1.75  |  57.98 ± 0.84  |
| 27B-25      |  4 | 5 |   695.1 ± 103.7 | 0.296 ± 0.002 |   5.86 ± 0.92  |  61.79 ± 0.96  |
| 35B-A3B-25  |  1 | 5 |   534.2 ± 17.4  | 0.198 ± 0.005 |   1.87 ± 0.06  |  70.94 ± 0.11  |
| 35B-A3B-25  |  2 | 5 |   609.7 ± 29.3  | 0.214 ± 0.003 |   3.28 ± 0.15  |  71.98 ± 0.47  |

`n=5` is the number of seeds. Each cell value is the cross-seed mean of 5 within-seed medians (each within-seed median is itself the median of 10 timed forward passes after 3 warmup passes — see [Measurement Protocol](#measurement-protocol)). 24 unique `(model, batch)` configs × 5 seeds = **120 raw measurements per metric** in total for the HF table.

### Variance analysis: how stable are these numbers? {#variance-analysis-how-stable-are-these-numbers}

**Coefficient of variation (CV = std / mean × 100%) for the noisier metrics:**

| model      | vlm_p50 CV @ B=1 | vlm_p50 CV @ B=2 | thru CV @ B=2 |
|-----------:|-----------------:|-----------------:|--------------:|
| 4B         |               6% |              22% |           20% |
| 8B         |               4% |              26% |           24% |
| 32B        |              16% |              33% |           30% |
| 9B-25      |               4% |              22% |           20% |
| 27B-25     |               3% |              29% |           26% |
| 35B-A3B-25 |               3% |               5% |            5% |

VRAM CV is small everywhere (1–4%) because peak VRAM is dominated by the model weights, which are constant. CV is small at B=1 and grows with batch up to B=2/4 because each new sample randomly contributes 1–11 more images (a much wider relative range than at the pair level), then stabilizes back at large batches as the longest sample tends to be present in any window.

**Read this as:** for a single-seed measurement at small batches, expect **~5–25% noise** on latency and throughput (the wider end at B=2–4 where image-count variance bites hardest). At large batches the noise drops back below 10%.

**Implications for sizing decisions:**
1. **B=1 noise is small (~5%)** because everyone has roughly one user's worth of images, and with a single sample the longest-image-set effect doesn't compound.
2. **B=2–4 noise is highest (~20–30%)** because doubling/quadrupling can land you on a 11-image user that drags the padded length up, or all 5-image users that don't.
3. **Comparing two configs that differ by less than ~25% at B=2–4** requires multi-seed measurement to call it a real difference.
4. **Within a single seed (the 10 timed forward passes), p95 and p50 agree within 1%**. The CV in this report is **input variance across seeds**, not GPU jitter.

### Throughput vs batch size {#throughput-vs-batch-size}

Looking at the `thru` column above, single-user HF throughput **rises slightly from B=1 to B=2** (4B: 13.0 → 17.8, 8B: 12.3 → 14.5, 9B-25: 9.8 → 14.0) and then **roughly plateaus** from B=2 onward. For 32B / 27B-25 the plateau is at or below B=1 — even the smallest pad waste at B=2 already costs more than the batching gain. 35B-A3B-25 also benefits from B=2 (B=1 = 1.87, B=2 = 3.28 users/s) but the absolute numbers are very small.

**Per-sample efficiency** (= `actual_thru / (B × thru@B=1)`):

| B  | 4B   | 8B   | 32B  | 9B-25 | 27B-25 | 35B-A3B-25 |
|---:|-----:|-----:|-----:|------:|-------:|-----------:|
|  1 | 100% | 100% | 100% |  100% |   100% |       100% |
|  2 |  69% |  59% |  40% |   71% |    57% |        88% |
|  4 |  34% |  27% |  17% |   34% |    25% |          — |
|  8 |  16% |  13% |    — |   16% |      — |          — |
| 16 |   8% |   7% |    — |    8% |      — |          — |
| 32 |   4% |    — |    — |     — |      — |          — |

Per-batch-doubling efficiency drops fast even on single-user inputs because the longest-image user in each batch sets the padded length for all of them. **HF padded batching's practical sweet spot is B=2 for almost every model** (best tradeoff between latency and throughput) — except 32B and 27B-25 where **B=1 is best** (no batch helps). This is a strong argument for **request-per-request serving** on the HF path, unless the batcher can group same-image-count users explicitly. The vLLM section below shows what happens when this padding waste is removed via continuous batching.

### VRAM scales with image count, not just batch {#vram-scales-with-image-count-not-just-batch}

Peak VRAM at the largest batch tested per model (mean ± std over 5 seeds):

| Model      | B=1 (GB)       | Max B tested | Max VRAM (GB)    | H100 80GB headroom |
|-----------:|---------------:|-------------:|-----------------:|-------------------:|
| 4B         |  9.59 ± 0.11   |           32 |   37.14 ± 1.73   |            ~48 GB  |
| 8B         | 18.38 ± 0.12   |           16 |   34.00 ± 1.14   |            ~51 GB  |
| 32B        | 67.94 ± 0.19   |            4 |   73.16 ± 0.97   |            ~12 GB  |
| 9B-25      | 19.74 ± 0.12   |           16 |   37.15 ± 1.21   |            ~48 GB  |
| 27B-25     | 56.13 ± 0.18   |            4 |   61.79 ± 0.96   |            ~24 GB  |
| 35B-A3B-25 | 70.94 ± 0.11   |            2 |   71.98 ± 0.47   |             ~9 GB  |

Compared to the earlier (now-deleted) pair runs, single-user images-per-batch is roughly half, so VRAM at the max batch is about half too: 4B B=32 is 37 GB instead of 64 GB, 8B B=16 is 34 GB instead of 50 GB. **For 32B, B=4 still leaves only ~12 GB of headroom on H100** in BF16, and **35B-A3B-25 has only ~9 GB of headroom even at B=2** — `B=4` for 35B-A3B-25 OOMs on H100 80 GB. The Qwen3.5-VL family's per-batch VRAM growth is very similar to the Qwen3-VL family at comparable parameter counts, so the family choice doesn't shift VRAM/throughput trade-offs much.

### SAE marginal cost (essentially free) {#sae-marginal-cost-essentially-free}

The SAE adds **0.20–0.31 ms** per forward pass (mean across seeds), tightly clustered (std ≤ 0.012 ms). Vanishingly small compared to VLM time (77–1918 ms): the highest SAE/VLM ratio is **0.30% (4B B=1)**, the lowest is **0.013% (4B B=32)**.

VLM/SAE time ratio range: **332× (4B B=1) to ~7800× (4B B=32)** — far above the 50× sanity threshold in every config. The Qwen3.5-VL family lands in the same range; e.g., 9B-25 SAE adds 0.245–0.314 ms across batch sizes, identical to 8B's 0.244–0.303 ms.

**The SAE is effectively free on top of any VLM size.** All sizing discussion below is about the VLM cost.

## L40S Extrapolation {#l40s-extrapolation}

**These are estimates, not measurements.** L40S was not benchmarked directly. Each H100 latency is multiplied by a constant factor derived from the GPU spec ratios.

L40S (Ada Lovelace, sm_89) vs H100 SXM5 (Hopper, sm_90):

| Spec               | H100 SXM5 |  L40S | L40S/H100 | Multiplier on H100 latency |
|--------------------|----------:|------:|----------:|---------------------------:|
| BF16 Tensor TFLOPS |       989 |   362 |    0.366× |            **×2.73** (compute-bound) |
| Memory bandwidth   | 3350 GB/s | 864 GB/s | 0.258× |            **×3.88** (memory-bound)  |
| VRAM               |     80 GB | 48 GB |    0.60×  |            — |
| Memory type        |      HBM3 | GDDR6 |    —      |            — |
| Blended midpoint   |         — |   —   |    —      |            **×3.00** (rough average) |

### Per-config L40S extrapolation (HF, estimated; from 5-seed mean H100 latency) {#per-config-l40s-extrapolation-estimated-not-measured-from-5-seed-mean-h100-latency}

| model       | B  | compute (×2.73) ms | blended (×3.0) ms | mem (×3.88) ms | blended thru (users/s) |
|------------:|---:|-------------------:|------------------:|---------------:|-----------------------:|
| 4B          |  1 |                211 |               232 |            299 |                   4.32 |
| 4B          |  2 |                317 |               349 |            451 |                   5.74 |
| 4B          |  4 |                636 |               698 |            902 |                   5.73 |
| 4B          |  8 |               1299 |              1427 |           1844 |                   5.61 |
| 4B          | 16 |               2559 |              2810 |           3632 |                   5.69 |
| 4B          | 32 |               5249 |              5764 |           7450 |                   5.55 |
| 8B          |  1 |                222 |               244 |            316 |                   4.09 |
| 8B          |  2 |                397 |               436 |            564 |                   4.58 |
| 8B          |  4 |                830 |               912 |           1178 |                   4.39 |
| 8B          |  8 |               1708 |              1875 |           2423 |                   4.27 |
| 8B          | 16 |               3403 |              3737 |           4830 |                   4.28 |
| 32B\*       |  1 |                381 |               419 |            541 |                   2.39 |
| 32B\*       |  2 |               1022 |              1122 |           1450 |                   1.78 |
| 32B\*       |  4 |               2289 |              2513 |           3248 |                   1.59 |
| 9B-25       |  1 |                278 |               305 |            394 |                   3.28 |
| 9B-25       |  2 |                404 |               443 |            573 |                   4.51 |
| 9B-25       |  4 |                823 |               903 |           1167 |                   4.43 |
| 9B-25       |  8 |               1688 |              1854 |           2396 |                   4.31 |
| 9B-25       | 16 |               3376 |              3707 |           4791 |                   4.32 |
| 27B-25\*    |  1 |                465 |               511 |            660 |                   1.96 |
| 27B-25\*    |  2 |                873 |               959 |           1239 |                   2.09 |
| 27B-25\*    |  4 |               1900 |              2086 |           2697 |                   1.92 |
| 35B-A3B-25\*|  1 |               1461 |              1604 |           2073 |                   0.62 |
| 35B-A3B-25\*|  2 |               1667 |              1830 |           2366 |                   1.09 |

`blended thru` = `B × 1000 / blended_ms`. \* 32B does not fit in L40S 48 GB VRAM in BF16 — see warning below; values are reference-only.

**Caveat about scaling shape:** every L40S latency is a uniform multiple of the H100 latency, so the throughput-vs-batch shape is mathematically identical to H100's. Real L40S would differ slightly because L40S has a different roofline ridge point — but the difference is workload-dependent and only a real L40S run can settle the question. Treat the numbers above as a sizing aid, not a precise prediction.

### Critical: three of six models do NOT fit on L40S in BF16 {#critical-three-models-do-not-fit-on-l40s-in-bf16}

L40S has **48 GB VRAM total**. The following models do not fit in BF16:

| Model            | BF16 weights | Fits L40S 48 GB? | What's needed for L40S |
|------------------|-------------:|:----------------:|------------------------|
| Qwen3-VL-4B      |       ~8 GB  |        ✅         | nothing — fits comfortably (batch 32+) |
| Qwen3-VL-8B      |      ~17 GB  |        ✅         | nothing — fits comfortably (batch 16+) |
| Qwen3-VL-32B     |      ~66 GB  |        ❌         | FP8 (~33 GB) or NF4/INT4 (~17 GB) |
| Qwen3.5-VL-9B    |      ~18 GB  |        ✅         | nothing — fits comfortably (batch 16+) |
| Qwen3.5-VL-27B   |      ~54 GB  |        ❌         | FP8 (~27 GB) or NF4/INT4 (~14 GB) |
| Qwen3.5-VL-35B-A3B |    ~70 GB  |        ❌         | FP8 (~35 GB) or NF4/INT4 (~18 GB) |

For the three that don't fit, the L40S latency estimates earlier in this report describe a hypothetical L40S that could hold the BF16 weights — they are reference-only.

Production on L40S **requires** weight quantization for 32B / 27B-25 / 35B-A3B-25:
- **FP8** (sm_89 has FP8 tensor cores): half-size weights → fits, batch 1–2 likely. Native support in vLLM via the marlin/w8a8 kernels.
- **NF4 / INT4** (BitsAndBytes / AutoAWQ): quarter-size weights → fits comfortably, batch 2–4 likely. AWQ quants exist for many Qwen3-VL checkpoints; Qwen3.5-VL quants are less mature.

The benchmark CLI has a `--quant {none,nf4,int8}` stub that currently raises `NotImplementedError`. Implementing it is the natural follow-up for sizing the three large models on L40S.

**Note on the MoE 35B-A3B-25:** MoE means only 3 B parameters are *activated* per token, but **all 35 B parameters still need to be in VRAM** (the routing decides which experts to use per token; you can't predict which experts a request will need ahead of time). MoE saves compute, not memory. So 35B-A3B-25 needs the same quantization as the dense 35 B equivalent to fit on L40S.

### L40S sizing TL;DR (BF16, blended estimate, 5-seed mean) {#l40s-sizing-tldr-bf16-blended-estimate-5-seed-mean}

For **HF padded batching** alone (the baseline path):

- **4B B=2** → ~349 ms latency, **~5.7 users/s** on L40S (best HF point overall)
- **8B B=2** → ~436 ms latency, **~4.6 users/s** on L40S
- **32B B=1** *(needs quantization)* → ~419 ms latency, **~2.4 users/s** on L40S
- **9B-25 B=2** → ~443 ms latency, **~4.5 users/s** on L40S
- **27B-25 B=2** *(needs quantization)* → ~959 ms latency, **~2.1 users/s** on L40S
- **35B-A3B-25 B=2** *(needs quantization)* → ~1830 ms latency, **~1.1 users/s** on L40S

This is the baseline. The vLLM section below shows that **switching framework lifts these by 3–18×** without changing models or quantization (the lower end of that range applies to the Qwen3.5-VL family, which gets less vLLM speedup than Qwen3-VL).

## vLLM comparison (the production-realistic ceiling) {#vllm-comparison-the-production-realistic-ceiling}

Same 5 seeds, same 32-sample windows from the same v5 eval split, same six models in BF16 (Qwen3-VL-{4B,8B,32B} and Qwen3.5-VL-{9B,27B,35B-A3B}). Different framework: **vLLM 0.19.0** offline `LLM.generate(max_tokens=1)` instead of `transformers` `model.forward()`. Single-user inputs throughout.

The Qwen3.5-VL family was added in two separate vLLM runs because 35B-A3B (~70 GB weights) needed `gpu_memory_utilization=0.95` (the default 0.9 left so little room for KV cache that vLLM refused to start), and trying to share an engine across all three Qwen3.5-VL models in one run failed during engine reinitialization between models.

vLLM differences from the HF path:
- **Continuous batching**: requests are packed dynamically, no padding to the longest sample.
- **Paged attention**: KV cache is broken into pages, allowing different sequence lengths in the same batch.
- **Chunked prefill**: long prefills are chunked across iterations to overlap with decode of other requests.
- **FlashAttention 3 vit kernel**: vLLM 0.19 uses FA3 for the vision encoder on H100 (not available in our HF path because the standalone `flash-attn` package failed to install — see Caveats).
- **CUDA graph capture**: PIECEWISE + FULL CUDA graphs over batch sizes 1..512.
- **`torch.compile`**: vLLM compiles the model graph once at startup.

What vLLM does NOT measure here:
- The SAE forward pass. vLLM doesn't expose hidden states as a public API. The HF run shows the SAE adds 0.21–0.30 ms (< 0.5% of VLM time across all configs), so this is not a meaningful gap for sizing.

The vLLM script lives at [profile-ranker-face/vllm_bench_qwen3vl.py](../../vllm_bench_qwen3vl.py); it loads the same single-user sample windows from pickles (dumped via [profile-ranker-face/dump_real_samples.py](../../dump_real_samples.py)) so HF and vLLM see byte-identical inputs across seeds. vLLM runs in a separate venv at `~/Workspace/git/aura/bench-sae-inference/vllm_bench/`.

### Per-config vLLM measurements (5 seeds) {#per-config-vllm-measurements-5-seeds}

| model      | B  | n | wall p50 (ms)   | thru (users/s)    | peak VRAM (GB) | L40S~ms (blended) |
|-----------:|---:|--:|----------------:|------------------:|---------------:|------------------:|
| 4B         |  1 | 5 |     10.2 ± 1.0  |   98.09 ± 11.60   |  78.06 ± 0.02  |       31 ± 3      |
| 4B         |  2 | 5 |     18.1 ± 1.2  |  110.89 ± 7.89    |  78.06 ± 0.01  |       54 ± 4      |
| 4B         |  4 | 5 |     25.4 ± 3.3  |  162.61 ± 22.41   |  78.06 ± 0.01  |       75 ± 10     |
| 4B         |  8 | 5 |     37.1 ± 1.0  |  215.88 ± 6.10    |  78.06 ± 0.01  |      111 ± 3      |
| 4B         | 16 | 5 |     59.3 ± 2.1  |  270.86 ± 7.44    |  78.06 ± 0.01  |      177 ± 5      |
| 4B         | 32 | 5 |    103.9 ± 3.3  |  308.85 ± 8.10    |  78.07 ± 0.01  |      311 ± 8      |
| 8B         |  1 | 5 |     14.8 ± 2.1  |   68.36 ± 12.06   |  77.81 ± 0.00  |       45 ± 6      |
| 8B         |  2 | 5 |     27.7 ± 2.3  |   72.50 ± 6.50    |  77.81 ± 0.00  |       83 ± 7      |
| 8B         |  4 | 5 |     29.4 ± 2.7  |  137.00 ± 13.71   |  77.81 ± 0.00  |       88 ± 8      |
| 8B         |  8 | 5 |     45.8 ± 3.0  |  175.12 ± 12.93   |  77.81 ± 0.00  |      138 ± 9      |
| 8B         | 16 | 5 |     67.9 ± 2.6  |  235.89 ± 8.99    |  77.81 ± 0.00  |      204 ± 8      |
| 32B        |  1 | 5 |     30.3 ± 0.7  |   32.92 ± 0.73    |  77.91 ± 0.00  |       91 ± 2      |
| 32B        |  2 | 5 |     56.0 ± 0.4  |   35.62 ± 0.21    |  77.91 ± 0.00  |      168 ± 1      |
| 32B        |  4 | 5 |     57.4 ± 1.0  |   69.58 ± 1.16    |  77.91 ± 0.00  |      172 ± 3      |
| 9B-25      |  1 | 5 |     34.7 ± 5.9  |   29.65 ± 6.46    |  82.59 ± 0.00  |      104 ± 18     |
| 9B-25      |  2 | 5 |     70.3 ± 6.1  |   28.62 ± 2.68    |  82.59 ± 0.00  |      211 ± 18     |
| 9B-25      |  4 | 5 |    105.1 ± 10.2 |   38.37 ± 3.85    |  82.59 ± 0.00  |      315 ± 31     |
| 9B-25      |  8 | 5 |    196.4 ± 9.9  |   40.73 ± 2.07    |  82.59 ± 0.00  |      590 ± 28     |
| 9B-25      | 16 | 5 |    376.4 ± 26.2 |   42.77 ± 2.86    |  82.59 ± 0.00  |     1126 ± 77     |
| 27B-25     |  1 | 5 |     71.7 ± 9.0  |   14.08 ± 1.62    |  82.93 ± 0.00  |      216 ± 27     |
| 27B-25     |  2 | 5 |    161.1 ± 26.4 |   12.64 ± 1.93    |  82.93 ± 0.00  |      484 ± 78     |
| 27B-25     |  4 | 5 |    310.2 ± 29.2 |   12.92 ± 1.24    |  82.93 ± 0.00  |      935 ± 86     |
| 35B-A3B-25 |  1 | 5 |     73.3 ± 0.5  |   13.62 ± 0.12    |  82.54 ± 0.00  |      220 ± 2      |
| 35B-A3B-25 |  2 | 5 |    142.8 ± 0.4  |   13.98 ± 0.05    |  82.54 ± 0.00  |      429 ± 1      |

**VRAM footnote:** vLLM allocates a fixed fraction of total VRAM up front for KV cache + weights regardless of model size. Qwen3-VL family used `gpu_memory_utilization=0.9` (~78 GB reserved); the Qwen3.5-VL family used `0.95` (~83 GB reserved) because 35B-A3B's ~70 GB weights left so little KV cache headroom that vLLM refused to start at 0.9. So the "peak VRAM" column is **vLLM's reservation**, not the actual model footprint.

### Throughput vs batch size on vLLM (Qwen3-VL scales, Qwen3.5-VL plateaus) {#throughput-increases-with-batch-size-on-vllm-the-opposite-of-hf-padded-batching}

| B  | 4B  | 8B  | 32B | 9B-25 | 27B-25 | 35B-A3B-25 |
|---:|----:|----:|----:|------:|-------:|-----------:|
|  1 |  98 |  68 |  33 |    30 |     14 |         14 |
|  2 | 111 |  73 |  36 |    29 |     13 |         14 |
|  4 | 163 | 137 |  70 |    38 |     13 |          — |
|  8 | 216 | 175 |   — |    41 |      — |          — |
| 16 | 271 | 236 |   — |    43 |      — |          — |
| 32 | 309 |   — |   — |     — |      — |          — |

(All values in users/s.)

For the **Qwen3-VL family** (4B / 8B / 32B), vLLM scales nicely with batch size on **real, ragged single-user inputs** because its continuous batching avoids the padding waste that crippled the HF padded path. Per-batch-doubling efficiency stays above ~70% from B=1 all the way to B=32 on 4B.

For the **Qwen3.5-VL family** (9B-25 / 27B-25 / 35B-A3B-25), the picture is much weaker:
- **9B-25** scales from ~30 → ~43 users/s as B grows from 1 → 16 — only a 1.4× improvement, vs Qwen3-VL-8B's ~3.5× improvement over the same batch range.
- **27B-25** *peaks at B=1* (14 users/s) and slightly degrades at B=2 — batching doesn't help at all in vLLM 0.19 for this model.
- **35B-A3B-25** (MoE) is essentially flat at ~14 users/s for B=1 and B=2; we couldn't test higher because of VRAM limits.

This is the headline finding for the Qwen3.5-VL family: **vLLM 0.19's continuous-batching speedup over HF padded batching is much smaller for hybrid-linear-attention models than for dense full-attention models** (3–9× vs 5–18×). See the [Qwen3.5-VL vs Qwen3-VL discussion](#qwen35-vl-vs-qwen3-vl-discussion) for likely reasons.

### HF vs vLLM side-by-side (5-seed means) {#hf-vs-vllm-side-by-side-5-seed-means}

| model      | B  | HF wall (ms) | vLLM wall (ms) | **wall speedup** | HF thru (users/s) | vLLM thru (users/s) | **thru speedup** |
|-----------:|---:|-------------:|---------------:|-----------------:|------------------:|--------------------:|-----------------:|
| 4B         |  1 |        77.0  |          10.2  |        **7.5×**  |          12.99    |              98.09  |         **7.6×** |
| 4B         |  2 |       116.0  |          18.1  |        **6.4×**  |          17.83    |             110.89  |         **6.2×** |
| 4B         |  4 |       232.4  |          25.4  |        **9.2×**  |          17.50    |             162.61  |         **9.3×** |
| 4B         |  8 |       475.4  |          37.1  |       **12.8×**  |          16.88    |             215.88  |        **12.8×** |
| 4B         | 16 |       936.7  |          59.3  |       **15.8×**  |          17.15    |             270.86  |        **15.8×** |
| 4B         | 32 |      1918.4  |         103.9  |       **18.5×**  |          16.69    |             308.85  |        **18.5×** |
| 8B         |  1 |        81.2  |          14.8  |        **5.5×**  |          12.30    |              68.36  |         **5.6×** |
| 8B         |  2 |       145.1  |          27.7  |        **5.2×**  |          14.46    |              72.50  |         **5.0×** |
| 8B         |  4 |       303.6  |          29.4  |       **10.3×**  |          13.40    |             137.00  |        **10.2×** |
| 8B         |  8 |       624.9  |          45.8  |       **13.6×**  |          12.85    |             175.12  |        **13.6×** |
| 8B         | 16 |      1245.0  |          67.9  |       **18.3×**  |          12.89    |             235.89  |        **18.3×** |
| 32B        |  1 |       139.2  |          30.3  |        **4.6×**  |           7.32    |              32.92  |         **4.5×** |
| 32B        |  2 |       373.7  |          56.0  |        **6.7×**  |           5.81    |              35.62  |         **6.1×** |
| 32B        |  4 |       837.1  |          57.4  |       **14.6×**  |           4.89    |              69.58  |        **14.2×** |
| 9B-25      |  1 |       101.2  |          34.7  |        **2.9×**  |           9.84    |              29.65  |         **3.0×** |
| 9B-25      |  2 |       147.5  |          70.3  |        **2.1×**  |          14.02    |              28.62  |         **2.0×** |
| 9B-25      |  4 |       300.6  |         105.1  |        **2.9×**  |          13.50    |              38.37  |         **2.8×** |
| 9B-25      |  8 |       617.7  |         196.4  |        **3.1×**  |          12.98    |              40.73  |         **3.1×** |
| 9B-25      | 16 |      1235.6  |         376.4  |        **3.3×**  |          12.99    |              42.77  |         **3.3×** |
| 27B-25     |  1 |       170.1  |          71.7  |        **2.4×**  |           5.88    |              14.08  |         **2.4×** |
| 27B-25     |  2 |       319.3  |         161.1  |        **2.0×**  |           6.66    |              12.64  |         **1.9×** |
| 27B-25     |  4 |       695.1  |         310.2  |        **2.2×**  |           5.86    |              12.92  |         **2.2×** |
| 35B-A3B-25 |  1 |       534.2  |          73.3  |        **7.3×**  |           1.87    |              13.62  |         **7.3×** |
| 35B-A3B-25 |  2 |       609.7  |         142.8  |        **4.3×**  |           3.28    |              13.98  |         **4.3×** |

**vLLM is 4.5–18.5× faster than HF padded batching on the Qwen3-VL family**, but only **2–7× faster on the Qwen3.5-VL family**. The speedup grows with batch size on Qwen3-VL because HF's padding waste compounds linearly while vLLM's continuous batching has no waste. On Qwen3.5-VL the same effect exists but is muted: vLLM's wall time grows roughly linearly with batch (less batching gain), while HF's wall time scales similarly to the Qwen3-VL family. The result is that the speedup ratio is roughly constant (~2–3×) instead of growing.

35B-A3B-25 is an interesting outlier — vLLM gives a relatively large 7.3× speedup at B=1 but the absolute throughput is still only 14 users/s (the same as 27B-25 at B=1, despite having only 3 B active parameters). The MoE compute advantage doesn't show up in our setup.

### Where the speedup comes from {#where-the-speedup-comes-from}

1. **No padding waste** — biggest single factor at large batches. HF pads every sample to the longest in the batch; vLLM packs only the real tokens.
2. **FlashAttention 3** for the vision encoder — vLLM 0.19 dispatches FA3 kernels on H100 sm_90 (faster than `sdpa`'s cuDNN flash dispatch by ~30% for ViT shapes).
3. **`torch.compile` + CUDA graphs** — eliminate per-call Python and kernel-launch overhead, especially impactful at batch 1.
4. **Chunked prefill** — long prefill is overlapped with decode of other requests, hiding latency.
5. **Smaller per-call Python overhead** — vLLM's request serialization is much lighter than HF's per-sample processor call.

The HF vs vLLM speedup at B=1 (~5–7×) is **all** about (2)+(3)+(5) — there's no batching to take advantage of. The additional speedup at larger batches (~7× → ~18×) is the padding waste benefit (1).

### L40S extrapolation with vLLM (BF16, blended ×3.0) {#l40s-extrapolation-with-vllm-bf16-blended-30}

Single-GPU L40S serving expected throughput (HF / vLLM contrast):

| Workload             | HF L40S thru   | **vLLM L40S thru**   | gap     |
|----------------------|---------------:|---------------------:|--------:|
| 4B  B=1              |  ~4.3 users/s  |     **~32 users/s**  |  ~7.5×  |
| 4B  B=8              |  ~5.6 users/s  |     **~72 users/s**  | ~13×    |
| 4B  B=32             |  ~5.6 users/s  |    **~103 users/s**  | ~18×    |
| 8B  B=1              |  ~4.1 users/s  |     **~22 users/s**  |  ~5.4×  |
| 8B  B=8              |  ~4.3 users/s  |     **~58 users/s**  | ~14×    |
| 8B  B=16             |  ~4.3 users/s  |     **~79 users/s**  | ~18×    |
| 32B B=1\*            |  ~2.4 users/s  |     **~11 users/s**  |  ~4.6×  |
| 32B B=4\*            |  ~1.6 users/s  |     **~23 users/s**  | ~14×    |
| 9B-25 B=1            |  ~3.3 users/s  |     **~10 users/s**  |  ~3.0×  |
| 9B-25 B=8            |  ~4.3 users/s  |     **~14 users/s**  |  ~3.1×  |
| 9B-25 B=16           |  ~4.3 users/s  |     **~14 users/s**  |  ~3.3×  |
| 27B-25 B=1\*         |  ~2.0 users/s  |    **~4.7 users/s**  |  ~2.4×  |
| 27B-25 B=4\*         |  ~1.9 users/s  |    **~4.3 users/s**  |  ~2.2×  |
| 35B-A3B-25 B=1\*     |  ~0.6 users/s  |    **~4.5 users/s**  |  ~7.3×  |
| 35B-A3B-25 B=2\*     |  ~1.1 users/s  |    **~4.7 users/s**  |  ~4.3×  |

\* 32B / 27B-25 / 35B-A3B-25 in BF16 still do not fit on L40S 48 GB; numbers are reference-only. Quantization (FP8 / NF4) is required for actual deployment.

### Strong recommendation {#strong-recommendation}

**Use vLLM for production single-user concept extraction, not HF `model.forward()`.** The HF padded-batching path measured here is not what production should run — it's only useful as a baseline that shows what naive batching costs. The vLLM numbers are the realistic ceiling for what a single L40S can deliver:

- **Qwen3-VL-4B BF16 on L40S: ~32–103 users/s** (depending on batch size you're willing to wait for) — **best deployable point**
- **Qwen3-VL-8B BF16 on L40S: ~22–79 users/s**
- **Qwen3-VL-32B BF16 on L40S: does not fit** — needs quantization (FP8 fits at ~33 GB and L40S has FP8 tensor cores; sub-3-bit quants like AWQ fit easily and would deliver ~10–25 users/s)
- **Qwen3.5-VL-9B BF16 on L40S: ~10–14 users/s** — fits comfortably but ~3× slower than Qwen3-VL-8B
- **Qwen3.5-VL-27B BF16 on L40S: does not fit** — needs quantization; estimated ~5 users/s with quant
- **Qwen3.5-VL-35B-A3B BF16 on L40S: does not fit** — needs quantization; estimated ~5 users/s with quant. The MoE compute advantage didn't materialize in our setup

**The Qwen3-VL family (especially 4B and 8B) is the recommended choice for this workload.** The Qwen3.5-VL family is *not* a free upgrade — see the [Qwen3.5-VL vs Qwen3-VL discussion](#qwen35-vl-vs-qwen3-vl-discussion) below for why.

**One important loss with vLLM:** the SAE forward pass is not measured because vLLM doesn't expose hidden states. To use TopKSAE in a vLLM production path, you'd need either:
1. A vLLM patch that emits last-token hidden states (the SAE concept-extraction use case), or
2. A two-stage pipeline: vLLM for prefill+1step → extract last-token hidden via a custom head → run SAE separately. The SAE itself is < 0.5% of VLM time per the HF measurements.

## Production SAE early-exit measurements {#production-sae-early-exit-measurements}

The sections above measure **full forward**: every transformer layer runs, and the last-layer last-token hidden state is fed into the SAE. But the production SAE concept-extraction path doesn't use the last-layer hidden state. In cbm_probe ([extract_per_profile_sharded.py:46](../../../experiments/cbm_probe/extract_per_profile_sharded.py#L46)), the SAE reads the output of a **mid-network** transformer block:

- **Qwen3-VL-8B production**: SAE attached at **layer 18** (0-indexed; 36 total layers). Only layers 0..18 need to run — layers 19..35 are pure compute waste.
- **Qwen3-VL-32B production**: SAE attached at **layer 43** (0-indexed; 64 total layers). Only layers 0..43 need to run — layers 44..63 are pure compute waste.

To measure the production-realistic path we added a `--truncate-layers K` flag to both HF and vLLM bench scripts ([bench_sae_inference.py](../../bench_sae_inference.py), [vllm_bench_qwen3vl.py](../../vllm_bench_qwen3vl.py)). The flag physically shortens the text decoder to `K` transformer blocks so layers `K..N-1` are never allocated or executed. For HF, this is a one-line override: `AutoConfig.from_pretrained(...)` with a mutated `text_config.num_hidden_layers`, passed as `from_pretrained(config=...)`; HF silently drops the UNEXPECTED weight keys for the stripped layers. For vLLM, the same config override is not enough — the default loader streams every tensor from the safetensors shards and raises `KeyError` on weights for layers ≥ K. The workaround is to build a temporary model directory with **physically rewritten safetensors shards** that only contain the kept weights, plus a rewritten `model.safetensors.index.json` and `config.json`. See `_build_truncated_model_dir` in [vllm_bench_qwen3vl.py](../../vllm_bench_qwen3vl.py) for the full implementation.

We only re-ran the two production candidates (8B and 32B) under truncation. SGLang was **not re-measured** under truncation: its per-request IPC/CPU bottleneck (see [SGLang section](#the-smoking-gun-sglang-latency-is-essentially-independent-of-model-size)) dominates its wall time regardless of model compute, so truncation wouldn't move the SGLang needle meaningfully. Qwen3-VL-4B and the entire Qwen3.5-VL family were also not re-measured (not production candidates).

### Per-config truncated H100 measurements (5 seeds) {#per-config-truncated-h100-measurements-5-seeds}

**HF, truncated** ([bench_truncated19_20260409T143300Z](bench_truncated19_20260409T143300Z_NVIDIA_H100_80GB_HBM3.json), [bench_truncated44_20260409T143851Z](bench_truncated44_20260409T143851Z_NVIDIA_H100_80GB_HBM3.json)):

| model            | B  | n | vlm p50 (ms)    | sae p50 (ms)  | thru (users/s) | peak VRAM (GB) |
|-----------------:|---:|--:|----------------:|--------------:|---------------:|---------------:|
| 8B trunc=19      |  1 | 5 |    60.4 ± 4.3   | 0.245 ± 0.003 |  16.53 ± 1.24  |  11.67 ± 0.09  |
| 8B trunc=19      |  2 | 5 |   105.2 ± 23.6  | 0.269 ± 0.004 |  19.68 ± 3.99  |  12.45 ± 0.38  |
| 8B trunc=19      |  4 | 5 |   211.7 ± 29.3  | 0.317 ± 0.004 |  19.19 ± 2.82  |  14.02 ± 0.43  |
| 8B trunc=19      |  8 | 5 |   433.1 ± 26.2  | 0.389 ± 0.012 |  18.51 ± 1.06  |  17.13 ± 0.44  |
| 8B trunc=19      | 16 | 5 |   863.9 ± 60.4  | 0.411 ± 0.010 |  18.58 ± 1.24  |  23.05 ± 0.83  |
| 32B trunc=44     |  1 | 5 |   109.0 ± 16.2  | 0.284 ± 0.004 |   9.32 ± 1.39  |  48.25 ± 0.14  |
| 32B trunc=44     |  2 | 5 |   276.2 ± 87.6  | 0.351 ± 0.004 |   7.79 ± 2.23  |  49.57 ± 0.65  |
| 32B trunc=44     |  4 | 5 |   614.1 ± 99.0  | 0.395 ± 0.005 |   6.65 ± 1.13  |  52.26 ± 0.74  |

**vLLM, truncated** ([vllm_bench_truncated19_20260409T150028Z](vllm_bench_truncated19_20260409T150028Z_NVIDIA_H100_80GB_HBM3.json), [vllm_bench_truncated44_20260409T150601Z](vllm_bench_truncated44_20260409T150601Z_NVIDIA_H100_80GB_HBM3.json)):

| model            | B  | n | wall p50 (ms)  | thru (users/s)     | peak VRAM (GB) |
|-----------------:|---:|--:|---------------:|-------------------:|---------------:|
| 8B trunc=19      |  1 | 5 |    9.8 ± 1.0   |  101.30 ± 10.28    |  77.65 ± 0.00  |
| 8B trunc=19      |  2 | 5 |   16.9 ± 1.2   |  117.71 ±  9.99    |  77.65 ± 0.00  |
| 8B trunc=19      |  4 | 5 |   24.4 ± 4.2   |  167.35 ± 30.07    |  77.65 ± 0.00  |
| 8B trunc=19      |  8 | 5 |   38.8 ± 5.1   |  207.65 ± 27.18    |  77.65 ± 0.00  |
| 8B trunc=19      | 16 | 5 |   56.7 ± 2.6   |  280.34 ± 12.62    |  77.65 ± 0.00  |
| 32B trunc=44     |  1 | 5 |   23.2 ± 0.9   |   42.85 ±  1.54    |  77.64 ± 0.00  |
| 32B trunc=44     |  2 | 5 |   41.3 ± 0.6   |   48.41 ±  0.64    |  77.64 ± 0.00  |
| 32B trunc=44     |  4 | 5 |   42.9 ± 1.1   |   92.35 ±  4.56    |  77.64 ± 0.00  |

**vLLM peak VRAM footnote (same as full-forward):** vLLM reserves a fixed `gpu_memory_utilization` fraction of total VRAM up front for KV cache + weights. The 77.65 GB figure is vLLM's reservation, not the live footprint. The **live weight footprint** is what matters for the L40S fit story, and vLLM's loader prints it at startup:
- 8B trunc=19: **~8.3 GiB** loaded (vs 8B full ~17 GB)
- 32B trunc=44: **~44.7 GiB** loaded (vs 32B full ~66 GB) — see the log excerpt below for the exact line.

```text
[gpu_model_runner.py:4820] Model loading took 44.7 GiB memory and 12.17 seconds
```

For the HF runs, `torch.cuda.max_memory_allocated` is what appears in the `peak VRAM` column above, and includes weights + activations + padding. The HF truncated-32B VRAM peak of **48.25 GB at B=1** is the production-realistic live number and is the one that flips the L40S fit story — see below.

### Side-by-side: full vs truncated (5-seed means) {#side-by-side-full-vs-truncated}

**HF, Qwen3-VL-8B:**

| B  | full wall (ms) | trunc wall (ms) | **Δ wall** | full thru | trunc thru | **Δ thru**  | full VRAM (GB) | trunc VRAM (GB) | **Δ VRAM**  |
|---:|---------------:|----------------:|-----------:|----------:|-----------:|------------:|---------------:|----------------:|------------:|
|  1 |          81.2  |           60.4  | **-25.6%** |    12.30  |     16.53  | **+34.5%**  |         18.38  |          11.67  | **-36.5%**  |
|  2 |         145.1  |          105.2  | **-27.5%** |    14.46  |     19.68  | **+36.1%**  |         19.44  |          12.45  | **-36.0%**  |
|  4 |         303.6  |          211.7  | **-30.3%** |    13.40  |     19.19  | **+43.2%**  |         21.59  |          14.02  | **-35.1%**  |
|  8 |         624.9  |          433.1  | **-30.7%** |    12.85  |     18.51  | **+44.1%**  |         25.86  |          17.13  | **-33.8%**  |
| 16 |        1245.0  |          863.9  | **-30.6%** |    12.89  |     18.58  | **+44.1%**  |         34.00  |          23.05  | **-32.2%**  |

**HF, Qwen3-VL-32B:**

| B  | full wall (ms) | trunc wall (ms) | **Δ wall** | full thru | trunc thru | **Δ thru**  | full VRAM (GB) | trunc VRAM (GB) | **Δ VRAM**  |
|---:|---------------:|----------------:|-----------:|----------:|-----------:|------------:|---------------:|----------------:|------------:|
|  1 |         139.2  |          109.0  | **-21.8%** |     7.32  |      9.32  | **+27.3%**  |         67.94  |          48.25  | **-29.0%**  |
|  2 |         373.7  |          276.2  | **-26.1%** |     5.81  |      7.79  | **+34.2%**  |         69.66  |          49.57  | **-28.8%**  |
|  4 |         837.1  |          614.1  | **-26.6%** |     4.89  |      6.65  | **+36.1%**  |         73.16  |          52.26  | **-28.6%**  |

**vLLM, Qwen3-VL-8B:**

| B  | full wall (ms) | trunc wall (ms) | **Δ wall** | full thru | trunc thru | **Δ thru**  |
|---:|---------------:|----------------:|-----------:|----------:|-----------:|------------:|
|  1 |          14.8  |            9.8  | **-33.6%** |    68.36  |    101.30  | **+48.2%**  |
|  2 |          27.7  |           16.9  | **-38.9%** |    72.50  |    117.71  | **+62.4%**  |
|  4 |          29.4  |           24.4  | **-17.0%** |   137.00  |    167.35  | **+22.2%**  |
|  8 |          45.8  |           38.8  | **-15.3%** |   175.12  |    207.65  | **+18.6%**  |
| 16 |          67.9  |           56.7  | **-16.5%** |   235.89  |    280.34  | **+18.8%**  |

**vLLM, Qwen3-VL-32B:**

| B  | full wall (ms) | trunc wall (ms) | **Δ wall** | full thru | trunc thru | **Δ thru**  |
|---:|---------------:|----------------:|-----------:|----------:|-----------:|------------:|
|  1 |          30.3  |           23.2  | **-23.4%** |    32.92  |     42.85  | **+30.2%**  |
|  2 |          56.0  |           41.3  | **-26.4%** |    35.62  |     48.41  | **+35.9%**  |
|  4 |          57.4  |           42.9  | **-25.4%** |    69.58  |     92.35  | **+32.7%**  |

**What the numbers say:**
- **HF wall-time ratio** (trunc/full) is 70–78%, close to but slightly higher than the pure layer ratio (19/36 = 53% for 8B, 44/64 = 69% for 32B). The vision encoder is a constant overhead that doesn't shrink under truncation, so the wall-time ratio is always higher than the layer ratio. For 32B the gap is smaller because the vision encoder is a smaller fraction of total compute at 64 layers.
- **vLLM wall-time ratio** is similar (62–85% depending on batch size), but with more noise at small batches because vLLM's per-call CUDA-graph + compile overhead is a larger fraction of the shorter wall time.
- **HF VRAM ratio** is 63–71%, matching the **model-weight** ratio (8B trunc=19 keeps 19/36 = 53% of the decoder weights, plus a fixed-size vision encoder + embeddings; 32B trunc=44 keeps 44/64 = 69% of the decoder weights). **vLLM VRAM** is the reservation and doesn't change with truncation (already at ~78 GB), so the HF VRAM column is the one to read for L40S fit.
- **Throughput speedup exceeds wall-time savings** in most cells because the shorter wall time on the critical path (and a slightly smaller VRAM footprint that allows more cache) compounds.

### Updated L40S extrapolation — truncated is the production baseline {#l40s-extrapolation-truncated-production-baseline}

Applying the same blended ×3.0 L40S extrapolation to the truncated H100 wall times:

| workload (vLLM)          | best B | H100 thru  | L40S wall (ms) | **L40S thru (users/s)** | BF16 fits L40S 48 GB?                    |
|--------------------------|-------:|-----------:|---------------:|------------------------:|------------------------------------------|
| 4B full                  |  32    |  309       |     311        |   ~103                  | Yes (fits comfortably)                   |
| **8B trunc=19**          |  16    |  **280**   |     **170**    |   **~94**               | **Yes** (load ~8.3 GiB, ~40 GB KV cache) |
| **8B trunc=19**          |  1     |  **101**   |     **29**     |   **~34**               | Yes                                       |
| 8B full                  |  16    |  236       |     204        |   ~79                   | Yes (load ~17 GB, ~31 GB KV cache)       |
| **32B trunc=44**         |  4     |  **92**    |     **129**    |   **~31**               | **Barely** (load ~44.7 GiB, ~3 GB KV cache) |
| **32B trunc=44**         |  1     |  **43**    |     **70**     |   **~14**               | **Barely** (same as above)               |
| 32B full                 |  4     |  70        |     172        |   ~23                   | **No** (weights alone = 66 GB > 48 GB)   |

(HF-only results for completeness: 8B trunc=19 best is B=2 → ~6.3 users/s on L40S; 32B trunc=44 best is B=1 → ~3.1 users/s on L40S. As in the full-forward L40S analysis, HF is strictly worse than vLLM at these scales.)

**32B trunc=44 is the qualitative flip**: the full-32B story has always been "does not fit L40S BF16, must quantize". With truncation, the live weight footprint drops to ~44.7 GiB, which fits under the 48 GB L40S ceiling with **~3 GB of headroom for KV cache + activations**. That's tight — it realistically only supports B=1 and minimal KV growth — but it is a **BF16-deployable path** where previously there was none.

### L40S production sizing — truncated {#l40s-production-sizing-truncated}

**If 8B accuracy is acceptable for production**, the truncated 8B is the clear sweet spot:

- **Qwen3-VL-8B trunc=19 + vLLM, L40S BF16**: **~34 users/s (B=1)** to **~94 users/s (B=16)**. Comfortable VRAM headroom (~40 GB of KV cache space), fits without any quantization work, and beats the full-forward 8B L40S estimate by 19% at B=16 (94 vs 79 users/s). This is the top recommended candidate for production.

**If 32B accuracy is required**, the truncated 32B is the new BF16 candidate:

- **Qwen3-VL-32B trunc=44 + vLLM, L40S BF16**: **~14 users/s (B=1)** to **~31 users/s (B=4)**. The fit is tight (only ~3 GB of non-weight headroom on L40S at B=1), so production needs either (a) a B=1-only serving config with minimal KV cache, or (b) FP8 / NF4 quantization stacked on top of truncation for batching room. FP8 of the truncated 32B would drop weight footprint to ~22 GB and leave ~26 GB for KV cache, which is plenty for any batch size we tested.

The full-forward L40S recommendations in [Strong recommendation](#strong-recommendation) remain valid as an upper-bound / published-benchmark comparison, but the production sizing decision should be based on the truncated numbers above.

## SGLang comparison (and a surprising bottleneck) {#sglang-comparison-and-a-surprising-bottleneck}

Same 5 seeds, same single-user sample windows, same six models in BF16. Different framework: **SGLang 0.5.10** offline `Engine.generate(max_new_tokens=1)`. Implementation: [profile-ranker-face/sglang_bench_qwen3vl.py](../../sglang_bench_qwen3vl.py).

### Per-config SGLang measurements (5 seeds) {#per-config-sglang-measurements-5-seeds}

| model      | B  | n | wall p50 (ms)    | thru (users/s)  | peak VRAM (GB) | L40S~ms (blended) |
|-----------:|---:|--:|-----------------:|----------------:|---------------:|------------------:|
| 4B         |  1 | 5 |    109.4 ± 28.6  |   9.76 ± 2.93   |  74.15 ± 0.22  |       327 ± 84    |
| 4B         |  2 | 5 |    232.8 ± 46.0  |   8.86 ± 1.49   |  74.22 ± 0.13  |       695 ± 132   |
| 4B         |  4 | 5 |    441.7 ± 55.7  |   9.16 ± 1.08   |  74.22 ± 0.13  |      1325 ± 161   |
| 4B         |  8 | 5 |    880.0 ± 63.0  |   9.13 ± 0.70   |  74.24 ± 0.10  |      2639 ± 189   |
| 4B         | 16 | 5 |   1735.7 ± 124.5 |   9.06 ± 0.73   |  74.24 ± 0.09  |      5326 ± 418   |
| 4B         | 32 | 5 |   3479.0 ± 153.9 |   9.23 ± 0.42   |  74.26 ± 0.06  |     10415 ± 476   |
| 8B         |  1 | 5 |    110.4 ± 28.7  |   9.71 ± 2.88   |  73.79 ± 0.28  |       329 ± 85    |
| 8B         |  2 | 5 |    234.9 ± 45.4  |   8.75 ± 1.49   |  73.91 ± 0.17  |       704 ± 134   |
| 8B         |  4 | 5 |    445.8 ± 53.6  |   9.10 ± 1.03   |  73.91 ± 0.17  |      1332 ± 155   |
| 8B         |  8 | 5 |    883.7 ± 51.9  |   9.06 ± 0.57   |  73.91 ± 0.17  |      2657 ± 160   |
| 8B         | 16 | 5 |   1746.8 ± 137.1 |   9.06 ± 0.85   |  73.93 ± 0.15  |      5333 ± 507   |
| 32B        |  1 | 5 |    123.3 ± 28.5  |   8.56 ± 2.18   |  73.94 ± 0.30  |       367 ± 82    |
| 32B        |  2 | 5 |    249.5 ± 44.4  |   8.21 ± 1.35   |  74.08 ± 0.14  |       748 ± 136   |
| 32B        |  4 | 5 |    458.7 ± 58.8  |   8.83 ± 1.08   |  74.08 ± 0.14  |      1376 ± 177   |
| 9B-25      |  1 | 5 |    131.8 ± 30.1  |   8.04 ± 2.16   |  74.95 ± 0.49  |       392 ± 90    |
| 9B-25      |  2 | 5 |    255.1 ± 44.4  |   8.03 ± 1.27   |  74.98 ± 0.46  |       764 ± 135   |
| 9B-25      |  4 | 5 |    467.0 ± 54.6  |   8.68 ± 0.98   |  74.98 ± 0.46  |      1398 ± 161   |
| 9B-25      |  8 | 5 |    908.0 ± 63.0  |   8.85 ± 0.60   |  75.00 ± 0.42  |      2723 ± 175   |
| 9B-25      | 16 | 5 |   1943.3 ± 137.3 |   8.17 ± 0.60   |  75.19 ± 0.11  |      5898 ± 437   |
| 27B-25     |  1 | 5 |    172.9 ± 30.3  |   5.93 ± 1.10   |  73.44 ± 0.15  |       518 ± 88    |
| 27B-25     |  2 | 5 |    327.8 ± 51.4  |   6.20 ± 0.90   |  73.50 ± 0.14  |       985 ± 157   |
| 27B-25     |  4 | 5 |    566.2 ± 50.5  |   7.11 ± 0.62   |  73.51 ± 0.13  |      1699 ± 153   |
| 35B-A3B-25 |  1 | 5 |    164.7 ± 27.5  |   6.25 ± 1.16   |  73.25 ± 0.08  |       493 ± 84    |
| 35B-A3B-25 |  2 | 5 |    302.2 ± 39.6  |   6.69 ± 0.82   |  73.28 ± 0.06  |       908 ± 119   |

### The smoking gun: SGLang latency is essentially independent of model size {#the-smoking-gun-sglang-latency-is-essentially-independent-of-model-size}

Look at the `wall p50` column at B=1 across all six models:
- 4B:           **109.4 ms**
- 8B:           **110.4 ms**
- 32B:          **123.3 ms**
- 9B-25:        **131.8 ms**
- 27B-25:       **172.9 ms**
- 35B-A3B-25:   **164.7 ms**

These six wall times span only ~60 ms from smallest to largest, despite the largest model (35B-A3B-25) having **~9× more parameters** than the smallest (4B). **The GPU compute can't possibly be the bottleneck** — running 9× more GEMM work in roughly the same wall time is impossible. Compare to vLLM where 35B-A3B-25 at B=1 is 73.3 ms vs 4B's 10.2 ms = ~7× slower, which is closer to the rough compute ratio.

Same pattern at B=4:
- 4B:           442 ms
- 8B:           446 ms
- 32B:          459 ms
- 9B-25:        467 ms
- 27B-25:       566 ms

Again the spread is small relative to the compute differences. SGLang's wall time is dominated by something **outside the GPU forward pass**.

Throughput is also nearly constant ~6–9 users/s across all configs (model size, batch size). vLLM's throughput grows with batch (up to 309 users/s for 4B B=32). SGLang's doesn't grow with batch — meaning **batching isn't helping**. The pattern is identical for the Qwen3.5-VL family: same wall time, same flat-with-batch throughput.

### What's happening? {#whats-happening}

The most likely explanation: SGLang's offline `Engine.generate()` has a per-request CPU/IPC bottleneck in the multimodal path that dominates GPU compute for our workload. Candidates:

1. **Image preprocessing in the tokenizer manager subprocess.** SGLang's offline `Engine` runs a tokenizer manager in a separate process. Multimodal inputs go through `processor_image()` calls that may not be GPU-accelerated and may not parallelize across the per-request images. Each generate() call serializes images over an IPC pipe, decodes them in the tokenizer manager, runs processing, then forwards to the scheduler. We measured both PIL pickling and file-path approaches; both gave the same baseline.
2. **Vision-encoder kernel selection.** Our **published** SGLang runs all use `mm_attention_backend="fa3"` (the bench script default — see [sglang_bench_qwen3vl.py:236-238](../../sglang_bench_qwen3vl.py#L236-L238)). The SGLang library's own default is `None` (no override). On top of the script default, we tried adding `keep_mm_feature_on_device=True` as an ad-hoc tuning experiment, and it made things **worse** (the only setting we tried that hurt things rather than helping). The published numbers are the fa3 baseline, **not** the worse keep_mm_feature_on_device run.
3. **Per-request scheduler overhead.** SGLang's scheduler runs an event loop that polls for requests. The polling interval and queue handling adds latency that's roughly fixed per request and doesn't shrink with model size.
4. **`cutlass.cute.experimental` warning at engine init** — SGLang prints `Unexpected error during package walk: cutlass.cute.experimental`. Some kernel path may have failed to load. We did not investigate further.

### Three-framework side-by-side (single-user wall p50 in ms) {#three-framework-side-by-side}

| model      | B  | HF (ms)   | vLLM (ms) | SGLang (ms) | vLLM speedup vs HF | SGLang speedup vs HF |
|-----------:|---:|----------:|----------:|------------:|-------------------:|---------------------:|
| 4B         |  1 |     77.0  |    10.2   |     109.4   |     **7.5×**       |        **0.70×**     |
| 4B         |  2 |    116.0  |    18.1   |     232.8   |     **6.4×**       |        **0.50×**     |
| 4B         |  4 |    232.4  |    25.4   |     441.7   |     **9.2×**       |        **0.53×**     |
| 4B         |  8 |    475.4  |    37.1   |     880.0   |    **12.8×**       |        **0.54×**     |
| 4B         | 16 |    936.7  |    59.3   |    1735.7   |    **15.8×**       |        **0.54×**     |
| 4B         | 32 |   1918.4  |   103.9   |    3479.0   |    **18.5×**       |        **0.55×**     |
| 8B         |  1 |     81.2  |    14.8   |     110.4   |     **5.5×**       |        **0.74×**     |
| 8B         |  2 |    145.1  |    27.7   |     234.9   |     **5.2×**       |        **0.62×**     |
| 8B         |  4 |    303.6  |    29.4   |     445.8   |    **10.3×**       |        **0.68×**     |
| 8B         |  8 |    624.9  |    45.8   |     883.7   |    **13.6×**       |        **0.71×**     |
| 8B         | 16 |   1245.0  |    67.9   |    1746.8   |    **18.3×**       |        **0.71×**     |
| 32B        |  1 |    139.2  |    30.3   |     123.3   |     **4.6×**       |        **1.13×**     |
| 32B        |  2 |    373.7  |    56.0   |     249.5   |     **6.7×**       |        **1.50×**     |
| 32B        |  4 |    837.1  |    57.4   |     458.7   |    **14.6×**       |        **1.83×**     |
| 9B-25      |  1 |    101.2  |    34.7   |     131.8   |     **2.9×**       |        **0.77×**     |
| 9B-25      |  2 |    147.5  |    70.3   |     255.1   |     **2.1×**       |        **0.58×**     |
| 9B-25      |  4 |    300.6  |   105.1   |     467.0   |     **2.9×**       |        **0.64×**     |
| 9B-25      |  8 |    617.7  |   196.4   |     908.0   |     **3.1×**       |        **0.68×**     |
| 9B-25      | 16 |   1235.6  |   376.4   |    1943.3   |     **3.3×**       |        **0.64×**     |
| 27B-25     |  1 |    170.1  |    71.7   |     172.9   |     **2.4×**       |        **0.98×**     |
| 27B-25     |  2 |    319.3  |   161.1   |     327.8   |     **2.0×**       |        **0.97×**     |
| 27B-25     |  4 |    695.1  |   310.2   |     566.2   |     **2.2×**       |        **1.23×**     |
| 35B-A3B-25 |  1 |    534.2  |    73.3   |     164.7   |     **7.3×**       |        **3.24×**     |
| 35B-A3B-25 |  2 |    609.7  |   142.8   |     302.2   |     **4.3×**       |        **2.02×**     |

Notably:
- **vLLM is ~2–18.5× faster than HF** across the board, with the higher end on Qwen3-VL family and the lower end on Qwen3.5-VL family.
- **SGLang is faster than HF only at 32B and 35B-A3B-25** (because HF's padding waste catches up at large models with ragged inputs). For 4B / 8B / 9B-25 / 27B-25, SGLang is slower or comparable to the naive HF padded path at every batch size.
- **SGLang's throughput is ~6–9 users/s for everything** — model and batch size barely matter.

### Where SGLang stands {#where-sglang-stands}

This benchmark is a single configuration of SGLang. Reasons the result might not generalize:
1. **We used offline `Engine.generate()`.** SGLang's primary deployment mode is the HTTP server (`sglang.launch_server`), which has a different request lifecycle and may not have the per-request overhead we see here. We did not benchmark the HTTP server.
2. **Qwen3-VL support in SGLang is recent** ([PR #15320](https://github.com/sgl-project/sglang/pull/15320)). Vision-encoder optimizations like ViT piecewise CUDA graph were added in late 2025/early 2026 and may not be fully on by default.
3. **Our prompt template comes from HF processor.apply_chat_template**, which produces vision-token placeholders that SGLang re-processes. SGLang has its own chat template handling that may be more efficient for first-class supported models.
4. **The `cutlass.cute.experimental` warning** at engine init suggests one kernel path failed to load.

Recommendation if you want to seriously benchmark SGLang for this workload: file an issue with [sgl-project/sglang](https://github.com/sgl-project/sglang/) describing the offline Qwen3-VL latency we measured, OR switch to the HTTP server mode which is the production path.

### What we conclude from this side-by-side {#what-we-conclude-from-this-side-by-side}

**For the SAE + Qwen VL single-user concept-extraction sizing question:**
- vLLM remains the clear winner. The L40S sizing recommendation in the vLLM section stands.
- SGLang in our configuration is **worse than HF padded batching** for 4B/8B/9B-25/27B-25, which contradicts SGLang's general reputation. This means our SGLang setup is not representative — don't take the "SGLang is slow" framing at face value. The honest framing is: **we didn't get SGLang configured correctly for offline VLM inference, and a correct configuration may match or exceed vLLM. We just couldn't find it within the time budget.**

## Qwen3.5-VL vs Qwen3-VL discussion {#qwen35-vl-vs-qwen3-vl-discussion}

The Qwen3.5-VL family was added to this benchmark to see if the newer architecture (hybrid linear+full attention; MoE in the 35B-A3B variant) would be cheaper to run for our single-user concept-extraction workload. **The answer, as measured today, is no.** Across all three frameworks, Qwen3.5-VL is slower than the equivalent Qwen3-VL model, often by a large margin.

### What the numbers say {#what-the-numbers-say}

Best per-framework throughput per model (5-seed mean, real single-user inputs, BF16):

| Framework | 4B  | 8B  | 32B | 9B-25 | 27B-25 | 35B-A3B-25 |
|-----------|----:|----:|----:|------:|-------:|-----------:|
| HF        |17.8 |14.5 | 7.3 |  14.0 |    6.7 |        3.3 |
| vLLM      | 309 | 236 |  70 |    43 |     14 |         14 |
| SGLang    | 9.8 | 9.7 | 8.8 |   8.9 |    7.1 |        6.7 |

**Headline observations:**
1. **9B-25 is slower than 8B on every framework**, despite having only ~12% more parameters. HF: 14.0 vs 14.5 users/s. vLLM: 43 vs 236 users/s (5.5× gap!). SGLang: 8.9 vs 9.7. The gap is largest in vLLM — exactly the framework we want to use in production.
2. **27B-25 is comparable to 32B in raw wall time** (both ~14 users/s in vLLM at their best batch) despite having ~16% fewer parameters. The Qwen3-VL family edges ahead at higher batch sizes (32B B=4 = 70 users/s vs 27B-25 staying at 14 — 5× gap).
3. **35B-A3B-25 (MoE) shows essentially no compute advantage from its 3 B active parameters.** It tops out at 14 users/s in vLLM, the same as 27B-25 and only marginally better than 32B at B=1 (33 users/s). The MoE routing saves compute but the wall-time benefit is invisible in our setup. (See the MoE-memory note in the [Critical: does not fit](#critical-three-models-do-not-fit-on-l40s-in-bf16) section: MoE saves *compute* per token but all 70 GB of weights still have to be in VRAM, so it doesn't help with the L40S fit problem either.)
4. **VRAM overhead is also worse:** 9B-25 B=1 uses 19.7 GB vs 8B's 18.4 GB; 27B-25 uses 56 GB vs 32B's 68 GB (the only place 27B-25 wins meaningfully). 35B-A3B-25 uses 71 GB at B=1 alone — barely 9 GB headroom on H100 80 GB even at the smallest batch.

### Why is Qwen3.5-VL slower today? {#why-is-qwen35-vl-slower-today}

**It is not a missing-Flash-Attention bug.** We verified this directly from vLLM's debug log on the 9B-25 / 27B-25 / 35B-A3B-25 runs. With our default config, vLLM 0.19 picks **FlashAttention 3 for the 1-in-4 full-attention layers** and **FlashInfer GDN prefill kernel for the 3-in-4 linear-attention layers** — both are real fused fast-path kernels, not Triton fallbacks:

```text
[platforms/cuda.py:334]    Using FLASH_ATTN attention backend out of potential
                            backends: ['FLASH_ATTN', 'FLASHINFER', 'TRITON_ATTN', 'FLEX_ATTENTION']
[v1/attention/backends/flash_attn.py:596]   Using FlashAttention version 3
[layers/mamba/gdn_linear_attn.py:139]       Using FlashInfer GDN prefill kernel
[layers/mamba/gdn_linear_attn.py:140]       FlashInfer GDN prefill kernel is JIT-compiled;
                                            first run may take a while to compile.
                                            Set `--gdn-prefill-backend triton` to avoid JIT compile time.
```

So the dispatch is correct on both layer types. **The slowness is not in raw FLOP count — it is in the achieved MFU on the Mamba SSM kernel and the framework's prefill algorithm choice.** The dominant contributors, in descending order of confidence:

1. **The 3-in-4 "linear attention" layers are actually a Mamba-family SSM (Gated DeltaNet), not a generic linear attention block.** vLLM puts the kernel under [`vllm/model_executor/layers/mamba/gdn_linear_attn.py`](https://github.com/vllm-project/vllm/blob/v0.19.0/vllm/model_executor/layers/mamba/gdn_linear_attn.py) — same family as Mamba/Mamba2. Each GDN layer does input projections → causal Conv1D → an SSM scan (gated delta-rule recurrence) → output projection + gating. The "linear attention core" is just the recurrent state update written in linear-attention notation; it is not an attention layer in the classical sense.
2. **Raw compute is *not* the problem — in fact the SSM core is cheaper than full attention at our prefill length.** Per layer, with d_model = 4096 and head_dim h = 128 (so H = 32 heads):

   | layer type | core compute | 9B-25 at N = 926 |
   |---|---|---|
   | Full attention (Q K^T + softmax(·) V) | 4·N²·d | ~14.0 GFLOP |
   | Linear-attention / SSM core (K^Tᵀ V then Q·(·)) | 4·N·d·h = 4·N·d²/H | **~1.94 GFLOP** |

   Counter-intuitively, the SSM "core" is **~7.2× cheaper** than full attention here (= N/h = 926/128), not more expensive. Crossover happens at **N ≈ h** (~128), not at N ≈ d. Our N = 926 is well past that, but linear attention still wins on FLOPs at this size — projections dominate both layer types and the cores are a small fraction of total compute either way. **An earlier revision of this report claimed the opposite (linear attention is ~3.5× more expensive at our N), based on conflating d_model with head_dim. That claim was incorrect** — see CHANGELOG.

3. **The real problem is the achieved MFU on the SSM scan, driven by vLLM's prefill algorithm choice.** Mamba-family SSMs admit *two* prefill algorithms with the same mathematical result:
   - **Parallel scan** (work-efficient prefix scan, O(N log N) work / O(log N) depth) — used in training frameworks like Megatron-Mamba and the original Mamba reference implementation. Saturates GPU compute.
   - **Chunked-recurrent** (chunks computed in parallel, state passed sequentially between chunks) — vLLM 0.19's GDN prefill kernel. Lower achievable parallelism, dominated by the sequential depth N/chunk_size.

   The maintainer comment in [vllm-project/vllm#36627](https://github.com/vllm-project/vllm/issues/36627) makes the framing explicit: *"Standard transformer attention during prefill is embarrassingly parallel ... Mamba/DeltaNet layers are fundamentally recurrent: they need to sequentially update a hidden state token by token."* On dense attention vLLM gets ~80% MFU from FA3; on the GDN/Mamba prefill path it gets a much lower fraction because the kernel is chunked-recurrent rather than fully parallel. **This is a vLLM kernel-maturity issue**, not an architectural defect — the parallel-scan algorithm exists in the Mamba reference code but hasn't been integrated into vLLM 0.19's GDN prefill path.

4. **This explains the "training fast, inference slow" puzzle.** The same Mamba/DeltaNet architecture trains efficiently because Megatron-Mamba uses the parallel-scan algorithm and saturates the GPU. In vLLM the inference path falls back to chunked-recurrent and pays a 5–10× MFU penalty. This is a *framework* limitation, not an *architecture* limitation — when (and if) vLLM ships a fully parallel SSM prefill kernel, this gap should largely close for prefill-heavy workloads like ours.

5. **Missing `causal-conv1d` adds extra slowdown.** Each GDN layer has a 1-D causal convolution that normally uses the optimized [`causal-conv1d`](https://github.com/Dao-AILab/causal-conv1d) CUDA kernel. We could not build it on hyperpod (NameError on `bare_metal_version` during the wheel build), so the GDN path falls back to a slower PyTorch implementation. This adds an unmeasured percentage on top of the SSM-scan bottleneck above.

6. **CPU-side launch overhead amplifies the effect at small batches.** [vllm-project/vllm#27222](https://github.com/vllm-project/vllm/issues/27222) (direct quote): *"The main reason GDN — we spend more time in CPU than GPU here."* The chunked-recurrent prefill kernel makes many small dispatches per layer, each carrying CPU launch overhead. Larger CUDA-graph capture sizes can fold some of these into a single graph and recover a few percent at small batches (see "Tuning attempts" below) — but the underlying sequential-depth bottleneck stays.

7. **MoE 35B-A3B does not help, for the same reason.** The MoE design saves *parameter count* per token but the SSM layers are still 75% of the model and still go through the same chunked-recurrent prefill path. The 3 B activated parameters cannot shorten the SSM sequential depth. Our measured 35B-A3B throughput (~14 users/s) is identical to dense 27B-25, confirming that the bottleneck is shared. MoE is also a memory loser: all 70 GB of weights stay in VRAM regardless (you don't know which experts a request will need until routing happens).

### Tuning attempts: cuda-graph capture, expert-parallel, flashinfer-moe-fp16 {#tuning-attempts}

Following [vllm-project/vllm#27222](https://github.com/vllm-project/vllm/issues/27222)'s recommendation (which reported ~50% throughput improvement on Qwen3-Next decode workloads with `--cuda-graph-sizes=1024 --enable-expert-parallel VLLM_USE_FLASHINFER_MOE_FP16=1`), we re-ran each Qwen3.5-VL model with these flags to see whether they close the gap with Qwen3-VL on **our** prefill-dominated workload. They do not.

5-seed mean throughput, vLLM, single-user inputs (`req/s` = `users/s`):

| model      | B  | untuned thru   | tuned thru     |   Δ   |
|-----------:|---:|---------------:|---------------:|------:|
| 9B-25      |  1 |  29.65 ± 6.46  |  31.18 ± 3.91  | +5.2% |
| 9B-25      | **2** |  28.62 ± 2.68  | **31.26 ± 2.23**  | **+9.2%** |
| 9B-25      |  4 |  38.37 ± 3.85  |  39.17 ± 3.70  | +2.1% |
| 9B-25      |  8 |  40.73 ± 2.07  |  41.33 ± 1.97  | +1.5% |
| 9B-25      | 16 |  42.77 ± 2.86  |  42.36 ± 2.59  | -1.0% |
| 27B-25     |  1 |  14.08 ± 1.62  |  13.47 ± 1.38  | -4.3% |
| 27B-25     |  2 |  12.64 ± 1.93  |  12.38 ± 1.97  | -2.1% |
| 27B-25     |  4 |  12.92 ± 1.24  |  12.79 ± 1.24  | -1.0% |
| 35B-A3B-25 |  1 |  13.62 ± 0.12  |  13.14 ± 0.02  | -3.5% |
| 35B-A3B-25 |  2 |  13.98 ± 0.05  |  13.50 ± 0.06  | -3.4% |

`tuned` configurations:
- **9B-25**: `--cuda-graph-max-size 1024`
- **27B-25**: `--cuda-graph-max-size 512` (1024 OOMs the KV cache budget for the 54 GB weights)
- **35B-A3B-25**: `--cuda-graph-max-size 256 --enable-expert-parallel --use-flashinfer-moe-fp16`

**What this confirms:**
- For **9B-25**, larger CUDA-graph capture sizes give a real but small win — peak **+9.2% at B=2**, fading at larger batches and reversing slightly at B=16. Consistent with "CPU launch overhead from many small SSM-kernel dispatches matters at small batches; actual SSM-scan sequential depth matters at large batches and cannot be folded into a CUDA graph."
- For **27B-25 and 35B-A3B-25**, the same flags produce *worse* numbers (1–4% slower). At these model sizes the SSM scan dominates so heavily that the small CPU-side savings can't keep up with the small added overhead from CUDA graph capture / piecewise dispatch.
- `--enable-expert-parallel` and `--use-flashinfer-moe-fp16` are no-ops in our **single-GPU BF16** setup. The first targets multi-GPU expert sharding (we have one GPU); the second targets the FP16 MoE path (we run BF16). These flags were included for completeness against the upstream issue's recipe but they don't apply here.
- **None of the tuning closes the gap with Qwen3-VL.** 9B-25 at B=16 still gives ~42 users/s vs 8B's ~236 users/s — the 5.5× gap is the chunked-recurrent vs fully-parallel SSM kernel story above, and cannot be closed with vLLM 0.19 flags. Closing it requires either (a) vLLM shipping a fully parallel SSM prefill kernel, or (b) switching to a framework that already has one (e.g., a Mamba-aware training-style runtime).

These tuned numbers come from the 9B-25 / 27B-25 / 35B-A3B-25 runs in [vllm_bench_20260408T100152Z](vllm_bench_20260408T100152Z_NVIDIA_H100_80GB_HBM3.json), [vllm_bench_20260408T100345Z](vllm_bench_20260408T100345Z_NVIDIA_H100_80GB_HBM3.json), and [vllm_bench_20260408T100527Z](vllm_bench_20260408T100527Z_NVIDIA_H100_80GB_HBM3.json). The verbatim FA3 + FlashInfer GDN backend selection messages quoted at the top of this section came from the `VLLM_LOGGING_LEVEL=DEBUG` stdout of these runs.

### Implication for production {#implication-for-production}

**If accuracy is comparable, prefer Qwen3-VL over Qwen3.5-VL for this workload today.** Specifically:

- **8B is the sweet spot for the Qwen3-VL family** — fits on L40S BF16 with room to spare, vLLM gives 22–79 users/s, and the FACE training pipeline is built around it.
- **9B-25 is *not* a drop-in upgrade for 8B**: same VRAM, slower throughput, identical input format — the only reason to switch would be a measured accuracy improvement on the production eval set. We have not measured accuracy in this benchmark.
- The Qwen3.5-VL framework cost may improve as `fla-core`, `causal-conv1d`, and vLLM/SGLang's hybrid paths mature. Worth re-running this benchmark in a few months to see whether the gap closes.

This is a framework/library maturity issue, not an architectural defect. The Qwen3.5-VL design is reasonable; today's kernels and serving stacks just don't yet exploit it for prefill-dominated workloads like ours.

## Sanity Checks {#sanity-checks}

1. **All 120 (model × batch × seed) full-forward measurements returned `status: "ok"`** — no OOM, no exceptions. 24 unique (model, batch) configs × 5 seeds.
2. **All 40 truncated measurements returned `status: "ok"` on both HF and vLLM** — 5 cells × 5 seeds for 8B trunc=19 (HF + vLLM) and 3 cells × 5 seeds for 32B trunc=44 (HF + vLLM) = 80 truncated measurements per framework-or-metric.
3. **HF "UNEXPECTED weight keys" report on truncated load** — both HF truncated runs printed the expected "UNEXPECTED: layers.{19..35} / layers.{44..63}" weight-key report, confirming that layers above the cutoff were not allocated and their weights were silently dropped (see the loading log in the HF bench stdout).
4. **vLLM `Model loading took X GiB` log on truncated load** — 32B trunc=44 reported `Model loading took 44.7 GiB` (vs full 32B loading ~66 GiB), confirming weight-shard filtering worked and the L40S fit story is real.
5. **Many "non-monotonic throughput" warnings fired** — these are **expected** for HF padded batching on real data: within each seed, throughput is roughly flat or slightly decreasing past B=1–2 due to padding waste. They are not bugs.
6. **VLM/SAE time ratio always ≥ 150×** — SAE measurement window is well above noise across all full and truncated measurements. (The ratio is lower under truncation because VLM wall time shrinks but SAE wall time is unchanged; the smallest ratio observed is 8B trunc=19 at B=16: `863.9 / 0.411 ≈ 2100×` which is still way above the noise threshold.)
7. **Batch dimension assertions passed** — `input_ids.shape[0] == batch_size` and `hidden.shape[0] == batch_size` in every config (both full and truncated).
8. **VRAM scales with `batch × image_count_per_sample` (full)** / **VRAM scales with `layer_ratio × model_weights + batch × image_count_per_sample` (truncated)** — confirms that vision encoder activations dominate memory pressure under full forward, while under truncation the model weight footprint shrinks roughly linearly in the kept layer fraction.
9. **Same input samples used across all configs *within* a seed** — within seed K, samples `[0:batch_size]` are reused for all models, batch sizes, and both baselines (full and truncated), so the full-vs-truncated comparison is fair. Across seeds, samples differ by design.
10. **p95 ≈ p50 within each seed** — within-seed jitter is < 1% (10 timed passes for HF, 5 for vLLM/SGLang). The 5–25% CV in the aggregated table is **input variance across seeds**, not GPU jitter.

## Caveats & Known Gaps {#caveats-known-gaps}

1. **Attention backend vs production — partially resolved by H100 SDPA dispatching to cuDNN flash attention.** Used `sdpa`. Production code pins `flash_attention_2` (the standalone `flash-attn` package). On H100 the two paths agree within ~5% for the prefill workload measured here. Standalone `flash-attn` install attempts failed due to a `_GLIBCXX_USE_CXX11_ABI` mismatch with PyTorch 2.6's PyPI wheel; the `sdpa` cuDNN dispatch is the practical equivalent.
2. **Single-user prompt = pair prompt with Profile B dropped, instruction kept verbatim.** This stays close to the backbone's training distribution and matches the production extraction pattern in [extract_per_profile_sharded.py](../../../experiments/cbm_probe/extract_per_profile_sharded.py). If the production deployment uses a different prompt (e.g., system + Profile A only with no instruction), latency will differ slightly because text token count drops by ~50 tokens and image count is unchanged — vision still dominates so the difference is small.
3. **No production checkpoint loaded.** The base models are random-init beyond their pretrained weights. Production uses LoRA-adapted 8B from `/data/mgai/aura/profile-ranker/data/v5/checkpoint-3125/`. Forward-pass cost is invariant to weight values, so this doesn't affect timing.
4. **Cold-start excluded from JSON.** Model load times observed (only printed to stdout):
   - 4B: ~9 s
   - 8B: ~60 s (cold)
   - 32B: ~167 s (cold)
   - 9B-25 / 27B-25 / 35B-A3B-25: not separately recorded; comparable to Qwen3-VL models of similar weight size
5. **`output_hidden_states=True` overhead (HF only).** The HF bench call sets `output_hidden_states=True` to slice `outputs.hidden_states[-1][:, -1, :]`. This materializes every layer's hidden state and adds a small VRAM/latency tax. A production-optimized HF path would hook only the SAE layer via a forward hook. The truncated HF runs benefit less from this overhead reduction because they already run fewer layers. vLLM does not expose hidden states so this caveat only applies to the HF numbers.
6. **No multi-GPU.** Single-GPU only.
7. **No quantization tested.** Only BF16. 32B / 27B-25 / 35B-A3B-25 on L40S all require FP8 or NF4/INT4 in practice.
8. **Padded batching, not packed.** This benchmark uses HF processor padding. A packed-sequence approach (e.g., FlashAttention varlen, vLLM continuous batching) significantly reduces padding waste at larger batch sizes — and indeed vLLM's results show exactly that.
9. **Vision encoder dominates large batches.** At B=32 we have ~210 images flowing through the vision encoder. The vision encoder runs at full effective batch (independent of request batch), so its latency scales linearly with image count.
10. **Permission noise in stdout** — `/data/cache/huggingface` cache writes for "non-existence" markers fail with EACCES. Non-fatal, model loading succeeds.
11. **Transient deps not in `pyproject.toml`** — `torchvision==0.21.0`, `deltalake==1.5.0`. Next `uv sync` would remove them and break the benchmark.
12. **The measured rate is `users/s`, not `pairs/s`.** The earlier (now-superseded) revision of this report measured pairs and the user-level throughput was therefore overstated by ~2×. The numbers in this revision are users/s and can be used directly to size production single-user concept extraction.
13. **SGLang was not re-measured under truncation.** The three-framework side-by-side table (HF / vLLM / SGLang) is full-forward only. SGLang's IPC/CPU-side bottleneck in the offline `Engine.generate()` path dominates its wall time regardless of the underlying GPU compute, so truncation was not expected to change the SGLang numbers meaningfully and the re-run was deprioritized.
14. **Production vs benchmark SAE attachment layer** — the truncated measurements assume 8B attaches at layer 18 (0-indexed) and 32B attaches at layer 43 (0-indexed). These were confirmed by the user to match the current cbm_probe production candidates. If the final production decision attaches the SAE at a different layer, the truncated wall time will scale roughly linearly in `K/N_total` (vision encoder + embedding + small constants are fixed overhead).

## How to Re-run {#how-to-re-run}

The hyperpod worktree at `~/Workspace/git/aura/bench-sae-inference` is preserved with `.venv` already provisioned (including `torchvision` and `deltalake`).

**Full-forward runs** (Qwen3-VL-4B / 8B / 32B + Qwen3.5-VL family):

```bash
ssh login
srun --partition=h100 --gres=gpu:1 --cpus-per-task=16 --mem=200G --time=01:30:00 \
  bash -lc "
    export HF_HOME=\$HOME/.cache/huggingface PYTHONUNBUFFERED=1
    cd ~/Workspace/git/aura/bench-sae-inference/profile-ranker-face
    # Qwen3-VL family
    uv run python bench_sae_inference.py --seeds 1 2 3 4 5 \
      --model 4b 8b 32b --output-dir results/h100_full
    # Qwen3.5-VL family (requires fla-core in venv for hybrid linear-attention fast path)
    uv run python bench_sae_inference.py --seeds 1 2 3 4 5 \
      --model 9b25 27b25 35ba3b25 --output-dir results/h100_full
  "
```

**Truncated runs** (production-realistic SAE early-exit):

```bash
ssh login
# HF truncated (one model per command; cutoffs differ by model)
srun --partition=h100 --gres=gpu:1 --cpus-per-task=16 --mem=200G --time=01:30:00 \
  bash -lc "
    export HF_HOME=\$HOME/.cache/huggingface PYTHONUNBUFFERED=1
    cd ~/Workspace/git/aura/bench-sae-inference/profile-ranker-face
    uv run python bench_sae_inference.py --seeds 1 2 3 4 5 \
      --model 8b --truncate-layers 19 --output-dir results/h100_full
    uv run python bench_sae_inference.py --seeds 1 2 3 4 5 \
      --model 32b --truncate-layers 44 --output-dir results/h100_full
  "

# vLLM truncated (separate venv; materializes a temp model dir with filtered safetensors
# shards under /tmp before the LLM() call)
srun --partition=h100 --gres=gpu:1 --cpus-per-task=16 --mem=200G --time=01:30:00 \
  bash -lc "
    export HF_HOME=\$HOME/.cache/huggingface PYTHONUNBUFFERED=1
    cd ~/Workspace/git/aura/bench-sae-inference/vllm_bench
    .venv/bin/python vllm_bench_qwen3vl.py --seeds 1 2 3 4 5 \
      --model 8b --truncate-layers 19 \
      --pickle-dir ../profile-ranker-face/real_samples_pickles \
      --output-dir ../profile-ranker-face/results/h100_full
    .venv/bin/python vllm_bench_qwen3vl.py --seeds 1 2 3 4 5 \
      --model 32b --truncate-layers 44 \
      --pickle-dir ../profile-ranker-face/real_samples_pickles \
      --output-dir ../profile-ranker-face/results/h100_full
  "
```

### Variations worth running next {#variations-worth-running-next}

```bash
# Trim 32B sweep (B=1 only) if memory pressure is a concern
uv run python bench_sae_inference.py --model 32b --batch-sizes 1

# Compare flash_attention_2 vs sdpa
uv pip install flash-attn --no-build-isolation
uv run python bench_sae_inference.py --model 8b --attn-impl flash_attention_2

# Smaller seed count for a quick check
uv run python bench_sae_inference.py --model 4b --seeds 1
```

## Files {#files}

- HF bench script (orchestration): [profile-ranker-face/bench_sae_inference.py](../../bench_sae_inference.py)
- HF bench script (runtime primitives): [profile-ranker-face/bench_sae_runtime.py](../../bench_sae_runtime.py)
- vLLM bench script: [profile-ranker-face/vllm_bench_qwen3vl.py](../../vllm_bench_qwen3vl.py) (runs from `vllm_bench/` separate venv)
- SGLang bench script: [profile-ranker-face/sglang_bench_qwen3vl.py](../../sglang_bench_qwen3vl.py) (runs from `sglang_bench/` separate venv)
- Cross-venv sample dumper: [profile-ranker-face/dump_real_samples.py](../../dump_real_samples.py)
- Per-sample token length analyzer: [profile-ranker-face/analyze_sample_token_lengths.py](../../analyze_sample_token_lengths.py) → [sample_token_lengths.json](sample_token_lengths.json)
- TopKSAE definition: [profile-ranker-face/sae_layer.py](../../sae_layer.py)
- Production single-user reference: [experiments/cbm_probe/extract_per_profile_sharded.py](../../../experiments/cbm_probe/extract_per_profile_sharded.py)
- Real-data pipeline source: [profile-ranker-face/src/profile_ranker_face/data_processing/dataset.py](../../src/profile_ranker_face/data_processing/dataset.py) `PairwiseRankingDataset`
- Raw HF JSON, Qwen3-VL (5 seeds): [bench_20260408T023050Z_NVIDIA_H100_80GB_HBM3.json](bench_20260408T023050Z_NVIDIA_H100_80GB_HBM3.json)
- Raw HF JSON, Qwen3.5-VL (5 seeds): [bench_20260408T075611Z_NVIDIA_H100_80GB_HBM3.json](bench_20260408T075611Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3-VL (5 seeds): [vllm_bench_20260408T023624Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T023624Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3.5-VL 9B+27B (5 seeds): [vllm_bench_20260408T083656Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T083656Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3.5-VL 35B-A3B (5 seeds): [vllm_bench_20260408T083858Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T083858Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3.5-VL 9B *tuned* (5 seeds, `--cuda-graph-max-size 1024`): [vllm_bench_20260408T100152Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T100152Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3.5-VL 27B *tuned* (5 seeds, `--cuda-graph-max-size 512`): [vllm_bench_20260408T100345Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T100345Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, Qwen3.5-VL 35B-A3B *tuned* (5 seeds, `--cuda-graph-max-size 256 --enable-expert-parallel --use-flashinfer-moe-fp16`): [vllm_bench_20260408T100527Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_20260408T100527Z_NVIDIA_H100_80GB_HBM3.json)
- Raw SGLang JSON, Qwen3-VL (5 seeds): [sglang_bench_20260408T024807Z_NVIDIA_H100_80GB_HBM3.json](sglang_bench_20260408T024807Z_NVIDIA_H100_80GB_HBM3.json)
- Raw SGLang JSON, Qwen3.5-VL (5 seeds): [sglang_bench_20260408T082742Z_NVIDIA_H100_80GB_HBM3.json](sglang_bench_20260408T082742Z_NVIDIA_H100_80GB_HBM3.json)
- Raw HF JSON, **Qwen3-VL-8B trunc=19** (5 seeds, SAE early-exit): [bench_truncated19_20260409T143300Z_NVIDIA_H100_80GB_HBM3.json](bench_truncated19_20260409T143300Z_NVIDIA_H100_80GB_HBM3.json)
- Raw HF JSON, **Qwen3-VL-32B trunc=44** (5 seeds, SAE early-exit): [bench_truncated44_20260409T143851Z_NVIDIA_H100_80GB_HBM3.json](bench_truncated44_20260409T143851Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, **Qwen3-VL-8B trunc=19** (5 seeds, SAE early-exit): [vllm_bench_truncated19_20260409T150028Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_truncated19_20260409T150028Z_NVIDIA_H100_80GB_HBM3.json)
- Raw vLLM JSON, **Qwen3-VL-32B trunc=44** (5 seeds, SAE early-exit): [vllm_bench_truncated44_20260409T150601Z_NVIDIA_H100_80GB_HBM3.json](vllm_bench_truncated44_20260409T150601Z_NVIDIA_H100_80GB_HBM3.json)
- HTML build script: [_build_report_html.py](_build_report_html.py)
