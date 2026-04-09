# Document workflow

- `00_note/`: Human-only writes. Claude reads as source material.
- `01_work/`: Claude writes versioned files (`-v1`, `-v2`, ...). Never modify existing versions. Create next version incorporating `[!NOTE]` feedback.
- `02_record/`: Immutable. Claude moves files here only with explicit human approval, stripping version suffix.
