# LLM Cost Estimation Cross-Check (LP-485)

> **Sources**: Notion (AURA x GenAI sync 4/7), Slack #photo-review threads (3/26, 4/3, 4/7), Slack #mgai-aura threads (4/6, 4/8), Gemini API pricing page (ai.google.dev/pricing)
>
> **Not accessible**: Kevin's Mode report, Google Doc (Profile Feedback - MGAI x Tinder)

---

## 1. 기존 추산치 요약

### 1A. Traffic (호출량)

| 항목 | 수치 | 출처 |
|------|------|------|
| Global DAU | ~18M | Kevin (Slack 4/3) |
| Day-1 calls (global rollout) | ~180k | Notion 4/7, Juniper (Slack 4/7) |
| Steady-state daily calls | ~12.6k/day | Notion 4/7, Juniper (Slack 4/7) |
| Monthly calls (steady-state) | ~360k/month | Juniper (Slack 4/7) — **단위 모호** |

### 1B. Cost (비용)

| 항목 | 수치 | 출처 |
|------|------|------|
| Cost per call (Trideep 추산) | $0.01/call | Trideep (Slack 4/3) |
| Daily cost — every DAU scenario | $180k/day | Trideep: 18M × $0.01 |
| Monthly cost projection | ~$360k/month | Notion 4/7 — **calls인지 dollars인지 모호** |
| Allocated budget | $50k/month | Notion 4/7 |
| Chemistry team budget risk | $2-4M/year | Tyler Hebert (Slack 3/26) |

### 1C. Assumptions

- Max 1 coaching generation per user per day
- Triggered by: (1) first app open of the day, OR (2) profile photo change
- Re-run only when profile photo changes (steady-state)
- Kevin modeled traffic as "% of DAU"

---

## 2. Cross-Check: Traffic 추산

### 2A. Day-1 calls: 180k vs 18M DAU

180k calls on day 1은 **18M DAU의 1%**에 해당한다.

| 해석 | 계산 | 설명 |
|------|------|------|
| Test region (1% rollout) | 18M × 1% = 180k | 1% 리전에서만 런칭 시 |
| Eligible users only | 전체 DAU 중 coaching 대상자 = 180k | DAU 중 사진 보유/조건 충족 유저만 |
| Global, 모든 DAU | 18M calls | "Max 1 gen per user per day" 가정 시 |

> [!NOTE]
> **180k가 어느 시나리오인지 불명확.** 만약 global rollout이면 18M calls/day여야 하고, 180k라면 test region 또는 eligible 필터가 있는 것. Kevin의 Mode report에서 "% of DAU" 모델링 확인 필요.

### 2B. Steady-state: 12.6k/day

12.6k daily profile changes out of 18M DAU = **0.07% daily change rate**.

참고 벤치마크: 일반적인 소셜앱 프로필 사진 변경 비율은 0.1-0.5%/day 수준. 0.07%는 약간 낮지만 "photo changes only" 조건에서는 합리적 범위 내.

12.6k/day × 30 = **378k calls/month ≈ 360k** (수치 일치 확인)

### 2C. Scenario 정리

| Scenario | Daily Calls | Monthly Calls |
|----------|-------------|---------------|
| A. Global, every DAU | 18,000,000 | 540,000,000 |
| B. Global, day-1 only | 180,000 (day-1) + 12,600 (after) | ~545,400 |
| C. Global, steady-state only | 12,600 | ~378,000 |
| D. Test region (1%), every DAU | 180,000 | 5,400,000 |
| E. Test region (1%), steady-state | 126 | ~3,780 |

---

## 3. Cross-Check: Cost per Call

### 3A. Gemini API 가격 (2026-04 기준, per 1M tokens)

| Model | Input | Output | Batch Input | Batch Output |
|-------|-------|--------|-------------|--------------|
| Gemini 2.5 Pro | $1.25 | $10.00 | $0.625 | $5.00 |
| Gemini 2.5 Flash | $0.30 | $2.50 | $0.15 | $1.25 |
| Gemini 2.5 Flash-Lite | $0.10 | $0.40 | $0.05 | $0.20 |
| Gemini 3.1 Pro Preview | $2.00 | $12.00 | $1.00 | $6.00 |
| Gemini 3 Flash Preview | $0.50 | $3.00 | $0.25 | $1.50 |

### 3B. 예상 Token 사용량 (Photo Review Call)

AURA는 유저 프로필 사진(multi-image)을 분석하여 coaching feedback을 생성하므로:

| Component | Token 추정 | 비고 |
|-----------|-----------|------|
| System prompt + instructions | ~500 | 고정 |
| User profile text | ~300 | bio, interests 등 |
| Photos (vision tokens) | ~6,000-12,000 | 6장 기준, 이미지당 1,000-2,000 tokens |
| Output (coaching text) | ~500-1,000 | 피드백 텍스트 |
| **Total Input** | **~7,000-13,000** | |
| **Total Output** | **~500-1,000** | |

### 3C. Cost per Call 재계산

| Model | Input Cost (10k tokens) | Output Cost (750 tokens) | **Total/Call** |
|-------|------------------------|--------------------------|----------------|
| Gemini 2.5 Pro | $0.0125 | $0.0075 | **$0.020** |
| Gemini 2.5 Flash | $0.003 | $0.00188 | **$0.005** |
| Gemini 2.5 Flash-Lite | $0.001 | $0.0003 | **$0.0013** |
| 2.5 Pro (Batch) | $0.00625 | $0.00375 | **$0.010** |
| 2.5 Flash (Batch) | $0.0015 | $0.00094 | **$0.0024** |

> [!NOTE]
> **Trideep의 $0.01/call 추산은 Gemini 2.5 Pro Batch 가격과 일치.** 그러나 실시간(non-batch) 서빙 시 Pro는 $0.02/call로 2배. 현재 어느 모델/모드를 기준으로 한 건지 확인 필요.

---

## 4. Cross-Check: Monthly Cost 재계산

### 4A. "$360k/month" 해석 분기

**해석 1: 360k = 호출 수 (calls/month)**

| Model | 360k calls × unit cost | Monthly Cost |
|-------|----------------------|--------------|
| Gemini 2.5 Pro | × $0.020 | **$7,200** |
| Gemini 2.5 Flash | × $0.005 | **$1,800** |
| Gemini 2.5 Flash-Lite | × $0.0013 | **$468** |
| 2.5 Pro (Batch) | × $0.010 | **$3,600** |

→ 모든 모델에서 $50k 예산 내. **Budget shortfall이 없음.**

**해석 2: 360k = 달러 ($360,000/month)**

이 경우 역산하면:

| Model | $360k ÷ unit cost = Required calls/month | Daily calls |
|-------|----------------------------------------|-------------|
| Gemini 2.5 Pro | 18,000,000 | 600,000 |
| Gemini 2.5 Flash | 72,000,000 | 2,400,000 |
| 2.5 Pro (Batch) | 36,000,000 | 1,200,000 |

→ **이것은 Scenario A (every DAU) 또는 D (test region, every DAU)와 규모가 맞음.**

> [!NOTE]
> **핵심 불일치**: "$360k/month"가 calls이면 비용은 $1.8k-$7.2k로 예산 내. dollars이면 매일 60만-240만 호출이 필요하며, "12.6k/day steady-state"와 양립 불가. Juniper 본인도 "calls인지 $$$인지 모르겠다"고 언급.

### 4B. Trideep의 "$180k/day" 추산 검증

- Trideep: 18M × $0.01 = $180,000/day
- Kevin이 정정: "actually no - it only triggers an execution" (incomplete)
- 재계산: 18M calls/day × $0.02 (Pro, non-batch) = **$360,000/day = $10.8M/month**
- 재계산: 18M calls/day × $0.01 (Pro, batch) = **$180,000/day = $5.4M/month**

> [!NOTE]
> **Kevin의 정정 내용이 불완전.** "It only triggers an execution"이 의미하는 바가 (a) 모든 DAU가 아닌 일부만 trigger된다, (b) call 비용 산정이 다르다, (c) 다른 모델이다 — 중 어느 것인지 확인 필요.

### 4C. 시나리오별 월 비용 요약

| Scenario | Monthly Calls | Pro ($0.02) | Flash ($0.005) | Flash-Lite ($0.0013) | Pro Batch ($0.01) |
|----------|--------------|-------------|----------------|---------------------|-------------------|
| A. Global, every DAU | 540M | $10.8M | $2.7M | $702k | $5.4M |
| C. Global, steady-state | 378k | $7,560 | $1,890 | $491 | $3,780 |
| D. Test 1%, every DAU | 5.4M | $108k | $27k | $7k | $54k |
| E. Test 1%, steady-state | 3,780 | $76 | $19 | $5 | $38 |

---

## 5. 불일치 요약 및 확인 필요 항목

### Critical

| # | 항목 | 현황 | 확인 대상 |
|---|------|------|----------|
| 1 | **"~360k/month"의 단위** | Juniper 본인도 calls vs dollars 불확실 | Trideep |
| 2 | **180k day-1 calls의 scope** | Global인지 test region인지 불명 | Kevin's Mode report |
| 3 | **Kevin의 정정 내용** | "actually no - it only triggers an execution" — 불완전 | Kevin |
| 4 | **사용 모델 미확정** | Pro vs Flash vs Flash-Lite에 따라 비용 15x 차이 | Engineering team |

### Important

| # | 항목 | 분석 결과 |
|---|------|----------|
| 5 | $50k budget vs 실제 비용 | Steady-state(378k calls/month)면 모든 모델에서 예산 내. Every-DAU면 Flash-Lite Batch만 가능 |
| 6 | Batch vs Real-time | Batch 시 비용 50% 절감. Notion에서 batching 방향 합의 언급 |
| 7 | 경량 모델 전환 효과 | Pro → Flash-Lite 전환 시 15x 절감 (Owen Lee의 "1/10 절감 가능" 추정과 일치) |

### 결론

현재 추산치의 가장 큰 문제는 **시나리오 혼재**다:
- "Steady-state 12.6k/day"와 "every DAU 18M"이 동일 맥락에서 논의되면서 비용 추산이 $3.6k-$10.8M 범위로 100배 이상 차이남
- "$360k"가 calls인지 dollars인지에 따라 budget gap 존재 여부가 완전히 달라짐
- 1순위로 확인해야 할 것: **실제 서빙 시나리오 (every DAU vs profile-change-only)와 $360k의 단위**
