# Work

Active working documents. Approved documents graduate to `02_record`.

File naming: `yyyy-MM-dd-<topic>-v<N>.md` (e.g. `2026-04-03-auth-design-v1.md`)

Plan 등 용도가 명확한 경우 topic 안에 포함한다 (e.g. `2026-04-03-auth-design-plan-v1.md`).

## Ownership

- **Writer**: Claude creates new version files. Human does not write here directly.
- **Reviewer**: Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments within the file.

## Versioning Rules

1. Claude는 기존 버전 파일을 직접 수정하지 않는다.
2. Human이 리뷰 후 파일 안에 `[!NOTE]` 또는 요청 사항을 남긴다.
3. Claude는 해당 피드백을 반영하여 다음 버전 파일(`-v2`, `-v3`, ...)을 새로 작성한다.
4. 이전 버전 파일은 그대로 보존한다 (history).

## Graduation

Human이 최종 승인하면 해당 버전 파일을 `02_record/`로 이동한다. 이동은 human 허가 후 Claude가 수행한다.
