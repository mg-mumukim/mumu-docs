# Context / Agenda

Gen AI x MG AI meeting note: https://docs.google.com/document/d/1TKot1Mp-CFgCFs6-DppVnflrGTX-ea0O0q98cOilkD8/edit?tab=t.vtjboyaji8a7#heading=h.e8ndhftarqhf

This meeting is **to solidify what we need to prepare and realign our priorities** based on our previous discussions, Trideep's comments, and the overall progress of the meetings. I've scheduled it for an hour since we need to both prep (ideation, organizing) and make decisions on action items, assignee, and timeline.

**The goal is to prepare sufficient context and empirical data to help both the Product and Engineering teams make informed decisions.** Ultimately, we will work backward from the specific deliverables and metrics that need to be presented to Jaini, Trideep, and Soup? (including their format) to determine exactly what tasks are required to produce them.

Finally, the three of us will discuss how to divide the workload for this week at the Jira ticket level.

# Requirements

> 뭘 원하는 지 여기다가 정리를 해보자
>
- **Product**
    - 각 시나리오 별로 유저가 코칭을 받기 전까지 뒤에서 어떤것들이 돌아가는지 → 의사 결정에 필요하다
        - Brandon) Juniper에게 많이 말씀드렸고, 문서 작성하면서 빠진 부분만 채운다고 보면 될 듯; 빠진 부분 = stakeholder에게 전달해야하는데 juniper가 이해가 덜 된 부분들을 이야기함; 내용은 머릿속에 있기는 한데 문서로 잘 나와있지 않음(e.g., as-is coaching이 1m 걸림 이런 것들)
        - Mumu) user story as-is는 (12345) 이해하기 어려움, 간략화된 sequence diagram 같은 게 나을듯 timeline이 여러 개고 분기가 있을 수도 있어서
    - 누락될수 있을만한 케이스의 유저는 뭐가 있는지 (e.g., Option 1 에서는 배칭 돌아가는동안 새로 가입한 사람은 코칭 못받을수 있음)
        - 이에 대한 fallback scenario 는 뭔지?
        - Mumu) kevin에게 시나리오를 주고 숫자를 달라고 할 수 있지 않을까?
        - Brandon) 시간이 중요하면 우리가 갖고 있는 숫자로도 할 수 있음
    - 추가적으로 결정해야 되는 부분은 뭔지 (어디 정보가 비어있는지)
        - Brandon) 최초 배포 대상, 인앱 넛지 디자인 - 목업 만들어주면 거기서 결정하기 편한가?
        - Juniper) 그냥 정해야한다고 주기에는 감을 못잡는 거 같아서 우리가 판을 깔고 빈칸을 채워달라고 하자. Okay, 상호 얼마까지 알아보고 오셨나요 상태라서 다 깔아주자.
- **Engineering**
    - [A] 79 profiles/sec seems too high throughput based on previous experience = need re-estimation → L40S에서 직접 돌려본 적이 없고, S3에서 다운로드받아 CPU 로직 도는 것까지 다 잰 게 아니기 때문에 추산이 확실하지 않은 것은 맞음
    - [B] We need to confirm upside of adding Qwen to our pipeline → Eval 4의 주제, 따라서 이것 때문에 지금 업무 내용이나 타임라인이 달라질 것은 없음
    - [C] How much tokens will be saved after adding Qwen to our pipeline → 커스텀 모델을 추가하면 Gemini가 할 일이 줄어들어서 빨라질 수 있는데 구체적으로 얼마이냐는 모르는 게 맞음, 속도 측정과 성능 측정이 모두 필요함(성능 비교는 Eval 4에서 끼워넣을 수 있으려나?)
        - 어느정도 granularity 로 계산해서 줘야 하는걸까? →
    - [D] Gemini batching — 24 hour scenario confirmation
        - Juniper) 내보내기로 한 시점에는 한 번에 배칭을 해두고 배포하면 되는 건데?? 뭐가 문젠건지 정확한 이해가 되지 않음
        - Brandon) 싱크가 안 맞는 부분 1) Loss 가 얼만큼 어떤게 일어나는지, 2) 코칭 생성 시간에 대한 싱크 (5초/ 30초/ 1분 등등 얼마나 걸리냐에 따라 flow 가 달라짐)

Mumu) 의사 결정이 오래 걸리는 이유는 기술적 구현 가능성의 불확실성이 해소되지 않은 상태로 안을 들고가면 의사 결정을 할 수 없기 때문이다. 그 외의 요소는 이야기가 나온 바가 없다.

Juniper) product+한 문서로 정리하며 일하되, 전달이나 소통이나 사람 별 범위는 나누어 가져가자.

Brandon) 이것저것 조사/집계한 문서

[2026-04-13-photo-review-llm-serving-options](https://www.notion.so/2026-04-13-photo-review-llm-serving-options-342ce00cd35480baa465e25fecbc7950?pvs=21)

# Action Items

1. before app open
2. on app open, ≥ 30s
3. on app open, ≤ 5s
- [ ]  @Mumu ​Kim, @Brandon, @Juniper.H (Songju) Han jira ticket 만들기
- **Product 의사 결정에 필요한 문서 작성하기**
    - [ ]  @Mumu ​Kim 지금 문서 구조 검토하고 필요한 section 미리 만들어두고 내부/외부 컨펌받기, 외부는 아래 액션 아이템과 연결됨
    - **옵션에 따른 각 시나리오의 user flow 설명**
        - [ ]  @Juniper.H (Songju) Han option 123 별 user scenario detail 채우기(내용이 빠진 것도 있고 설명이 이해하기 어려운 것도 있고)
        - [ ]  @Brandon option 123 별 user scenario + processing 다이어그램 준비
            - (mumu: 이런 느낌)

                ![image.png](attachment:2b8488e0-fd14-4221-b720-5e5985bb1c10:image.png)

        - [ ]  @Juniper.H (Songju) Han option 123의 내용 이해도 물론 쉬워야겠지만, 핵심은 읽고 그들이 결정을 할 수 있도록 pros/cons 정리
            - (mumu: 이런 느낌)


                |  | Option 1 | Option 2 | Option 3 |
                | --- | --- | --- | --- |
                |  Pros |   • Gain 1
                  • Gain 2
                  • Gain 3 |   • Gain 1
                  • Gain 2
                  • No Risk 1 |   • Gain 2
                  • Gain 3
                  • New Gain 4
                  • No Risk 2 |
                | Cons |   • Risk 1
                  • Risk 2 |   • No Gain 3
                  • Risk 2 |   • No Gain 1
                  • Risk 1
                  • New Risk 3 |
                | Confidence
                in 6 weeks | High | Medium | Low |
                | Suggestion | 이러저러하면 이거 고르면 된다, 우리의 의사 결정 기준을 언급 | 이러저러하면 이거 고르면 된다, 우리의 의사 결정 기준을 언급 | 이러저러하면 이거 고르면 된다, 우리의 의사 결정 기준을 언급 |
    - **시나리오 별 누락되는 케이스 조사, 분석**
        - [ ]  @Juniper.H (Songju) Han 누락될 수 있는 케이스와 이에 따른 유저 경험 예측
        - [ ]  @Brandon 누락되는 케이스의 number 조사하여 채우기, 필요시 기술적 fallback plan 추가
    - **그 외 의사 결정 포인트 정리**
        - [ ]  @Juniper.H (Songju) Han 최초 배포 대상, 인앱 넛지 등 항목 별로 내용이 뭐고 누가 의사 결정을 해주면 되고(e.g., pm/design/ml/backend) 우리 업무가 어떻게 영향을 받는지 정리해서 — 빈칸만 채우면 되게 전달
        - [ ]  @Juniper.H (Songju) Han, @Brandon 그 외 의사 결정 포인트가 있는지 파악해서 필요시 추가
- **Engineering 의사 결정에 필요한 자료 준비하기**
    - [ ]  @Mumu ​Kim 이 “의사 결정” 논의에서 무엇을 준비할 것이고 얼마나 시간이 걸릴 것인지 Tinder로 전달하는 거 초안 작성해서 리뷰 받고 내보내기 - 목적 + 답변 중요, 이번에 끝내고 싶다는 내용
    - **@Brandon Gemini batch API benchmark (throughput, turn-around time)**

        rational) google 문서에 그냥 쓰여있는 내용 못믿는다 + 실제로 돌려보았을 때 special error handling이 필요하다던지 oom이 난다던지 이런 거 모른다

        - [ ]  gemini benchmark 계획 - 범위/지표/일정 - @Mumu ​Kim advisor
        - [ ]  gemini benchmark 환경 구축, 실행, 집계
        - [ ]  gemini benchmark 결과 해석하여 문서에 결론 정리 → option 1 evidence
        - [ ]  문서에 cost estimation number 정리
    - **@Brandon L40S Qwen + s3 image download benchmark (throughput, latency)**

        rational) 실제로 돌려본 적이 없어서 필요함 위에 이야기했듯 실측치가 중요함

        - [ ]  l40s + s3 benchmark 계획 - 범위/지표/일정 - @Mumu ​Kim advisor

            memo) real-time approach에서는 latency가 유효하다. batch approach에서는 throughput이 유효하다. 따라서 실행 시나리오를 잘 짜서 다 재야한다.

        - [ ]  l40s + s3 benchmark 환경 구축, 실행, 집계
        - [ ]  l40s + s3 benchmark 결과 해석하여 문서에 결론 정리 → all options or independently
        - [ ]  문서에 cost estimation number 정리
    - **@Brandon [gemini-pro, …, qwen + gemini-flash-light] PoC && benchmark (latency, score)**
        - [ ]  몇 가지 옵션을 poc해볼 거고, 각 옵션 별로 점수를 누가 어떻게 매길 것인지 전략부터 세워야함  - @Mumu ​Kim advisor
        - [ ]  문서에 cost estimation number 정리
    - [ ]  @Mumu ​Kim F/U QoS of Gemini API
    - [x]  Cost dashboard update w/ real numbers — 이거 해야하는 게 맞나?

        결론: 이거는 지금 버전은 당분간 deprecated 시키고 필요하면 되살리던지 하자
