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

### `work/` — Working documents

Structure: `work/<project>/yyyy-MM-dd-<topic>-v<N>.md`

Claude creates new version files. Human reviews each version and leaves feedback as `[!NOTE]` callouts. Claude incorporates feedback into the next version.

## Skills

| Skill | Description |
|-------|-------------|
| `/writing-draft` | Draft or revise a deliverable document from `note/` sources into `work/<topic>`. |
| `/writing-report` | Research a topic and write a report to `work/<topic>`. Requires plan approval. |
| `/writing-review` | Evaluate an existing document against its stated purpose and produce structured findings. |
| `/importing-gdocs` | Download a Google Doc as Markdown and save it as a source note under `note/source/`. |
