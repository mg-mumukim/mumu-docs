<!-- sources:
- Jira LP project: issues assigned to juniper, brandon, parker, biggie, claude, ken; updated 2026-04-21 to 2026-04-29
- Notion PM-TL Sync (Juniper) 2026-04-24: https://www.notion.so/hpcnt/MGAI-PM-TL-Sync-Juniper-34bce00cd354809f91d8fed2c736f9e9 (AURA section)
- Slack #mgai-aura (channel C06FB511ZRR): messages 2026-04-21 to 2026-04-29
-->

**Changes from v1**
- 한국어로 전환
- Section 1: 제목 수정, 인트로 재작성 (Eval 5 skip 표현 → KB/프롬프팅 평가를 다음 iteration으로 이동한 이유 중심), 표 → nested bullet, Parker/Biggie 역할 분리, 4/29 EOD 확인 예정 추가, Prompt Factory 인프라 항목 추가, Eval 6 KB 현황 추가, 마지막 항목을 "Tinder on-site"로 변경
- Section 2: 제목 수정, Serving options 의사 결정 항목 추가, decision brief 의도 명시, Owen 제거, PRD/product decisions 항목 이동(Section 1에서), system design document 항목 추가
- Section 3: Parker(demo)/Ken(dataset) MECE 구조로 재편, 여파 설명 강화
- Section 4: 제목에서 Biggie 제거, MVP 범위 아님 설명 추가, Trideep 공유 이유 추가, Eval 6 baseline 언급 제거

# AURA Weekly Update — 4/29

## 1. AURA demo eval iteration — Eval 4 closing, Eval 6 prep

Knowledge Base 확장 및 프롬프팅 작업의 평가를 이번 주 iteration이 아닌 다음 iteration(Eval 6)에서 Tinder voice tone-and-manner 평가와 함께 진행하기로 했다. 왜냐하면 Biggie가 4/28에 확인한 결과 25-concept KB를 데모에 통합하려면 프롬프트 전반 재구조화가 함께 필요한데, 상호 의존적인 변경이 너무 많아 이번 iteration 안에서 별도로 분리하기 어렵기 때문이다.

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
  - SOUP 엔지니어 대상으로는 Track B 리뷰만 요청 (전체 문서 29p)
  - 의도: PRD 리뷰 완료로 PM-side가 언블락된 상태에서, SOUP이 5/4 투입되기 전 engineering-side 계약 내용을 확정하는 것

- **SOUP engineering review 요청** (Brandon) — In Progress
  - SOUP 엔지니어(Ryan Burns, Nathan Lamb)에게 Track B 리뷰 및 미결 항목 코멘트 요청 발송 예정
  - Juniper가 spec kick-off 미팅에서 follow-up 예정

- **PRD 및 product decisions** (Juniper) — Done this week
  - 4/28 SOUP engineering 대상 team-wide PRD review 완료
  - engineering sync를 막고 있던 product decision 항목 정리 완료 (LP-613)

- **Mock server 배포** (Brandon) — To Do
  - SOUP backend 구현 시작 전 필요. 목표: US 월요일 5/4 전

- **System design document** (Mumu) — To Do
  - mock server 이후, SOUP backend 구현 시작 시점에 맞춰 작성 예정
  - SOUP 공유용 문서(system context + contract)와 내부용 문서(ML server 컴포넌트 설계 + 실행 계획) 두 유형이 필요하며, 내부 문서는 RFC 형태로 공유 예정

- **타임라인**: SOUP 투입 5/4 → Release QA 6/1

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
