# Document workflow

- `00_note/`: Human-only writes. Claude reads as source material.
- `01_work/`: Claude writes versioned files (`-v1`, `-v2`, ...). Never modify existing versions. Create next version incorporating `[!NOTE]` feedback.
- `02_record/`: Immutable. Claude publishes files here via `/publish` only with explicit human approval, stripping version suffix.

# Reference documents

When drafting or revising any project-related document, always read and reference:
- `02_record/2026-04-09-mumu-kim-profile-summary.md` -- Mumu Kim's profile summary (projects, competencies, people, channels)
