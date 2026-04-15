<!-- Sources:
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- work/photo-review-serving-options/2026-04-14-document-structure-v3.md
- LP-531
-->

# Slack message to Trideep — Serving Option Decision Readiness

Hi Trideep,

[!NOTE] last week가 아니라 4/13 (2 days ago)이야.
Following up from our sync last week — we're working on resolving the three engineering uncertainties you raised:

[!NOTE] https://hpcnt.slack.com/archives/C06FB511ZRR/p1776171430236209?thread_ts=1776162341.278579&cid=C06FB511ZRR Brandon의 계획 내용도 참고해서, Trideep이 걱정하는 부분에 대한 내용으로 다시 채워 내 추측이 아니라. 확인하고자 하는 것이 구체적으로 어떤 지표이고, 그 방식이 무엇인지에 대한 간략한 설명을 덧붙이고.
1. **Gemini batch API behavior** — actual throughput and turnaround time (not just Google's published numbers; given the NanoBanana QPS issue we want real measurements)
[!NOTE] 이미 했다는 게 아니라 실측치를 잴 것이다 라는 내용이 맞는지 봐. 1/2/3 다른 항목처럼 충분히 설명력있게 하려면 분량도 늘어나야 할 수 있어. 기존 자료를 참고해.
2. **Qwen3-VL performance** — measured throughput and latency including image download (the 79 profiles/sec estimate hasn't been validated on real hardware)
[!NOTE] 이게 왜 cost saving인지, 그리고 이 일이 어떤 규모인지 잘 드러나지 않아. 내용을 비구조화된 상태로 설명해보자면, 실제로 좋은 코칭을 판단해야하는 기존 pure prompting에 비하여 Qwen3-VL이 앞단에 추가되면 gemini가 자연어로 풀어내기만 하면 되므로 prompt를 간소화하거나, 이미지 크기를 줄이거나, thinking budget을 제거하는 등의 최적화가 가능할 것으로 생각되는데 실제로 이를 PoC 수준에서 수행해서 비용 절감 정도의 가닥을 잡는 업무야. 이게 드러나도록 해.
3. **Cost savings effect of Qwen3-VL** — Gemini Pro vs. Qwen + Gemini Flash Light comparison on latency, quality, and token usage

Brandon is leading the benchmarking effort and we expect to have results within the next couple of weeks.

My question: once we have confidence on these three items, is there anything else on the engineering side that would block us from making the Option 1/2/3 decision? We're asking because our downstream work — backend contract refinement (e.g., do we even need API?), dummy server testing, and integration planning — is blocked until this serving option is decided. We'd like to move to the decision as soon as the measurement is ready.

If there are additional inputs you or Jaini would need beyond these three, it'd be great to know now so we can prepare them in parallel.
