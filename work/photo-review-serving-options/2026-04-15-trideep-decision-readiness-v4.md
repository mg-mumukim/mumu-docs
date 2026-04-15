<!-- Sources:
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- work/photo-review-serving-options/2026-04-14-document-structure-v3.md
- Slack thread: mgai-aura, Brandon's benchmark plan (2026-04-14)
- LP-531
-->

## Changes from v3
- Item 1: credited 10K backfill to Joel Ham's chat tag extraction pipeline work for Recs pillar
- Item 1 approach-a: clarified dummy requests match input/output token counts only, not semantic correctness
- Item 1 approach-c: specified #tinder_gcp channel
- Item 3 approach: added need for working prompt engineering + human-eval or LLM-as-a-judge scoring (not dummy requests)
- Item 3: added timeline concern — Eval 4 development consuming MLE/MLSE resources this week, likely next week for this item; asked Trideep's opinion on this timeline
- Replaced "next couple of weeks" with specific per-item timeline
- Changed "blocked" tone to "dependent" tone in closing
- Rearranged paragraphs

# Slack message to Trideep — Serving Option Decision Readiness

Hi Trideep and Ben,

Following up from our sync on 4/13 — we identified the three engineering uncertainties that need to be resolved before we can confidently compare the serving options, and Brandon is leading the effort. Engineering details might be changed while investigation or benchmark.

**Question for you:** Once we have confidence on these three items, is there anything else on the engineering side that would hinder the Option 1/2/3 decision?

Our downstream work — backend contract refinement, dummy server testing, integration planning — depends on this decision. If our direction is different from what you expect, or if you or Ben need additional inputs, we'd like to know now so we can prepare it instead of them.

**1. Gemini Batch API behavior**
- Goal: Confirm that batch processing completes within a predictable window. Our current estimate is 1-6 hours for English regions, based on a prior 10K-request backfill (Joel Ham's chat tag extraction pipeline work for Recs pillar).
- Approach: Rather than running a full 100K-request dry run, we plan to:
    - (a) Send batch API calls with dummy requests that match real input/output token counts (semantic correctness is not required — we only need to measure throughput and latency at representative payload sizes)
    - (b) Extrapolate from Joel's existing 10K-request backfill data
    - (c) Reach out to the Gemini team via #tinder_gcp to confirm max throughput limits, error handling, and whether the throttling issues we saw with NanoBanana apply to batch API
- This gives us the turnaround time confidence needed for Option 1.
- Timeline: This week, excluding the GCP team response which may take longer.

**2. Qwen3-VL L40S performance**
- Goal: Validate our throughput estimate (currently 79 profiles/sec) on actual L40S hardware with the full pipeline — S3 image download, CPU preprocessing, and GPU inference.
- Approach: End-to-end benchmark on L40S (MG Core CPTS infrastructure), measuring both throughput and per-request latency.
- This matters for all options: in batch mode it determines GPU node runtime (and cost); in real-time mode it determines whether we can get reasonable latency and throughput.
- Timeline: This week.

**3. Cost reduction from Qwen3-VL + Gemini pipeline**
- Context: Today our pure-Gemini approach handles everything in a single call — image analysis, coaching quality judgment, and natural language generation. Adding Qwen3-VL + BoostingTree as a front-end layer means Gemini only needs to convert structured findings into coaching text if everything goes well.
- Why it matters: This enables simpler prompts, smaller image payloads, and removal of the thinking budget — all reducing token usage and cost.
- Approach: Unlike items 1 and 2, this cannot be measured with dummy requests. We need working prompt engineering and either human evaluation or LLM-as-a-judge scoring to assess whether the cost savings come with acceptable quality degradation — and whether there is room to close the gap.
- Timeline concern: This week, our MLE and MLSE resources are fully committed to Eval 4 development (comparing the existing Gemini-based solution against the new model+Gemini solution in AURA demo). Realistically, we would plan and execute this item next week. Would you still consider it worth pursuing on this timeline, or would you prefer we prioritize differently?
