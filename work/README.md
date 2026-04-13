# Work

Active working documents organized by project. The latest version is the canonical final document.

Structure: `work/<project>/yyyy-MM-dd-<topic>-v<N>.md`

Example: `work/photo-review-llm-cost-estimation/2026-04-10-photo-review-llm-cost-estimation-v1.md`

## Ownership

- **Writer**: Claude creates new version files. Human does not write here directly.
- **Reviewer**: Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments within the file.

## Versioning Rules

1. Claude never modifies existing version files directly.
2. Human reviews each version and leaves feedback as `[!NOTE]` callouts or inline comments.
3. Claude incorporates the feedback into a new version file (`-v2`, `-v3`, ...).
4. Previous version files are preserved as-is (history).

## Cross-project reference

If a finalized document needs to be referenced by other projects, use `/publish` to copy it to `note/context/` (version suffix stripped).
