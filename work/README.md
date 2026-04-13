# Work

Active working documents organized by project. Approved documents graduate to `note/task/` via `/publish`.

Structure: `work/<project>/yyyy-MM-dd-<topic>-v<N>.md`

Example: `work/photo-review/2026-04-10-photo-review-llm-cost-estimation-v1.md`

## Ownership

- **Writer**: Claude creates new version files. Human does not write here directly.
- **Reviewer**: Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments within the file.

## Versioning Rules

1. Claude never modifies existing version files directly.
2. Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments.
3. Claude incorporates the feedback into a new version file (`-v2`, `-v3`, ...).
4. Previous version files are preserved as-is (history).

## Graduation

When human gives final approval, the approved version file is published to `note/task/` (without `-wip` suffix). Claude performs this only after explicit human permission via `/publish`.
