<!-- sources:
- Jira LP project: issues assigned to juniper, brandon, parker, biggie, claude, ken; updated 2026-04-21 to 2026-04-29
- Notion PM-TL Sync (Juniper) 2026-04-24: https://www.notion.so/hpcnt/MGAI-PM-TL-Sync-Juniper-34bce00cd354809f91d8fed2c736f9e9 (AURA section)
- Slack #mgai-aura (channel C06FB511ZRR): messages 2026-04-21 to 2026-04-29
- Slack SOUP engineering channel (C0AAR3F1KR6): Brandon review request thread (4/29), SOUP effort estimation thread (4/30)
- MGAI x Tinder ML AURA Sync meeting notes (4/29): https://docs.google.com/document/d/1TKot1Mp-CFgCFs6-DppVnflrGTX-ea0O0q98cOilkD8/edit?tab=t.vtjboyaji8a7#heading=h.z4i8mnf6aiw
-->

**Changes from v4**
- Eval skip 공식 사유 추가: KB 개선에 더 많은 시간이 필요하며, 불안정한 데모로 평가할 경우 Flash Lite 성능을 부정확하게 평가할 위험이 있음
- Gemini Pro MVP 가능성 제기 + 비용 추산 진행 중 추가 (Kevin)
- Mock server 타임라인: 5/6(수) PT 오전으로 확정, ML server design doc도 같은 날 공유 목표로 병기
- Deployment & testing: 5/11 주로 구체화
- Events logging 누락 이슈 추가 (Trideep 지적, ML server doc에 포함 예정)
- ML server API에 user-id/user-number 필요 확인
- SOUP Q2 capacity: late May로 업데이트
- Trideep: optimization stream 병행 필요 언급 추가

# AURA Weekly Update — 4/29

## 1. AURA demo eval iteration — Eval 4 closing, Eval 6 prep

Knowledge Base 확장 및 프롬프팅 작업의 평가를 이번 주 iteration이 아닌 다음 iteration(Eval 6)에서 Tinder voice tone-and-manner 평가와 함께 진행하기로 했다. KB 개선에 더 많은 시간이 필요하며, 불안정한 상태로 평가를 진행할 경우 Gemini Flash Lite 자체의 성능을 부정확하게 평가하게 될 위험이 있기 때문이다.

- **Eval 4 labeling & findings** (Juniper) — Done
  - 라벨링 완료; Eval 5 반영 필요 항목 및 Flash Lite go/not-yet-go 판단 기준 정리하여 팀 공유
  - 도출된 개선 영역: repetition cluster 로직, diversity detection, blank photo 처리, UI 버그

- **Eval 4 피드백 → 데모 반영** — In Progress, 4/29 KST EOD Juniper 확인 예정
  - **Biggie (프롬프트)**: prompt v4.6.1 배포
    - repetition cluster에서 replace 대상을 non-representative photo로만 한정
    - diversity lever 조건 단순화: 얼굴 사진 + 정면 전신샷 보유 시 pass
    - blank/무의미 사진 2장 이상을 repetition cluster로 처리
    - 기타 prompt drift, 출력 구조 불일치 소규모 수정
  - **Parker (UI)**: repetition indicator 우선순위 로직 구현, replacement icon 노출 범위 수정

- **Prompt Factory — Eval 인프라 정비** (Parker) — Done
  - coaching snapshot 캐싱 버그 수정: single-view에서 run별 output 대신 최신 runtime cache를 읽던 문제 수정 — 동일 프로파일을 다른 run에서 재생성할 때 해당 run의 결과가 아닌 다른 run의 결과가 표시되는 문제였음
  - 테스트셋 추가 워크플로우 단순화 및 문서화 완료 (LP-564)
  - human eval 페이지 (`/promptfactory/aura-evals`) 에서 run별 결과 확인 및 evaluator 할당 가능

- **Eval 6 KB 확장** (Biggie) — In Progress
  - 목표: 25-concept KB를 Eval 6 prompt에 통합하여 평가
  - KB v2.0 통계 검증 완료 (Cohen's d, Bonferroni, gender×age 층화분석), Prompt Factory에 등록됨
  - 현황: KB 전체를 prompt에 주입하면 모든 프로파일에 13개 이상의 atom이 포함되어 LLM 부하가 지나치게 커질 것으로 판단. KB 활용 방식 로직 전반을 재검토 중
  - prompt v5.0.0 재구조화 병행 계획: 중복·모순 제거, output schema 재정의 포함 — 변경량이 많고 함께 결정해야 할 사항이 있어 Eval 6 전체 범위 안에서 함께 진행 예정

- **MVP 모델 검토** (Juniper) — 논의 중
  - 4/29 미팅에서 Gemini Pro 버전을 MVP에 사용하는 방향 제기
  - Kevin이 비용 추산 진행 중 (Photo Feedback Experiment Cost Estimate)
  - Ben Shin: Bedrock 경유 OpenAI API 사용 검토 제안

- **Tinder on-site** (Juniper) — In Progress
  - 출장 중 코칭 워크샵 진행; 결과 팀 공유 예정 (LP-611)
  - Eval 6 전 Photo Review voice/tone baseline 정렬 작업 진행 중 (LP-610)

## 2. Photo Review Engineering Kickoff

- **Serving options 의사 결정** (Brandon) — Done
  - Option 1 (batch): Gemini Batch API를 활용해 코칭 결과를 미리 대량 생성
  - Option 2 (on-demand): 사용자 요청 시점에 실시간으로 Gemini 호출
  - Brandon이 두 옵션의 architecture, trade-off, open question을 비교 정리한 문서를 작성하고, Trideep sync를 포함한 논의를 거쳐 **Option 2로 확정**
  - 현재 데모에 사용 중인 Gemini Flash Lite 모델 자체는 프로덕션 확정이 아님. 성능 미달 시 test region에서 scope 축소 실험 가능. eng region 100% 롤아웃은 optimization 완료 이후

- **Photo Review product/engineering decision brief** (Brandon) — Done
  - "Photo Review Option 2 Kick-off — Decision Brief" 작성 완료
  - 구성: Track A (ML-side 구현 사항) + Track B (SOUP backend 연동 스펙)
  - SOUP 엔지니어 대상으로는 Track B + Track A→B 핸드오프 리뷰 요청 (전체 문서 29p)
  - 의도: PRD 리뷰 완료로 PM-side가 언블락된 상태에서, SOUP이 5/4 투입되기 전 engineering-side 계약 내용을 확정하는 것

- **SOUP engineering review 요청** (Brandon) — Done
  - Ryan Burns, Nathan Lamb, Henry Au, Evan Keys에게 Track B 리뷰 및 미결 항목 코멘트 요청 발송 완료
  - Evan Keys 초기 검토 완료: no blockers, 전반적인 방향과 문서 명확하다는 평가
  - Henry Au 추가 검토 대기 중

- **SOUP 공수 추산** (SOUP eng) — In Progress
  - Ryan Burns가 estimate sheet 작성 시작하여 채널 공유
  - Evan Keys + Henry Au 4/30 working session 예정으로 추산 진행 중
  - SOUP Q2 2026 capacity: late May로 파악됨

- **PRD 및 product decisions** (Juniper) — Done / Part II 미정
  - 4/28 SOUP engineering 대상 team-wide PRD review 완료; engineering sync 블로커 product decision 정리 완료 (LP-613)
  - PRD Review Part II 예정: Juniper가 Track A 최종 텍스트 반영 후 진행 (bottom sheet 방향은 최단 경로로 정렬, Nancy 최종 확인 대기)

- **Mock server 배포 + ML server design doc** (Brandon, Mumu) — To Do
  - 이번 주 계약 정렬 완료 조건부로 **5/6(수) PT 오전** 목표
  - Mock server 배포와 ML server design doc 공유를 같은 날(5/6) 함께 진행 예정
  - ML server API에 user-id 또는 user-number 포함 필요 확인 (이벤트 로깅 및 user behavior feature 활용)
  - Events logging 누락 지적 (Trideep) → ML server design doc에 포함 예정
  - Deployment & integration testing: 5/11 주 예정 (e2e 테스트는 SOUP backend 준비 이후 별도 논의)

- **System design document** (Mumu) — To Do (5/6과 연계)
  - SOUP 공유용 문서(system context + contract)와 내부용 문서(ML server 컴포넌트 설계 + 실행 계획) 두 유형이 필요하며, 내부 문서는 RFC 형태로 공유 예정

- **Optimization stream** — 논의 중
  - Trideep: Photo Review에 대해 낙관적이나, MVP 이후 optimization stream을 병행으로 준비해야 함을 언급
  - 모든 기능이 LLM 생성일 필요는 없음 (예: LLM 코칭 전 단순 분류 단계 선행 가능)

- **타임라인**: 계약 정렬 ~5/2 → Mock server + design doc 5/6(수) PT 오전 → Deployment & testing 5/11 주 → SOUP Q2 capacity late May → Release QA 6/1

## 3. Demo DLP

다음 주 중 실행 예정. DLP를 염두에 두고 작성된 코드가 아니어서 삭제 전 여파를 충분히 점검해야 한다.

- **Parker — AURA demo 데이터** (In Progress)
  - 1개월 이상 미접속한 비활성 유저 데이터를 삭제 대상으로 식별; 활성 유저 약 470명 보존
  - Dry-run 완료. 이상적인 여파: eval iteration에 사용 중인 프로파일 풀이 약 160개 → 140개로 감소, 그 외 시스템 영향 없어야 함
    - 삭제 범위: DB 및 S3에 분산된 프로파일, 이미지, 분석 결과, 코칭 데이터, 유사도 데이터 등
  - 실행 승인 대기 중

- **Ken — 연구용 training dataset** (Done)
  - rSRR Predictor 학습에 사용한 dataset v5 (S3) 삭제 완료
  - 삭제 전 AURA Demo 영향 없음을 Parker, Brandon과 확인 후 진행
  - ldm-models repo cleanup (branch 정리, PR 공유) 진행 중 (LP-594)

## 4. rSRR Predictor (Ken)

MVP 범위에는 포함되지 않으나, MVP 마일스톤 이후 연구 목표를 수립하기 위해 진행 중인 연구.

- 첫 번째 모델 학습 완료: Qwen3-VL 8B + LoRA, raw rSRR regression objective. 기존 pairwise profile-ranker와 달리 프로파일 단위 expected rSRR을 직접 예측
- 평가 완료: R², MAE, Pearson, Kendall/Spearman, profile-ranker 비교, Smart Photo counterfactual alignment check
- Trideep 공유용 외부 문서 작성 중 (Ken 주도) — 이전 미팅에서 Trideep이 결과가 궁금하다고 요청한 것에 대한 응답
- ldm-models repo 정리 병행 중
