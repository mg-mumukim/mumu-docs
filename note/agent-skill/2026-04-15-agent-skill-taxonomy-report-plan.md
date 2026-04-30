# Report Plan: Agent Skill Taxonomy

**Date:** 2026-04-15

## Question
Across three agent-skill ecosystems (addyosmani/agent-skills, vercel-labs/agent-skills, EveryInc/compound-engineering-plugin), what common structural dimensions do skills share, what major types (taxonomy) can be identified, and what core subcomponents does each type contain?

## Sources to check
1. **GitHub repo: addyosmani/agent-skills** — Read README and individual skill files to understand structure and metadata.
2. **GitHub repo: vercel-labs/agent-skills** — Read README and skill definitions.
3. **GitHub repo: EveryInc/compound-engineering-plugin (plugins/compound-engineering/skills)** — Read skill files in the `skills/` subtree.
4. **note/idea/how-to-write-skill.md** — Existing local notes on skill design (if present).

## Scope
- In scope: Structure of skill definitions (metadata fields, trigger conditions, action steps, tool references, output expectations). Cross-repo commonalities and differences. Type taxonomy and subcomponent identification.
- Out of scope: Runtime execution details, performance benchmarks, security analysis, which AI model runs the skills.

## Output
Report file postfix: `-report`
Target path: `work/agent-skill-taxonomy/`
