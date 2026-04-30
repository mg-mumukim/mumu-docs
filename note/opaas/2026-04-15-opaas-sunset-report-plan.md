# OPaaS Sunset Report Plan

## Question
OPaaS(On-device Platform as a Service)를 sunset하기 위해 현재 운영 중인 컴포넌트 — k8s 워크로드, DB, Kafka 토픽, 자동화 파이프라인, API 키 — 는 무엇이고, prod/dev/qa 환경별로 어떤 식별자(ID, 이름)를 가지며, 올바른 셧다운 순서는 무엇인가?

## Sources to check
- **Notion** — OPaaS, Top Photo, Photo Ordering, Photo Selector, Photo Selection, Photo Finder, Photo Picker 관련 프로젝트 페이지, 아키텍처 문서, 인프라 가이드
- **Glean** — "OPaaS", "on-device platform", "top photo", "photo ordering", "photo selector" 키워드로 사내 문서 통합 검색
- **Google Docs** — Notion에 없는 설계 문서, Hinge 협업 문서 (Glean을 통해 접근)
- **Jira** — OPaaS/Photo 관련 에픽/티켓에서 인프라 컴포넌트 식별자 수집; 주요 개발 멤버: Parker, Joel
- **Slack** — engineering 채널에서 OPaaS/photo 서비스 관련 논의, 운영 이슈, 셧다운 논의; 주요 관련자: Mario
- **GitHub (matchgroup-ai)** — 관련 레포 코드에서 deployed 컴포넌트 이름, k8s manifest, Kafka topic 이름 추출

## Scope
- **In**: 인프라 컴포넌트 목록 및 환경별 식별자, 서비스 간 의존 관계, 셧다운 순서 및 체크리스트
- **Out**: ML 모델 성능, 비즈니스 전략, 실험 결과, 서비스 재설계 제안

## Output
`-shutdown-guide`
