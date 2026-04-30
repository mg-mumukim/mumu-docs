# Report Plan: Agent Skill Taxonomy — Anthropic & OpenAI Supplement

**Date:** 2026-04-23

## Question

Which skills in `anthropics/skills` and `openai/skills` fit the existing 11-type taxonomy (v4), and which ones reveal types, sub-variants, or intra-skill components not yet covered? How should the taxonomy be updated to reflect them?

## Sources to check

1. **GitHub repo: anthropics/skills** — Read README and all skill files (SKILL.md / equivalent).
2. **GitHub repo: openai/skills** — Read README and all skill files.
3. **work/agent-skill-taxonomy/agent-skill-taxonomy-report-v4.md** — Current taxonomy (already read).

## Scope

- In scope: Matching each skill to one of the 11 existing types; identifying skills that don't fit and classifying them; identifying new intra-skill components not in Appendix B; proposing taxonomy additions or structural improvements.
- Out of scope: Rewriting the existing taxonomy findings for already-covered types. Runtime execution, model selection, performance benchmarks.

## Method

1. Read all skills from both repos.
2. For each skill: assign to an existing type, or flag as "unclassified."
3. Count matched skills by type (for Conclusion and Appendix C update).
4. Group unclassified skills and determine: new type, new sub-variant of an existing type, or new intra-skill component.
5. Propose taxonomy additions/improvements; produce v5.

## Output

- Report postfix: `-report`
- Target path: `work/agent-skill-taxonomy/`
- Output file: `agent-skill-taxonomy-report-v5.md`
