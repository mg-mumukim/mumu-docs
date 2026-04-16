# Mumu Docs

## Document flow

```
note/  →  work/
(source)  (output)
```

Source material collected in `note/` is turned into versioned working documents in `work/` by Claude.

## Directory structure

### `note/` — Source material

File naming: `yyyy-MM-dd-[a-z0-9-].md`

| Directory | Writer | Description |
|-----------|--------|-------------|
| `source/` | Human | Raw inputs: meeting notes, system context, plans, prompts, CSVs, discussion notes. |
| `task/` | Human or Claude | Requests (`-request`), draft plans (`-draft-plan`), report plans (`-report-plan`). |
| `context/` | Human | Persistent reference material for cross-project use. |
| `idea/` | Claude via `/idea` | Quick ideas captured by topic. |
| `external/` | Human | External reference documents (e.g. RFCs). |
| `personal/` | Human | Private notes (1:1, evaluations). Git-ignored. |

### `work/` — Working documents

Structure: `work/<project>/yyyy-MM-dd-<topic>-v<N>.md`

Claude creates new version files. Human reviews each version and leaves feedback as `[!NOTE]` callouts. Claude incorporates feedback into the next version.

## Skills

| Skill | Description |
|-------|-------------|
| `/idea` | Capture a quick idea into `note/idea/` as a timestamped entry. |
| `/write-draft` | Draft or revise a deliverable document from `note/` sources into `work/<topic>`. |
| `/write-report` | Research a topic and write a report to `work/<topic>`. Requires plan approval. |
| `/self-review` | Review a `work/` document against writing conventions. Read-only. |
