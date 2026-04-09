# Work

Active working documents. Approved documents graduate to `02_record`.

File naming: `yyyy-MM-dd-<topic>-v<N>.md` (e.g. `2026-04-03-auth-design-v1.md`)

When the purpose is clear (e.g. plan), include it in the topic (e.g. `2026-04-03-auth-design-plan-v1.md`).

## Ownership

- **Writer**: Claude creates new version files. Human does not write here directly.
- **Reviewer**: Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments within the file.

## Versioning Rules

1. Claude never modifies existing version files directly.
2. Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments.
3. Claude incorporates the feedback into a new version file (`-v2`, `-v3`, ...).
4. Previous version files are preserved as-is (history).

## Graduation

When human gives final approval, the approved version file is moved to `02_record/`. Claude performs the move only after explicit human permission.
