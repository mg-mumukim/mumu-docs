# Literature Survey Skill

주어진 주제에 대해 체계적인 문헌 조사를 수행하고, `literature-survey/` 폴더에 결과물을 저장한다.

이 스킬은 대규모 작업이다. 반드시 Agent 도구를 사용해 general-purpose 에이전트에게 위임해서 처리한다. 직접 처리하지 않는다.

## 사용법

```
/literature-survey <주제> [출력 폴더]
```

- `<주제>`: 조사할 주제 (필수). 예: "model cascading in ML systems", "prevalence sampling for imbalanced classification"
- `[출력 폴더]`: 결과물을 저장할 폴더 (선택). 지정하지 않으면 현재 작업 디렉토리 아래 `literature-survey/` 폴더에 저장.

예시:
- `/literature-survey model cascading in ML systems`
- `/literature-survey backward compatible embeddings projects/recommendation/model-cascading`

## 폴더 구조

```
literature-survey/
├── [주제]-requirements.md             # 조사 요구사항 및 검색 전략
├── [주제]-literature-review.md        # 전체 논문 목록 + 요약 리뷰
├── [주제]-executive-summary.md        # 의사결정자용 요약
├── [주제]-method-tracker.md           # 방법론 근본성 추적 (PDF에서 누적)
├── [주제]-queue.md                    # 논문 처리 큐
└── read-papers/                       # 개별 논문 상세 분석
    ├── 2024_NeurIPS_MethodName_Paper-Title.md
    ├── 2024_NeurIPS_MethodName_Paper-Title.pdf
    └── ...
```

## 파일 이름 규칙

```
연도_(학회or소속기관)_메소드이름_논문전체제목.md
```

- 공백 → `-`
- 특수문자 제거 (`:` `(` `)` `/` `'` `"` 등)
- 학회/저널에 게재된 논문은 학회명 사용 (예: `CVPR`, `NeurIPS`, `ICML`, `AAAI`)
- arxiv preprint (학회 미게재)는 소속 기관으로 표시:
    - 저자의 50% 이상이 같은 기관이면 그 기관명 사용
    - 해당 기관 없으면 1저자 소속 사용
    - 잘 알려진 기관은 약어 사용: MIT, Stanford, Harvard, CMU, KAIST, POSTECH 등
    - 기업은 약어 or 통용명 사용: `Google-DeepMind`, `Meta`, `OpenAI`, `Mistral`
    - 중국 대학은 `중국대학`으로 통일
- 메소드 이름 불명확하면 첫 번째 핵심 키워드 사용
- 예시:
    - 학회 논문: `2025_CVPR_ResCLIP_Residual-Attention-for-Training-free-Dense-Vision-language-Inference.md`
    - arxiv (MIT 저자): `2026_MIT_InverseDepthScaling_Inverse-Depth-Scaling-From-Most-Layers-Being-Similar.md`
    - arxiv (Google DeepMind): `2026_Google-DeepMind_AutoHarness_Improving-LLM-Agents.md`
    - arxiv (중국 대학): `2024_중국대학_MethodName_Paper-Title.md`
- PDF 파일명 = markdown 파일명에서 확장자만 `.pdf`로 변경
    - 예: `2025_CVPR_ResCLIP_Residual-Attention-for-Training-free-Dense-Vision-language-Inference.pdf`

## 소속 기관 찾는 법

소속 기관은 반드시 논문 원문(PDF)에서 확인한다. PDF 첫 페이지 저자 이름 아래에 표시된다.
- PDF를 Read 도구로 읽으면 첫 페이지에 affiliation이 나온다.
- PDF 접근 불가 시: Semantic Scholar, Google Scholar, DBLP에서 검색
- 그래도 없으면: "정보 없음"으로 표기

## 실행 순서

### Phase 1: 준비 및 검색 전략 수립

1. 출력 폴더와 `read-papers/` 서브폴더가 없으면 생성한다.
2. 주제에서 핵심 키워드를 추출한다 (3-6개).
3. 아래 기준으로 검색 전략을 결정한다.
4. 검색 전략이 결정되면 즉시 `[주제]-requirements.md` 파일을 생성한다. 이 파일에는 사용자가 요청한 주제, 추출한 키워드, 선택한 학회/블로그 목록, 검색 쿼리, 그리고 조사 범위 관련 제약 사항을 기록한다. 아래 템플릿을 따른다.
5. requirements 파일 생성 후 반드시 사용자에게 내용을 보여주고 확인을 받는다. 사용자가 수정을 요청하면 반영한 뒤 다시 확인을 받는다. 사용자가 승인한 후에야 Phase 2로 넘어간다.
    - 핵심 키워드 그룹: 주제의 다양한 표현, 관련 개념들
    - 가장 관련성 높은 학회 목록 (아래 목록에서 선택)
    - 검색할 엔지니어링 블로그 목록

관련성 높은 학회/저널:
- ML 일반: NeurIPS, ICML, ICLR, MLSys
- 시스템: OSDI, SOSP, EuroSys, VLDB, SIGMOD
- 컴퓨터 비전: CVPR, ICCV, ECCV
- NLP: ACL, EMNLP, NAACL
- 추천 시스템: RecSys, WWW, KDD
- 소프트웨어 공학: ICSE, FSE, ASE
- 신호 처리: ICASSP, Interspeech

### 논문 우선순위 기준

검색된 논문을 처리할 때 아래 우선순위를 따른다. 우선순위가 높은 논문은 더 깊이 분석(Phase 3 read-papers/)하고, 낮은 논문은 간략히만 언급한다.

#### 주제 연관성 (최우선 필터)

학회/인용 수보다 상위에 있는 기준이다. 연관성이 낮은 논문은 아무리 학회가 좋아도 순위가 내려간다.

연관성 수준 판단 기준 (제목, abstract, introduction을 보고 판단):

- 핵심 (Core): 조사 주제를 직접 다루는 논문. requirements에 명시된 키워드가 논문의 중심 contribution임. → 연관성 가중치 없음 (기본)
- 관련 (Related): 조사 주제와 인접하지만 직접 다루지는 않는 논문. 배경 이해에 필요하거나 비교 대상이 될 수 있음. → 아래 1/2/3/4 순위에서 한 단계 강등
- 주변 (Peripheral): 조사 주제와 간접적으로만 연결되는 논문. → 아래 1/2/3/4 순위에서 두 단계 강등. 강등 후 3순위 미만이면 "제외"로 처리.

연관성은 큐에 항목을 추가할 때 "[핵심/관련/주변]"으로 명시한다. 이후 배치 처리 시 실제 PDF/abstract를 보고 재평가해서 순위를 조정한다.

#### 학회/기관 기반 기본 순위

1순위 - 최우선 처리:
- NeurIPS, ICML, ICLR 게재 논문
- Google Scholar h5-index 100 이상인 학회의 논문 (NeurIPS, ICML, ICLR, CVPR, ACL, EMNLP 등이 해당)
- Google, Meta/FAIR, Microsoft Research, DeepMind, Apple, Amazon, OpenAI 등 빅테크 소속 저자의 논문

2순위 - 다음으로 처리:
- CVPR, KDD 게재 논문
- Google Scholar h5-index 100 이상이지만 1순위에 포함되지 않은 학회의 논문
- 인용 수 100 이상인 논문 (학회 무관)

3순위 - 보충 자료:
- ICCV, ECCV, WWW, RecSys, EMNLP, NAACL, SIGMOD, VLDB 등 주요 학회
- arXiv preprint이지만 저명 저자 또는 빅테크 소속 + 인용 수 50 이상

4순위 - 참고 수준 (아래 조건 중 하나라도 해당하면 4순위로 강제 배정):
- 기타 학회 또는 인용 수가 낮은 arXiv preprint
- top conference (1순위 학회) 미게재이면서 빅테크/유명 연구기관 소속이 아닌 경우
- 중국 대학 소속 저자 논문이면서 회사(기업) 소속이 없는 경우 (재현성 이슈 우려)
    - 단, 학회에 게재된 논문이고 인용 수 200 이상이면 3순위로 올릴 수 있음

각 논문을 찾았을 때 Semantic Scholar 또는 Google Scholar에서 인용 수를 확인하고 우선순위를 결정한다.

논문 연도 기준 (위 우선순위보다 낮은 보조 조건):
- 최근 논문(5년 이내)을 우선한다. 같은 우선순위 내에서는 최신 논문을 먼저 처리한다.
- 5년 이상 된 논문은 인용 수가 유의하게 많은 경우(대략 500회 이상, 또는 해당 분야 평균 대비 현저히 높은 경우)에만 포함한다. 단, ViT 원논문처럼 해당 분야의 기반이 되는 seminal work는 연도/인용 수 무관하게 포함한다.

### 방법론 근본성 가중치 (Method Fundamentality Weight)

위 우선순위와 별개로, Phase 3 진행 중 method-tracker에서 누적된 데이터를 기반으로 아래 가중치를 부여한다. 이 가중치가 높으면 같은 순위 내에서 먼저 처리한다. 상위 순위로 승격도 가능하다 (예: 3순위 논문이라도 가중치 +3이면 2순위로 올림).

- +3: 5편 이상의 다른 논문에서 baseline으로 사용된 메소드를 제안한 논문
- +2: 해당 메소드에서 파생된 변형(variant)이 3개 이상 존재하는 논문
- +2: 독립적인 3편 이상의 논문에서 consistent하게 높은 성능이 보고된 메소드
- +1: 단순한 구조 (컴포넌트 수 ≤ 3, 하이퍼파라미터 ≤ 2)로 좋은 성능을 내는 논문
- -1: 복잡한 구조 (컴포넌트 수 ≥ 6 또는 하이퍼파라미터 ≥ 5)인데 타 논문에서 재현이 어렵다고 언급된 논문

이 가중치는 Phase 3의 배치 처리 도중 method-tracker 업데이트를 통해 동적으로 반영된다. 큐를 재정렬할 때마다 가중치를 반영해 순서를 갱신한다.

### Phase 2: 시드 논문 수집

전체 200편을 한꺼번에 리스트업하지 않는다. 시드 논문 15-25편을 먼저 수집하고, Phase 3에서 각 논문의 related works를 따라가며 리스트를 점진적으로 확장한다.

웹 검색으로 시드 논문 수집:

WebSearch 도구와 WebFetch 도구를 적극적으로 활용한다. 검색 엔진과 학술 데이터베이스를 다양하게 조합해서 최대한 많은 논문을 발굴한다.

검색 쿼리 패턴:
- `site:arxiv.org [키워드]`
- `site:proceedings.mlr.press [키워드]`
- `site:proceedings.neurips.cc [키워드]`
- `site:openreview.net [키워드]`
- `[키워드] site:dl.acm.org`
- `[키워드] NeurIPS OR ICML OR ICLR 2020 2021 2022 2023 2024`

Semantic Scholar API (논문 메타데이터 + 인용 수 수집에 최적):
- `https://api.semanticscholar.org/graph/v1/paper/search?query=[키워드]&fields=title,year,venue,citationCount,externalIds,authors`

Papers With Code (최신 SOTA 추적에 유용):
- `https://paperswithcode.com/search?q_meta=&q_type=&q=[키워드]`

엔지니어링 블로그 검색 (주제와 관련성 높은 곳을 선별해서 검색):

빅테크 리서치 블로그:
- `site:research.google [키워드]`
- `site:ai.googleblog.com [키워드]`
- `site:deepmind.google [키워드]`
- `site:research.facebook.com [키워드]`
- `site:microsoft.com/en-us/research [키워드]`
- `site:apple.com/research [키워드]`
- `site:machinelearning.apple.com [키워드]`
- `site:amazon.science [키워드]`
- `site:openai.com/research [키워드]`
- `site:anthropic.com/research [키워드]`

빅테크 엔지니어링 블로그:
- `site:engineering.fb.com [키워드]`
- `site:netflixtechblog.com [키워드]`
- `site:eng.uber.com [키워드]`
- `site:engineering.linkedin.com [키워드]`
- `site:eng.snap.com [키워드]`
- `site:research.bytedance.com [키워드]`
- `site:engineering.atspotify.com [키워드]`
- `site:doordash.engineering [키워드]`
- `site:eng.lyft.com [키워드]`
- `site:medium.com/airbnb-engineering [키워드]`
- `site:medium.com/pinterest-engineering [키워드]`
- `site:medium.com/twitter-engineering [키워드]`
- `site:shopify.engineering [키워드]`
- `site:stripe.com/blog/engineering [키워드]`

아시아 빅테크:
- `site:d2.naver.com [키워드]`
- `site:tech.kakao.com [키워드]`
- `site:engineering.linecorp.com [키워드]`
- `site:medium.com/coupang-engineering [키워드]`

ML/AI 전문:
- `site:huggingface.co/blog [키워드]`
- `site:wandb.ai/fully-connected [키워드]`

시드 수집 완료 후, 주제 특성을 판단하여 목표 편수(200편)가 적절한지 확인한다. 관련 논문이 충분히 존재하는 주제인지, 아니면 매우 좁은 분야라 50-100편이 더 현실적인지 평가한다. 200편이 과도하다고 판단되면 Phase 2를 멈추고 사용자에게 목표 편수를 확인한다. 사용자가 승인한 목표 편수로 이후 Phase를 진행한다.

시드 수집 완료 후 두 개의 파일을 생성한다:
1. `[주제]-queue.md` — 논문 처리 큐
2. `[주제]-method-tracker.md` — 방법론 근본성 추적 테이블 (초기에는 빈 테이블. Phase 3에서 채워진다)

`[주제]-queue.md` 형식:
```
# [주제] 논문 큐

## 처리 대기 (To Process)
- [제목 또는 arxiv URL] | [출처] | [연관성: 핵심/관련/주변] | [최종 순위 1/2/3/4]
- ...

## 처리 완료 (Done)
- [파일명.md] | [제목] | [처리일]
- ...

## 제외 (Skipped)
- [제목] | [제외 이유]
- ...
```

### 큐 고갈 대응 전략

큐가 목표 편수(200편)를 채우지 못하고 소진될 위험이 있을 때, 아래 순서로 확장한다. 반드시 0단계부터 시작한다.

#### 0단계 (최우선): 이미 받아둔 PDF에서 인용 논문 수확

인접 분야 확장으로 넘어가기 전에, `read-papers/` 폴더에 이미 다운로드된 PDF들을 전수 검토해서 새로운 논문 후보를 수확한다. 이 단계는 웹 검색보다 우선한다.

수확 절차:

1. `read-papers/` 폴더의 모든 PDF 파일 목록을 가져온다.
2. 각 PDF를 Read 도구로 열어 Introduction과 Related Works 섹션을 읽는다.
3. 두 섹션에서 인용된 모든 논문 제목을 추출한다. 본문에서 "[논문 제목] [인용번호]" 형태로 언급되거나, References 섹션에 나열된 항목들을 모두 추출한다.
4. 추출된 논문 목록에서 이미 큐에 있거나("처리 대기" 또는 "처리 완료" 또는 "제외") 이미 `read-papers/`에 파일이 있는 것을 제외한다.
5. 남은 신규 후보 논문들에 대해 Semantic Scholar에서 인용 수 및 학회 정보를 확인하고 우선순위를 부여한다 (논문 우선순위 기준 섹션 참조).
6. requirements 파일에서 사용자가 설정한 조사 범위 및 요구사항을 다시 확인하고, 그 기준에 더 부합하는 논문을 높은 우선순위로 배치한다.
    - requirements의 "우선 포함할 내용"에 명시된 세부 주제와 관련된 논문은 같은 순위 내에서 앞으로 이동
    - requirements의 "제외할 내용"에 해당하는 논문은 "제외"로 처리
7. 신규 후보 논문을 큐 "처리 대기"에 추가한다. 출처를 "PDF 수확: [원본 PDF 파일명]"으로 명시한다.
8. 큐를 재정렬한다: 우선순위 기준 + method-tracker 가중치 + requirements 적합도 순으로.

이 단계에서 충분히 확보됐으면 (처리 대기 > 50편) 1단계 이하로 넘어가지 않는다. 아직 부족하면 1단계로 계속 진행한다.

#### 1단계 이후: 인접 분야 확장

0단계 수행 후에도 여전히 큐가 부족할 때, 인접 분야로 확장한다. 아래 우선순위로 확장한다:

1. 조사 중인 주제의 하위 주제 또는 세부 방법론 탐색
2. 같은 문제를 다른 도메인(modality)에서 다루는 논문:
    - 예: "추천 시스템 scaling law"가 부족하면 → "일반 scaling law" 확장
    - 이때 도메인 우선순위: vision/speech 등 비NLP 도메인 먼저, NLP는 마지막
    - 이유: NLP scaling law 논문은 너무 많아서 관련성 낮은 논문이 섞일 수 있음
    - NLP에서 발견된 핵심 원리가 다른 도메인에 어떻게 전파됐는지에 집중
3. 이 주제와 관련된 baseline/foundation 방법론 논문
4. 이 주제를 응용하거나 비판하는 논문

확장 시 `[주제]-requirements.md`의 "조사 범위 및 제약" 섹션에 확장 내용을 기록한다. 큐에 추가할 때 출처를 "인접 분야 확장: [이유]"로 명시한다.

큐 상태 체크 기준:
- 처리 대기 < 50편이 되면 즉시 0단계(PDF 수확)부터 시작해서 확장을 시도한다. 기다리지 않는다.

### Phase 3: 배치 처리 + 리스트 갱신 반복 (read-papers/ 폴더)

큐는 항상 200편 규모로 유지한다. 20편씩 읽고, 읽은 논문들의 related works를 반영해 큐를 갱신하는 방식으로 반복한다. 200편을 모두 처리할 때까지 계속한다.

배치 1회 처리 절차 (20편 단위):

1. 큐 "처리 대기"에서 우선순위 상위 20편을 꺼낸다.

2. 20편을 4-5개 그룹으로 나눠 Agent 도구로 subagent에게 병렬 위임한다.
   - 각 subagent는 4-5편을 독립적으로 처리한다.
   - subagent prompt에는 아래 내용을 포함한다:
     - 처리할 논문 목록 (제목, URL, 우선순위)
     - 출력 폴더 경로 (`read-papers/` 절대 경로)
     - 파일 이름 규칙 (이 문서의 "파일 이름 규칙" 섹션 전체)
     - 마크다운 파일 작성 형식 (이 문서의 "개별 논문 마크다운 파일 형식" 섹션 전체)
     - PDF 다운로드 지침 (이 문서의 "PDF 다운로드 지침" 섹션 전체)
     - method-tracker 업데이트용 추출 데이터: baseline 목록, variant 목록, 성능 수치, 컴포넌트 수
     - 신규 논문 후보 추출 지시: 각 논문의 Introduction과 Related Works 섹션을 읽으면서 인용된 논문들 중 이 배치 주제와 관련성 있는 것들을 모두 수집해 응답에 포함할 것. 형식: `[논문 제목] | [arxiv/URL 있으면 포함] | [연관성: 핵심/관련/주변]`. 큐 중복 여부는 main agent가 판단하므로 subagent는 필터링하지 말고 발견된 것을 모두 반환한다.
   - 모든 subagent가 완료될 때까지 기다린다.

3. 각 subagent의 결과에서 두 종류의 데이터를 수집한다:

   [method-tracker 데이터]
   - 이 논문이 실험에서 baseline으로 비교한 메소드 목록 (Experiments 또는 Comparison 섹션)
   - 이 논문이 Related Work에서 "OO의 변형" 또는 "OO을 확장"이라고 언급한 메소드
   - 보고된 baseline 성능 수치 (어떤 데이터셋에서 얼마인지)
   - 이 논문의 메소드가 얼마나 단순한지 (핵심 컴포넌트 수, 추가 하이퍼파라미터 수)

   수집 후 `[주제]-method-tracker.md`를 업데이트한다:
   - 새로 등장한 메소드를 테이블에 추가
   - 기존 메소드가 이 논문에서 baseline으로 쓰였으면 "baseline 언급 횟수" +1
   - 기존 메소드에서 파생된 변형이 이 논문이면 원본 메소드의 "파생 variant 수" +1
   - baseline 성능 수치를 "독립 측정 성능" 컬럼에 추가

   [신규 논문 후보 데이터]
   - 각 subagent가 반환한 신규 논문 후보 목록을 모두 합친다.
   - 이미 큐("처리 대기", "처리 완료", "제외") 또는 `read-papers/`에 있는 논문은 제거한다.
   - 남은 후보들을 4단계로 넘긴다.

4. 3단계에서 수집한 신규 논문 후보를 큐에 추가한다:
   - Semantic Scholar 또는 Google Scholar에서 인용 수·학회 정보를 확인해 우선순위를 부여한다.
   - method-tracker에서 baseline 언급 횟수가 높은 메소드를 제안한 논문이 있으면 우선순위를 높인다.
   - 새로 발굴된 논문을 큐 "처리 대기"에 추가한다. 출처는 "Related Work 수확: [원본 논문 파일명]"으로 명시한다.

5. 큐를 재정렬한다:
   - "처리 대기" 전체를 우선순위 기준으로 다시 정렬한다.
   - method-tracker의 가중치를 반영해 순위를 조정한다 (방법론 근본성 가중치 참조).
   - 큐가 200편을 초과하면 하위 우선순위 항목을 "제외"로 이동시켜 200편으로 유지한다.
   - 큐가 50편 미만이면 즉시 "큐 고갈 대응 전략"을 실행해서 보충한다.

6. `[주제]-queue.md`를 업데이트한다: 처리 완료 20편을 "처리 완료"로 이동, 큐 재정렬 결과 반영.

7. 진행 상황을 한 줄로 출력한다. 예: `[진행] 완료 40편 / 대기 160편 / 목표 200편 | tracker 상위: MethodA(5회), MethodB(4회)`

8. 다음 배치로 넘어간다.

관련성 Low인 논문은 논문 마크다운 파일 생성 대신 큐의 "제외" 항목에 한 줄 메모만 남긴다.

200편 처리 완료 또는 큐가 완전히 소진되면 Phase 3.5로 넘어간다.

### Phase 3.5: Method Tracker 최종 정리

Phase 3이 끝나면 `[주제]-method-tracker.md`를 최종 정리한다:
- 테이블을 "baseline 언급 횟수" 내림차순으로 정렬
- 단순성 점수, 성능 일관성 점수를 계산해서 "근본성 종합 점수" 컬럼 추가
- 상위 10개 메소드에 대해 "왜 이 메소드가 근본적인가" 한 줄 설명 추가

이 데이터는 Phase 5의 executive summary에서 "가장 근본적인 메소드" 섹션에 활용된다.

### Phase 3.7: 역인용 관계 정리 (Cross-Reference Mapping)

Phase 3.5 완료 후 반드시 이 단계를 실행한다. Phase 4(분류 체계 수립)는 이 단계가 끝나기 전에 절대로 시작하지 않는다.

목적: `read-papers/` 안의 논문들이 서로를 어떻게 평가하고 언급하는지를 개별 마크다운 파일에 기록한다. 이를 통해 각 논문의 마크다운 파일이 "내가 다른 논문을 어떻게 보는가" + "다른 논문들이 나를 어떻게 보는가" 두 방향을 모두 담게 된다.

처리 절차:

1. `read-papers/` 폴더의 모든 `.md` 파일 목록을 가져온다 (PDF 파일 제외).

2. 각 논문 A의 마크다운 파일을 읽어서 Introduction과 Related Works 섹션에서 다른 논문을 언급하는 문장을 모두 추출한다. 추출 기준:
   - 다른 논문의 제목이나 저자명을 직접 언급하는 문장
   - "[X]는 Y를 제안했다", "[X]와 달리 우리는", "[X]의 방법을 확장하여" 등 평가적 언급
   - "기존 방법의 한계"로 특정 논문을 지목하는 문장

3. 언급된 논문이 `read-papers/` 내에 마크다운 파일로 존재하는지 확인한다. 존재하면 해당 파일(논문 B의 마크다운)에 아래 섹션을 추가하거나 업데이트한다:

   ```markdown
   ## 이 논문을 언급한 논문들 (역인용 맵)

   | 언급한 논문 | 언급 맥락 | 원문 요약 |
   |------------|----------|----------|
   | [논문 A 파일명.md](./논문A파일명.md) | [어떤 섹션에서 언급했는지: Related Work / Introduction / Experiments] | [어떻게 평가했는지 한 줄: "baseline으로 사용", "한계를 지적함", "우리 방법의 출발점", "성능이 낮다고 보고" 등] |
   | ... | ... | ... |
   ```

4. 이 작업을 `read-papers/` 내 전체 마크다운 파일에 대해 완료한다. 병렬 처리가 가능하면 Agent 도구로 나눠서 처리한다 (논문 20편 단위로 그룹화해서 subagent에게 위임).

5. 작업 완료 후 한 줄로 출력한다. 예: `[Phase 3.7 완료] 역인용 맵 생성: 총 X편 논문, Y건 역인용 관계 기록`

이 단계가 완전히 끝난 후에만 Phase 4로 넘어간다.

### Phase 4: 분류 체계 수립

주의: Phase 3.7(역인용 관계 정리)이 완전히 끝난 후에만 이 Phase를 시작한다.

수집된 논문을 주제에 맞게 카테고리로 분류한다:
- 주제의 핵심 측면들을 카테고리로 구성
- 각 카테고리에 5-15편 배치
- 관련성 낮은 논문은 "참고 문헌" 카테고리로 분리

### Phase 5: 결과물 작성 및 커버리지 평가

주의: Phase 3.7(역인용 관계 정리)이 완전히 끝난 후에만 이 Phase를 시작한다. 역인용 맵이 미완성인 상태에서 summary/review 파일을 작성하지 않는다.

세 개의 파일을 완성한다:

#### 파일 1: `[주제]-requirements.md` (요구사항) - Phase 1에서 생성, 여기서 업데이트

"실제 검색 결과 요약" 섹션을 채워 넣는다. 총 논문 수, 카테고리 수, 주요 발견 사항을 기록한다.

#### 파일 2: `[주제]-literature-review.md` (메인 리뷰)

전체 논문 목록과 카테고리별 요약. 각 논문 항목에 `read-papers/` 폴더 내 개별 파일 링크를 포함한다.

#### 파일 3: `[주제]-executive-summary.md` (요약)

의사결정자를 위한 2-3페이지 요약. 모든 주장과 권장사항은 반드시 논문 full name으로 출처를 명시한다. 약칭(예: "PoLL 논문") 대신 "저자 et al., 논문 제목, 학회 연도" 형식으로 인용한다.

executive summary에는 반드시 "가장 근본적인 메소드" 섹션을 포함한다. method-tracker 최종 데이터를 기반으로 상위 5개 메소드를 기술한다.

#### 파일 4: `[주제]-method-tracker.md` (방법론 근본성 추적) - Phase 3에서 점진적으로 채워짐

아래 형식을 따른다 (위의 폴더 구조에 포함됨).

#### Phase 5 종료 조건: 커버리지 평가

파일 2와 파일 3 초안을 작성한 후, 멈추기 전에 반드시 커버리지 평가를 수행한다.

평가 방법:

1. requirements 파일("요청 내용", "우선 포함할 내용", "핵심 키워드")에 명시된 모든 항목을 목록화한다.
2. 작성된 literature-review와 executive-summary가 각 항목을 얼마나 다루고 있는지 평가한다.
   - "다루고 있다"의 기준: 해당 항목에 대해 의사결정에 충분한 근거(논문 수, 분석 깊이)가 존재하는가.
3. 전체 커버리지를 퍼센트로 산출한다. 예: "requirements 20개 항목 중 18개 커버 = 90%"

커버리지 결과에 따른 행동:

- 커버리지 ≥ 95%이고 executive summary만 봐도 의사결정이 가능하다고 판단되면: 최종 완료. 사용자에게 완료 보고를 한다.
- 커버리지 < 95%이거나 특정 항목이 얇게 다뤄졌다고 판단되면:
    1. 어떤 requirements 항목이 부족한지 명시한다.
    2. 부족한 항목을 채우기 위한 논문 100편을 추가로 수집한다. 큐에 넣고 Phase 3부터 다시 반복한다.
    3. 추가 100편 처리 완료 후 Phase 3.5 → 3.7 → 4 → 5를 다시 수행한다.
    4. 다시 커버리지를 평가하고 ≥ 95% 달성 시 종료한다.
    - 이 반복은 커버리지 ≥ 95%가 될 때까지 계속한다. 단, 총 누적 논문 수가 500편을 초과하면 사용자에게 현황을 보고하고 진행 여부를 확인한다.

## PDF 다운로드 지침

모든 논문에 대해 PDF를 반드시 다운로드하려고 적극적으로 시도한다. 스킵하지 않는다.

### PDF 찾는 순서

1. arxiv ID가 있으면: `curl -L https://arxiv.org/pdf/{id} -o "read-papers/{파일명}.pdf"`
2. OpenReview에 있으면: OpenReview PDF URL로 직접 다운로드
3. 학회 proceedings 공식 사이트: proceedings.mlr.press, proceedings.neurips.cc, openaccess.thecvf.com 등
4. WebSearch로 PDF 직접 링크 검색: `"{논문 제목}" filetype:pdf` 또는 `"{논문 제목}" site:arxiv.org OR site:openreview.net`
5. Semantic Scholar의 "Open Access PDF" 링크: `https://api.semanticscholar.org/graph/v1/paper/search?query=[제목]&fields=title,openAccessPdf`
6. 저자 개인 홈페이지, 기관 페이지, ResearchGate 등에서 WebFetch로 탐색

### PDF 접근 불가 시

유료 결제(ACM DL, IEEE Xplore, Springer, Elsevier 등)가 필요한 논문이면:
- PDF 다운로드를 포기하고 마크다운 파일의 "메타 정보" 섹션에 아래 문구를 명시한다:
  `PDF 다운로드 불가: 유료 결제 필요 ([ACM DL / IEEE Xplore / 기타]). 무료 버전 없음.`
- 마크다운 파일은 abstract, WebFetch로 가져온 정보 등 접근 가능한 내용만으로 작성한다.

### 중요

- PDF 없이 마크다운 파일만 있는 상태로 넘어가지 않는다. 위의 모든 경로를 시도한다.
- 다운로드 성공 시: 마크다운 파일명과 동일한 이름(확장자만 .pdf)으로 `read-papers/` 폴더에 저장.
- PDF 파일명 예시: `2025_NeurIPS_MethodName_Paper-Title.pdf` (마크다운이 `2025_NeurIPS_MethodName_Paper-Title.md`일 때)

## 개별 논문 마크다운 파일 형식

`read-papers/` 폴더에 저장되는 각 논문 마크다운 파일은 아래 템플릿을 따른다.

```markdown
원본 링크: [URL 또는 제목]
처리일: YYYY-MM-DD

# [논문 전체 제목]

## 메타 정보

- 제목:
- 저자:
- 소속 기관:
- 학회/저널/회사:
- 연도:
- arXiv: (있으면)
- 인용 수: (Google Scholar 기준, 없으면 "정보 없음")
- 코드: (GitHub 링크 또는 "비공개")

## 한 줄 elevator pitch

이 논문이 뭔지 한 문장으로.

## 세 줄 요약

1.
2.
3.

## 해결하려는 문제

기존 방법이 왜 안 되는지, 어떤 상황에서 실패하는지. 고등학생도 이해할 수 있게 설명.

## 방법론

핵심 메커니즘을 고등학생도 이해할 수 있게 설명. 핵심 수식이 있으면 포함.

## "뭉개서 푼" 여부

아래 기준으로 평가:
- 문제 증상만 처리했는지 vs 근본 원인 해결인지
- 컴포넌트 수와 하이퍼파라미터 수 (많을수록 뭉개서 푼 것)
- 동시에 3개 이상 원인을 해결한다고 주장하는지
- 결론: [직접 해결 / 뭉개서 해결 / 혼합] + 근거

## 기존 방법과의 차별점 및 성능

- 주요 비교 대상:
- 무엇이 다른지:
- 정량적 성능 차이: (데이터셋, 지표 명시)
- 실험 환경: (어떤 데이터셋, 어떤 설정)

## 한계 및 가정

- 저자가 인정한 한계:
- 암묵적 가정:
- 비판적으로 찾은 약점:

## 인용한 논문들의 평가

citing papers에서 이 논문을 어떻게 언급하는지, 성능이 재현됐는지.
최신 논문이라 정보가 없으면: "정보 없음 (논문이 너무 최신)"

## 이 논문을 언급한 논문들 (역인용 맵)

Phase 3.7에서 자동으로 채워진다. 초기에는 비워둔다.

| 언급한 논문 | 섹션 | 언급 내용 요약 |
|------------|------|--------------|
| (Phase 3.7에서 채워짐) | | |

## 커뮤니티 반응 검증

Reddit, X.com, Hacker News, 웹에서 찾은 반응을 정리한다.
각 반응이 맞는 말인지 근거와 함께 평가한다.
없으면 "정보 없음"으로 표기.

## 배경 지식

이 논문을 이해하려면 알아야 할 선행 개념 목록:
-

## 함께 읽으면 좋은 논문

- [논문 제목] - [이유 한 줄]

## 우리 업무 연관성

moderation / recommendation / AI Lab 관점에서 적용 가능성.
해당 없으면 "해당 없음".
```

커뮤니티 반응 검색 방법:
각 논문에 대해 다음 소스를 WebSearch로 검색한다:
1. Reddit: `site:reddit.com "{논문 제목}"` 또는 `site:reddit.com "{메소드명}"`
2. X.com (Twitter): `"{논문 제목}" site:x.com` 또는 `"{메소드명}" twitter`
3. Hacker News: `site:news.ycombinator.com "{논문 제목}"`
4. 기타 웹: `"{논문 제목}" reaction` 또는 `"{논문 제목}" discussion`

## 결과물 형식

### Requirements 파일 템플릿

```markdown
날짜: YYYY-MM-DD
주제: [주제]

# [주제] - 조사 요구사항

## 요청 내용

[사용자가 요청한 내용을 그대로 기록. 추가 제약이나 맥락이 있으면 함께 기록]

## 핵심 키워드

- [키워드 1]
- [키워드 2]
- [키워드 3]
...

## 검색 대상 학회/학술지

[주제와 관련성 높다고 판단한 학회/저널 목록 및 선택 이유]

## 검색 대상 엔지니어링 블로그

[검색할 블로그 목록]

## 검색 쿼리 목록

[실제 사용할 검색 쿼리들을 미리 나열]

## 조사 범위 및 제약

- 목표 논문 수: 200편 (100-300편 범위)
- 연도 범위: [기준 연도] 이후 (특별한 seminal work 제외)
- 우선 포함할 내용: [주제와 특히 관련성 높은 세부 주제나 방법론]
- 제외할 내용: [범위에서 제외하기로 한 것이 있다면 기록]
- 인접 분야 확장 계획: [큐 고갈 시 확장할 분야 목록]

## 실제 검색 결과 요약

[조사 완료 후 채워 넣는 섹션]

- 총 논문 수: [숫자]
- 카테고리 수: [숫자]
- 주요 발견: [간략히]
```

### 메인 리뷰 파일 템플릿

```markdown
날짜: YYYY-MM-DD
주제: [주제]
논문 수: [총 논문 수]

# [주제] 문헌 리뷰

## Executive Summary

[3-5 문장으로 이 분야의 핵심 발견사항 요약]

### 가장 유망한 접근법

1. [접근법 1]: [한 줄 설명]
2. [접근법 2]: [한 줄 설명]
3. [접근법 3]: [한 줄 설명]

### 실무 적용 권장사항

단기 (1-3개월):
- [권장사항]

중기 (3-6개월):
- [권장사항]

---

## 1. [카테고리 1] ([논문 수]편)

[카테고리 설명: 왜 이 카테고리가 주제와 관련되는지]

### [논문 제목] ([학회/저널] [연도])

- 출처: [학회/저널명] [연도]
- 상세 분석: [read-papers/파일명.md](./read-papers/파일명.md) (있는 경우)

[주제]와의 관련성:
[이 논문이 조사 주제와 어떻게 연결되는지 1-2문장으로]

---

[다른 논문들...]

## 참고 문헌 (관련성 낮음)

[관련성 낮지만 배경 지식으로 유용한 논문들 - 간략히 제목/출처만]
```

### Method Tracker 파일 템플릿

```markdown
날짜: YYYY-MM-DD (마지막 업데이트)
주제: [주제]

# [주제] - 방법론 근본성 추적

이 파일은 Phase 3 배치 처리 중 PDF를 읽으면서 자동으로 누적된다.
각 배치 처리 후 업데이트. 최종 정렬은 Phase 3.5에서 수행.

## 방법론 테이블

| 메소드명 | 제안 논문 (연도) | baseline 언급 횟수 | 파생 variant 수 | 독립 측정 성능 (데이터셋: 수치 | 출처) | 컴포넌트 수 | 단순성 점수 (1-5) | 성능 일관성 점수 (1-5) | 근본성 종합 점수 |
|---------|----------------|-------------------|----------------|--------------------------------------|-------------|-----------------|----------------------|----------------|
| [메소드A] | [저자 et al., 연도] | 8 | 5 | ImageNet: 82.3% (논문X), 82.1% (논문Y) | 2개 | 5 | 5 | 38 |
| [메소드B] | [저자 et al., 연도] | 5 | 2 | ... | 3개 | 4 | 4 | 22 |
| ... | | | | | | | | |

## 근본성 종합 점수 계산 방법

종합 점수 = (baseline 언급 횟수 × 3) + (파생 variant 수 × 2) + (단순성 점수 × 1) + (성능 일관성 점수 × 2)

- baseline 언급 횟수: 몇 편의 다른 논문에서 이 메소드를 비교 대상으로 삼았는지
- 파생 variant 수: 이 메소드를 직접 변형/확장한 논문 수 (Related Work에서 "based on X", "variant of X" 언급)
- 단순성 점수: 5=컴포넌트 1-2개, 4=3개, 3=4개, 2=5개, 1=6개 이상
- 성능 일관성 점수: 독립 논문에서 보고된 수치들의 분산이 낮을수록 높음 (5=표준편차 <0.5%, 1=>3%)

## 상위 메소드 분석 (Phase 3.5에서 작성)

Phase 3 완료 후, 종합 점수 상위 10개 메소드에 대해:

### 1위: [메소드명] (종합 점수: N)

- 왜 근본적인가: [한 줄 설명]
- 대표 논문: [저자 et al., 제목, 학회 연도]
- 어떤 논문들이 이 메소드를 baseline으로 썼는가: [목록]
- 알려진 변형들: [목록]
- 독립 측정 성능 범위: [최소] ~ [최대] (N개 논문)

...
```

### Executive Summary 파일 템플릿

```markdown
날짜: YYYY-MM-DD

# [주제] - Executive Summary

## 핵심 질문

이 문헌 조사가 답하려는 질문:
- [질문 1]
- [질문 2]

## 주요 발견사항

[총 X편 논문 분석 결과]

1. [발견사항 1]
2. [발견사항 2]
3. [발견사항 3]

## 가장 근본적인 메소드 (method-tracker 기반)

이 섹션은 [주제]-method-tracker.md의 종합 점수 상위 5개 메소드를 요약한다.
"근본적"의 기준: (1) 다수 논문에서 baseline으로 채택됨, (2) 변형이 많이 파생됨, (3) 독립 측정에서 성능이 일관됨, (4) 구조가 단순함.

1. [메소드명] (종합 점수 N): [왜 근본적인지 한 줄]. 제안: 저자 et al., 논문 전체 제목, 학회 연도.
2. [메소드명] (종합 점수 N): [왜 근본적인지 한 줄]. 제안: 저자 et al., 논문 전체 제목, 학회 연도.
3. [메소드명] (종합 점수 N): ...
4. ...
5. ...

## 접근법 비교

| 접근법 | 장점 | 단점 | 관련 논문 |
|--------|------|------|-----------|
| [접근법 1] | [장점] | [단점] | 저자 et al., [논문 전체 제목], [학회] [연도] |
| [접근법 2] | [장점] | [단점] | 저자 et al., [논문 전체 제목], [학회] [연도] |

## 우리 상황에 가장 유망한 방향

[2-3 문단으로 Hyperconnect/Azar 맥락에서 가장 적용 가능한 방향 서술]

## 다음 단계

- [ ] [액션 아이템 1]
- [ ] [액션 아이템 2]

## 검색 범위

- 조사 기간: [날짜 범위]
- 총 논문 수: [숫자]
- 주요 학회: [목록]
- 검색 키워드: [목록]
```

## 작성 원칙

- 모든 내용은 한국어로 작성. 전문 용어 첫 등장 시 한영 병기 (예: "역방향 호환성(backward compatibility)").
- 볼드 강조(`** **`) 사용 금지.
- 설명은 고등학생도 이해할 수 있는 수준으로. 이 기준은 파일 전체에 적용된다. 어려운 개념이 나오면 반드시 쉬운 비유나 예시를 들어 설명한다.
- 수식이 있어도 그 수식이 "어떤 아이디어를 표현하는지"를 먼저 말로 설명하고, 그 다음 수식을 보여준다.
- 논문을 과도하게 긍정적으로 평가하지 말 것. 한계와 가정을 비판적으로 평가.
- 웹에서 찾을 수 없는 정보는 "정보 없음"으로 명시하고 추측하지 말 것.
- 적용 가능성 평가 시 Hyperconnect/Azar의 맥락 (추천 시스템, 모더레이션, 소규모 팀) 고려.
- 목표 논문 수는 100-300편 (200편 기준). 부족하면 인접 분야 확장 전략 실행, 초과하면 우선순위 기준으로 탈락.
- 1-2순위 논문은 깊이 있게 분석하고, 3-4순위 논문은 간략한 요약으로 처리한다.
- executive-summary.md의 모든 주장과 권장사항에는 논문 full name을 인용한다. 약칭 금지. 형식: "저자 et al., 논문 전체 제목, 학회/저널 연도".
- 조사 시작 전 현재 프로젝트의 CLAUDE.md와 상위 폴더의 CLAUDE.md를 읽어 사용자 맥락을 파악한다.

## 접근 가능성 원칙 (필수)

논문 포함 기준: 무료로 전문(full text)에 접근 가능한 논문만 포함한다.

무료 접근 가능으로 인정되는 소스:
- arxiv.org (preprint 또는 학회 발표 버전)
- openreview.net (ICLR, NeurIPS 등 OpenReview 기반 학회)
- 학회 공식 홈페이지 공개 PDF (proceedings.mlr.press, proceedings.neurips.cc, aaai.org, aclanthology.org 등)
- Semantic Scholar / Papers With Code에서 "Open Access PDF" 링크 제공되는 경우
- 저자 개인 홈페이지 또는 기관 페이지에 공개된 PDF

제외 대상: ACM DL, IEEE Xplore, Springer, Elsevier 등 유료 결제 없이 전문 접근이 불가능한 논문.

확인 방법: Semantic Scholar에서 논문 검색 후 "Open Access PDF" 버튼 여부 확인. 없으면 `site:arxiv.org [제목]` 또는 `site:openreview.net [제목]` 검색.

큐의 "제외" 항목에 "유료 접근만 가능 - 무료 버전 없음"으로 이유를 명시한다.

## Rate Limit 대응 방법

처음부터 rate limit을 피하기 위한 인위적인 지연이나 간격 조절은 하지 않는다. 그냥 진행한다.

429 오류, "Too Many Requests", 빈 결과 반환, 응답 지연 등이 발생하면 아래 절차를 따른다.

### Rate Limit 발생 시 대응 순서

1. Backoff 파일 확인: `[출력폴더]/rate-limit-backoff.md` 파일을 읽는다. 해당 서비스에 대한 마지막 성공 대기 시간이 기록돼 있으면 그 값으로 시작한다. 파일이 없거나 기록이 없으면 30초로 시작한다.

2. 대기 후 재시도: Bash 도구로 `sleep [N]`을 실행하고, 같은 요청을 다시 시도한다.

3. 결과 기록 및 조정:
   - 재시도 성공 시: `rate-limit-backoff.md`에 `서비스: [서비스명] | 대기시간: [N]초 | 결과: 성공 | 날짜: [날짜]`를 기록한다. 다음 rate limit 발생 시 이 값에서 시작한다.
   - 재시도 실패 시: 대기 시간을 2배로 늘린다 (30 → 60 → 120 → 300초, 최대 300초). 다시 대기 후 재시도한다. 300초로도 실패 시 해당 소스를 포기하고 대안 소스로 전환한다.

`rate-limit-backoff.md` 형식:
```
# Rate Limit Backoff 기록

| 서비스 | 마지막 성공 대기시간(초) | 마지막 시도일 |
|--------|------------------------|-------------|
| google-search | 60 | 2026-03-25 |
| semantic-scholar | 30 | 2026-03-25 |
| arxiv | 30 | 2026-03-25 |
```

### 검색 엔진 전환 (rate limit 극복 불가 시)

특정 소스가 지속적으로 막히면 아래 대안으로 전환한다:
- Google 검색 막힘 → Semantic Scholar API, arxiv 직접 검색, Papers With Code
- Semantic Scholar API 막힘 → Google Scholar WebSearch, arxiv 직접 검색
- arxiv 막힘 → openreview.net, 학회 proceedings 직접 접근

큐 항목에 "접근 불가 - rate limit, 대안 소스로 전환" 메모를 남기고 계속 진행한다.

## 주의사항

- WebSearch 결과만으로 판단하지 말고, 주요 논문은 WebFetch로 abstract/intro를 직접 읽는다.
- arxiv 논문의 경우 학회 게재 여부를 확인한다 (arxiv preprint와 학회 발표 논문 구분).
- 인용 수가 높은 논문일수록 더 자세히 다룬다.
- 2015년 이전 논문은 특별히 중요한 경우 (seminal work)만 포함.
- 완료 후 생성된 파일 목록을 출력한다.
