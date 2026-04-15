<!-- Sources:
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- work/photo-review-serving-options/2026-04-14-document-structure-v3.md
- Slack thread: mgai-aura, Brandon's benchmark plan (2026-04-14)
- LP-531
-->

## Changes from v1
- Fixed timeline reference (2 days ago, not last week)
- Rewrote all three items with concrete metrics, methods, and rationale based on Brandon's benchmark plan
- Expanded Qwen3-VL cost savings explanation: why adding Qwen reduces Gemini cost

# Slack message to Trideep — Serving Option Decision Readiness

Hi Trideep,

Following up from our sync on 4/13 — we identified the three engineering uncertainties that need to be resolved before we can confidently compare the serving options, and Brandon is leading the effort. Here's a summary of what we're working on:

[!NOTE] 내용이 상세한 것은 좋으나, 각 항목마다 줄글로 죽 이어져 있어서 슬랙 메시지로 전달하면 읽기 어려워보여. 구조해줄 수 있을까?

[!NOTE] 100K가 비용이 아니라 요청 수인지 이해하기 어려워.
1. **Gemini Batch API behavior** — We need to confirm that batch processing can reliably complete within a predictable window (our current estimate ranges from 1 to 6 hours for English regions, based on a prior 10K-scale backfill). Rather than running a costly 100K dry run, we plan to: (a) measure per-request latency by sending actual batch API calls, (b) reference our existing 10K backfill data to extrapolate scaling behavior, and (c) reach out to the Gemini team directly to confirm edge cases — max throughput limits, error handling under load, and whether the throttling issues we saw with NanoBanana apply to gemini batch API. This gives us the turnaround time and throughput confidence needed for Option 1.

2. **Qwen3-VL L40S performance** — Our current throughput estimate of 79 profiles/sec on L40S has not been validated on actual hardware with the full pipeline (S3 image download + CPU preprocessing + GPU inference). We will benchmark end-to-end throughput and per-request latency on L40S (MG Core CPTS infrastructure). This matters for all options: in batch mode it determines how long GPU nodes need to run (and therefore cost), and in real-time mode it determines whether we can meet the latency target.

3. **Cost reduction from Qwen3-VL + Gemini pipeline** — Today, our pure-Gemini approach handles everything in a single call: image analysis, coaching quality judgment, and natural language generation. By adding Qwen3-VL as a front-end layer, Gemini would only need to convert structured findings into coaching text (no concrete implentation plan yet). This opens up several optimizations — simpler prompts, smaller image payloads, removal of the thinking budget — all of which reduce token usage and cost. We plan to run a PoC comparing pure Gemini Pro against Qwen3-VL + Gemini Flash Light, measuring latency, output quality, and token consumption to quantify the actual savings.

Brandon is working on the detailed investigation / benchmark plans and we expect to have results within the next couple of weeks.

[!NOTE] 의도는 이해했는데 문장이 길고 이해하기 버거워.
My question: once we have confidence on these three items, is there anything else on the engineering side that would block us from making the Option 1/2/3 decision? We're asking because our downstream work — backend contract refinement (e.g., do we even need a real-time API?), dummy server testing, and integration planning — is blocked until this serving option is decided. We'd like to move to the decision as soon as the data is ready. If we need to meet other requirements or our direction is slight different with yours, please let me know.

If there are additional inputs you or Jaini would need beyond these three, it would be great to know now so we can prepare them in parallel.
