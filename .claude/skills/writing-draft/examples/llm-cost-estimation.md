# Example: LLM Cost Estimation Draft

## Invocation

```
/writing-draft photo-review-cost-estimation
```

## Inputs

- `note/task/2026-04-10-photo-review-llm-cost-estimation-request.md` — draft outline with `[!NOTE]` reviewer directives
- `note/source/2026-04-10-photo-review-kevin-trideep-discussion-traffic.md` — traffic model and Slack discussion
- `note/source/2026-04-10-photo-review-ben-dbx-notebook.md` — Databricks notebook with benchmark numbers

## Output

`work/photo-review-cost-estimation/2026-04-10-llm-cost-estimation-v1.md`

Revised through v2 with `[!NOTE]` feedback in the document incorporated each cycle.

## Notes

The request file doubled as the draft outline: it contained section headings and inline
`[!NOTE]` directives specifying what each section must answer. The skill treated those
as the primary input rather than a separate request file.
