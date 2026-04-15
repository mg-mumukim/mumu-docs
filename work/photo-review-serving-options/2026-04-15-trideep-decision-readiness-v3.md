<!-- Sources:
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- work/photo-review-serving-options/2026-04-14-document-structure-v3.md
- Slack thread: mgai-aura, Brandon's benchmark plan (2026-04-14)
- LP-531
-->

## Changes from v2
- Restructured each item with sub-bullets for readability as a Slack message
- Clarified "100K" as "100K requests" with explicit unit
- Simplified the closing question into shorter sentences

# Slack message to Trideep — Serving Option Decision Readiness

Hi Trideep,

Following up from our sync on 4/13 — we identified the three engineering uncertainties that need to be resolved before we can confidently compare the serving options, and Brandon is leading the effort. Here's where we are:

[!NOTE] goal: 10K-request backfill은 recs pillar의 chat tag extraction pipeline을 구성할 때 Joel.Ham's work라는 것을 명시해줘
[!NOTE] approach-a: 정합성은 맞지 않아도 되고, input/output token 개수 정도만 맞추는 정도로 가짜 요청을 채울 거라고 해줘.
[!NOTE] approach-c: #tinder_gcp 채널이라고 언급해줘.
**1. Gemini Batch API behavior**
- Goal: Confirm that batch processing completes within a predictable window. Our current estimate is 1-6 hours for English regions, based on a prior 10K-request backfill.
- Approach: Rather than running a full 100K-request dry run, we plan to:
    - (a) Measure per-request latency with actual batch API calls
    - (b) Extrapolate from our existing 10K-request backfill data
    - (c) Reach out to the Gemini team to confirm max throughput limits, error handling, and whether the throttling issues we saw with NanoBanana apply to batch API
- This gives us the turnaround time confidence needed for Option 1.

**2. Qwen3-VL L40S performance**
- Goal: Validate our throughput estimate (currently 79 profiles/sec) on actual L40S hardware with the full pipeline — S3 image download, CPU preprocessing, and GPU inference.
- Approach: End-to-end benchmark on L40S (MG Core CPTS infrastructure), measuring both throughput and per-request latency.
- This matters for all options: in batch mode it determines GPU node runtime (and cost); in real-time mode it determines whether we can get the reasonable latency and throughput.

[!NOTE] approach에서, 유의미한 성능 절감인지 알기 위해서는 성능 저하 폭이 얼마인지도 알아야하기 때문에, 그리고 성능 저하를 줄일 가능성이 있는지도 알고 싶을 것이기 때문에, 1번 Gemini Batch API처럼 dummy 요청을 보내는 것이 아니라 실제로 동작하는 프롬프트 엔지니어링 및 human-eval 또는 llm-as-a-judge scoring이 필요하다고 생각하고 있다는 내용을 적어줘.
[!NOTE] concern: 이렇게 하면 하루 이틀만에 확인해보기는 어려울 것 같다는 내용을 적어줘. 이번 주는 기존 gemini-based solution과 신규 model+gemini solution을 비교하는 Eval 4를 위한 AURA demo 개발에 MLE와 MLSE의 리소스가 모두 쓰이고 있어서 다음 주에 기획하여 진행해야 할 수 있을 것 같다고. 이런 타임라인으로 봐야하더라도 보는 게 좋다고 생각하는지 의견이 궁금하다고도 적자.
**3. Cost reduction from Qwen3-VL + Gemini pipeline**
- Context: Today our pure-Gemini approach handles everything in a single call — image analysis, coaching quality judgment, and natural language generation. Adding Qwen3-VL + BoostingTree as a front-end layer means Gemini only needs to convert structured findings into coaching text if everything goes well.
- Why it matters: This enables simpler prompts, smaller image payloads, and removal of the thinking budget — all reducing token usage and cost.
- Approach: PoC comparing pure Gemini Pro vs. Qwen3-VL + Gemini Flash Light, measuring latency, output quality, and token consumption.

[!NOTE] next couple of weeks 같은 너의 추측 적지 마. evidence 1, 2는 다소 부가적인 gcp 응답을 제외하면 이번 주 중으로 생각하고 있고 evidence 3는 상기하였듯 다음 주로 생각하고 있어.
Brandon is working on the detailed investigation and benchmark plans. We expect results within the next couple of weeks.

**Question for you:** Once we have confidence on these three items, is there anything else on the engineering side that would block the Option 1/2/3 decision?

[!NOTE] blocked 보다는 dependent하다는 어조로 전환하여 적어줘.
Our downstream work — backend contract refinement, dummy server testing, integration planning — is blocked until this decision is made. If our direction is different from what you expect, or if you or Ben need additional inputs, we'd like to know now so we can prepare it instead of them.
