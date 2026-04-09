## Tinder Infra 연동

Tinder 인프라와의 연결은 크게 두 가지 방식으로 이루어집니다. 정형 데이터는 Delta Sharing을 통해 받아오고, 비정형 원본 파일은 Tinder S3 버킷이 연결된 External Volume을 통해 간접적으로 가져옵니다.

Phase 1에서는 Tinder Databricks에서 DLP에 필요한 최소 테이블만 우선 Delta Sharing으로 수집합니다. 비정형 파일의 경우 현재 Tinder S3 버킷과 직접 연결되어 있지 않아 아래와 같이 단계적으로 전환합니다.

- **Phase 1 (현행)**: External Volume 준비 전까지 업무 연속성을 위해 MLE가 Databricks 노트북에서 인스턴스 프로파일(IAM Role)을 통해 S3 API로 파일을 직접 가져오도록 임시 허용합니다.
- **Phase 2 (예정)**: 전면적인 External Volume 강제 전환을 통해 S3 직접 접근을 차단합니다.

```python
import boto3
from io import BytesIO
from PIL import Image

# Databricks 노트북에서 인스턴스 프로파일(IAM Role)로 S3 접근
s3 = boto3.client("s3")

BUCKET = "tinder-prod-media"  # Tinder 원본 S3 버킷
KEY = "photos/00/ab/cd/00abcdef1234.jpg"  # 예시 사진 경로

response = s3.get_object(Bucket=BUCKET, Key=KEY)
image_bytes = response["Body"].read()
image = Image.open(BytesIO(image_bytes))
```

Phase 1에서 DLP 목적으로 Delta Sharing으로 수집하는 테이블은 다음과 같습니다.

- 탈퇴 계정 처리를 위해서 `published_prod.tinder_events_delta.trust_case_review`, `profile_delete_photo`, `trust_account_status` 테이블에서 `uid` / `user_number`를 추출합니다.
- 삭제된 미디어(사진, 동영상 등)의 경우 `photo_deleted` / `video_deleted` / `content_purged` 이벤트를 `source_events` 계약 형태로 수집합니다.

## MG AI 인프라 설정

MG AI 인프라에서 정형 및 비정형 파생데이터는 단일 S3 버킷(`mg-ai-uw1-hyperpod-datasets`)을 공유하되, 프로젝트 상위 폴더 하위에서 내부 경로 규칙을 통해 명확히 분리하여 관리합니다.

- **정형 데이터**: Databricks Unity Catalog의 External Location으로 특정 경로 prefix(예: `/.../__unity_catalog/*`)를 연결합니다.
- **비정형 데이터**: 별도 폴더 규칙(예: `/.../images/*`)을 따릅니다.

```
/mgai/tinder/recs/__unitystorage/** → 정형
/mgai/tinder/recs/**               → 비정형
/mgai/pairs/recs/__unitystorage/** → 정형
/mgai/pairs/recs/**                → 비정형
```

## 접근 권한 제어

접근 권한 제어의 기본 방향은 "최소 권한의 원칙(Deny by default)"입니다. DLP 관련 핵심 시스템 테이블 및 매핑 테이블 원본은 인프라 담당자와 Service Principal로 접근을 엄격히 제한합니다.

### 1. 그룹 및 권한 동기화 (Databricks & HyperPod)

사용자 권한은 프로젝트 단위 그룹과 ACL을 기준으로 관리합니다.

- **Databricks 그룹**: `MGAI Aura`, `MGAI Tinder RnS` 등으로 구분되며, IaC(`mg-databricks`, `mgcore-terragrunt`)를 통해 관리합니다.
- **HyperPod 그룹**: Linux 그룹(`mgai-aura` 등)으로 매핑하여 분리 관리합니다. (현재 GitOps 기반으로 운영 중이며, Phase 2에서 Databricks 권한 변경 시 자동 PR 생성 동기화 예정)

### 2. S3 접근 통제 (API 직접 접근 차단)

Tinder 원본 및 MG AI 버킷 모두 **AWS IAM 기반 직접 접근을 차단**하고, **Databricks External Volume을 통한 접근만 허용**합니다.

- 외부 볼륨을 경유함으로써 자동 감사 로깅, ABAC, 프로젝트 권한 검사가 이루어집니다.
- Databricks 내에서도 정해진 Job / External Volume / Service Principal 경로를 통해서만 접근 가능합니다.
- **권한 수준**: 원본 데이터는 읽기(Read) 전용, 파생 데이터(MG AI 버킷)는 HyperPod 활용을 위해 쓰기(Write) 권한을 부여합니다.

### 3. 데이터 유형별 세부 접근 정책

- **Closed Table**: 읽기 전용, 필요 최소 인원에게만 공유
- **Audit Log**: 인프라 담당자 및 매니저 전용
- **정형 파생데이터**: Unity Catalog 태그, Upstream ABAC 상속, Column Masking, Baseline Grant로 통제
- **비정형 파생데이터**: 프로젝트 그룹 및 디렉터리 ACL로 분리. 데이터 Import 요청 시 허용 그룹과 S3 Prefix 교차 검증

### 4. 가명화 매핑 테이블 및 UDF 보안

매핑 테이블 원본에 대한 일반 MLE의 직접 접근(읽기/쓰기)을 전면 차단하고, 안전한 UDF 또는 마스킹 결과를 활용하도록 강제합니다.

- **역조회 원천 차단 (Phase 1)**: UDF를 통한 가명화만 허용하며 원본 ID 복원은 불가합니다.
- **실행 권한 격리 (Execute-only)**: MLE에게는 UDF의 `EXECUTE` 권한만 부여하여 내부 로직 및 원본 조회(`READ/MODIFY`)를 차단합니다.
- **이상 탐지 로깅**: UDF 호출 및 변환 시도를 감사 로그에 상시 기록합니다.

## 삭제 프로세스 및 완료 검증

삭제 요청은 접수부터 최종 검증까지 상태값으로 엄격히 추적되며, API 응답에 의존하지 않고 S3 인벤토리 스냅샷을 통한 독립적 검증을 수행합니다.

**데이터 삭제 프로세스**

1. **삭제 접수**: 탈퇴·삭제 이벤트 감지 시 내부 추적 테이블에 등록
2. **캐시 삭제**: 이력 조회 후 FSx 캐시 및 HyperPod 로컬 복사본 선행 삭제
3. **S3 삭제 검증**: 삭제 전·후의 S3 인벤토리 스냅샷(일별 생성)을 수학적으로 대조해 실제 제거 여부 확인
4. **완료 처리**: 검증 통과 시 완료 전환, 실패 시 수동 개입 알림 트리거

*참고사항 (SLA): 삭제 처리 SLA는 기본 48시간이나, Phase 1 실측 데이터를 바탕으로 추후 세밀히 조정할 계획입니다.*

## Audit 로그

감사 로그는 목적에 따라 다음과 같은 테이블에 분리 저장됩니다.

- 물리적 삭제 감사: 물리적 삭제 감사 로그
- 가명 매핑 삭제 감사: 가명화 매핑 삭제 감사 로그
- 캐시 로드 이력: 캐시 노출 및 데이터 로드 감사 로그

모든 감사 테이블에 대한 운영 접근은 인프라 담당자/매니저만 가능하도록 제한합니다.

DLP 파이프라인 대상 테이블 관리(어떤 테이블이 DLP 적용 대상인지 여부)는 별도 파이프라인 제어 테이블에서 관리하며, 위 감사 로그와는 구분됩니다.

## 예외 처리

예외 상황에 대한 처리는 다음과 같이 규정하여 자동화 및 모니터링을 수행합니다.

- **경로 위반 파일 격리**: 정기적으로(Daily) S3 인벤토리를 검사하여 경로 규칙을 위반한 파일은 즉각 접근 차단된 격리(Quarantine) 폴더로 이동시키고, 24시간 보관 후 자동 영구 삭제합니다.
- **삭제 예외 파일 (모델 Weight 등)**: 개별 파일 단위 예외가 아닌 예약된 허용 폴더(Prefix, 예: `/.../model/...`)를 지정하여 Policy Registry에서 예외 항목으로 일괄 관리합니다.
- **DLP 미적용 데이터 모니터링**: DLP 정책은 `dlp: enabled` 태그가 설정된 Managed Table에만 적용되므로, 미적용 테이블 목록을 대시보드로 상시 모니터링하여 누락을 방지합니다.
