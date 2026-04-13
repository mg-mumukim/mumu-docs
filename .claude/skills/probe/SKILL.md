---
name: probe
description: Research a topic and write a concise summary to `note/source/`.
---

# Usage
/probe <topic or question>

# Workflow

1. Search relevant sources (Slack, Jira, Glean, Notion, Confluence, web) for the given topic.
2. Synthesize findings into a concise, factual summary.
3. Write to `note/source/yyyy-MM-dd-<topic-slug>-<postfix>.md`. Choose a postfix that describes the output type (e.g. `-report`, `-benchmark`, `-comparison`, `-summary`, `-timeline`).
4. Include a `> **Sources**:` block at the top listing URLs or references used.

# Rules
- **Write everything in English.** All output files must be in English regardless of the user's input language.
- Keep it short. One topic per file.
- Do not editorialize. State facts and cite sources.
- Do not duplicate existing files in `note/source/`. Check first.
- If insufficient information is found, report what was found and what is missing.
