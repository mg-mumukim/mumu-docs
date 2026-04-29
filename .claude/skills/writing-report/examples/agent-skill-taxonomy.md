<!--
Taxonomy research report: shows the sources preamble, changes log, and Conclusion → Key Findings → Evidence hierarchy.
Read through the Conclusion section only to see the required opening structure and how much detail belongs there.
-->

# Agent Skill Taxonomy: Structure, Types, and Components

Sources:
- https://github.com/addyosmani/agent-skills (skills/, README.md, AGENTS.md, CONTRIBUTING.md, docs/skill-anatomy.md)
- https://github.com/vercel-labs/agent-skills (skills/, README.md, AGENTS.md)
- https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills (all skill directories)
- https://github.com/hyperconnect/hc-agent-registry/tree/main/skills/ailab (literature-survey, paper-summary)
- https://github.com/TinderBackend/tinder-ml-skill-registry/tree/main/skills (47 skills listed; 15 fully read)
- https://github.com/matchgroup-ai/mgai-skills-registry/tree/main/skills (70 unique skills across 9 categories; ~40 fully read)
- https://github.com/anthropics/skills (skills/, README.md; all 17 SKILL.md files read)
- https://github.com/openai/skills (skills/.curated/, skills/.system/; README.md; 44 unique skills read: 5 fully, remainder by content + name)
- note/idea/how-to-write-skill.md

---

## Changes from v4

- Added anthropics/skills (17 skills) and openai/skills (44 unique skills) as new sources
- All 61 new skills map to existing 11 types — no new types added; taxonomy validated across 8 vendor ecosystems
- Intra-skill component catalog extended from 35 to 44 components (components 36–44)
- Appendix C: new skill-to-type mapping tables and updated skill counts by type
- Frontmatter field inventory updated for `metadata.short-description` (openai)

---

## Conclusion

All skills share three universal structural elements. 11 distinct types cluster into 4 functional categories: Knowledge (2), Execution (4), Coordination (3), System (2). Three dimensions — corpus size, interaction model, persistence model — differentiate skill complexity. 44 reusable intra-skill components span 6 groups and recombine across types.

Repository counts: addyosmani (20 skills), vercel-labs (7), EveryInc (~35), hc (2), tinder-ml (~47), mgai (~70, ~40 unique to this repo), anthropics (17), openai (44 unique; openai-docs appears in both .curated and .system).

All 61 skills from the two new repos map to existing types. anthropics skills concentrate in Action/Automation (11) and Process-Enforcement (2), with single entries in Rule Library (2), Knowledge Connector (1), and Meta-Skill (1). openai skills concentrate in Action/Automation (26) and Knowledge Connector (7), with entries in Meta-Skill (3), Process-Enforcement (2), Diagnostic/Investigation (2), Multi-Source Aggregator (2), Living Knowledge Base (1), and Rule Library (1). Orchestrator, Multi-Model Consultation, and Onboarding/Tutorial have no instances in either new repo.

---

## Universal Structure

Every skill across all repos contains three components:

1. **Frontmatter** — `name` (kebab-case slug) and `description` (trigger-focused). Optional extensions: `metadata.author/version`, `license` (vercel-labs); `argument-hint` (EveryInc); `team` (hc); `allowed-tools` (mgai); `metadata.short-description` (openai).

2. **Activation conditions** — when the skill fires. Appears in the frontmatter `description`, a "When to Use / When NOT to Use" body section, or both.

3. **Process body** — a sequential or structured set of steps.

---

## Type Taxonomy

### A. Knowledge — encode domain rules for the agent to reference

These types are passive: the agent reads them to know what rules apply.

| Type | What it does | What distinguishes it |
|---|---|---|
| Rule Library | Encodes domain rules as a prioritized catalog | Rules stored in separate files ranked CRITICAL to LOW; SKILL.md is an index |
| Living Knowledge Base | Encodes all domain rules in one large file | Each rule shows wrong code first (annotated with failure mode), then correct code |

### B. Execution — single-agent task or investigation

These types execute a concrete task: automate an operation, connect to a service, investigate a failure, or enforce a practice.

| Type | What it does | What distinguishes it |
|---|---|---|
| Action/Automation | Automates a discrete operational task | State detection block, decision tree, environment-specific branching |
| Knowledge Connector | Wraps an authenticated internal service | Session state, API endpoint table, error code table, mandatory precondition |
| Diagnostic/Investigation | Systematic root cause analysis | Requires confirming failure reproduction before forming a hypothesis |
| Process-Enforcement | Encodes a multi-step practice with human approval gates | Anti-rationalization table, red flags list, verification checklist prevent agent shortcuts |

### C. Coordination — multi-agent or multi-source delegation

These types delegate work across multiple agents or data sources. The skill acts as coordinator; it does not do the work directly.

| Type | What it does | What distinguishes it |
|---|---|---|
| Orchestrator | Spawns and coordinates named subagents | Four sub-variants: static team, batch pipeline, template-instantiated, domain-specialized with lightweight validator |
| Multi-Source Aggregator | Fans out to N sources in parallel and synthesizes one output | Parallel Fan-Out, post-filter by secondary criterion, Fixed Output Template |
| Multi-Model Consultation | Wraps an external AI CLI for adversarial analysis | Multi-turn interrogation protocol; `allowed-tools` sandboxed to target CLI only |

### D. System — operate on the skill ecosystem

These types treat the skill system itself as the subject.

| Type | What it does | What distinguishes it |
|---|---|---|
| Onboarding/Tutorial | Progressive skill adoption with cross-session progress tracking | Persistent progress file, sub-skill orchestration, contextual authoring nudge |
| Meta-Skill | Creates, validates, and registers other skills | Guardrails checklist, progressive disclosure doctrine, degrees-of-freedom matching |

---

## Structural Dimensions

Three dimensions differentiate skill complexity across all types:

| Dimension | Values |
|---|---|
| Corpus size | Single-item / Bounded-set / Large-corpus |
| Interaction model | Autonomous / Interactive / Headless |
| Persistence model | Stateless / Session-stateful / Cross-session-stateful |

Two additional dimensions from mgai: `allowed-tools` restriction (declares which tool categories a skill may invoke) and bilingual documentation (parallel English/Korean reference files).

---

## Appendix A: Type Subcomponents

### Process-Enforcement

Core subcomponents:

1. Frontmatter: `name`, `description` (trigger-focused)
2. Overview: one-paragraph statement of purpose
3. When to Use / When NOT to Use: explicit include and exclude conditions
4. Gated workflow: named phases with explicit human-approval gates
5. Anti-rationalization table
6. Red flags list
7. Verification checklist
8. (Interactive variant) AskUserQuestion mandatory channel declaration
9. (Interactive variant) Incremental Draft Checkpoint

Key patterns:

- Components 5–7 exist to prevent agent tendency to shortcut process steps. None appear in Rule Library or Action/Automation skills.
- Interactive Process-Enforcement (tinder-ml workplan, brainstorming) declares AskUserQuestion as the mandatory channel for all user interaction. Embedding questions in prose risks users missing them.
- openai/security-threat-model: 8-phase gated workflow; user approval gate at phase 6 ("pause and wait for user feedback before producing the final report"); quality checklist at phase 8; "only load the reference files you need" lean reference management.
- anthropics/mcp-builder: 4-phase gated workflow; phase-specific dynamic URL fetching (component 37); evaluation creation as a formal phase.

---

### Rule Library

Core subcomponents:

1. Frontmatter: `name`, `description`, `license`, `metadata` (author, version)
2. Priority-ranked rule index (CRITICAL → LOW)
3. Per-rule files: `rules/<slug>.md`
4. Compiled document: `AGENTS.md`
5. Metadata sidecar: `metadata.json`

Key patterns:

- Two-level structure: SKILL.md is an index with priority-ranked slugs; detail lives in `rules/<slug>.md`. A compiled `AGENTS.md` and `metadata.json` sidecar complete the structure.

---

### Living Knowledge Base

Core subcomponents:

1. Frontmatter: `name`, `description` (trigger phrases embedded)
2. Domain sections (one `##` section per topic)
3. Problem statement per section
4. Correct/Incorrect Code Pair
5. Per-section Quick Checklist
6. (Some sections) Interactive Trigger

Key patterns:

- Inverts the Rule Library structure: all rules in a single large SKILL.md, organized by domain section.
- Wrong code is shown first with an annotation explaining the failure mode, then correct code.
- Interactive Trigger: a list of detection signals that, when found in context, cause the skill to ask for confirmation before applying the section's rules.
- Some variants include an `examples/` subdirectory (dispatch-router variant in tinder-ml).

---

### Action/Automation

Core subcomponents:

1. Frontmatter: `name`, `description`, `argument-hint` (EveryInc) / `allowed-tools` (mgai)
2. State detection block
3. Decision tree
4. Step sequences with inline shell code blocks
5. Environment-specific notes
6. Error handling / troubleshooting
7. Fallback paths

---

### Orchestrator

Core subcomponents:

1. Frontmatter: `name`, `description`
2. Phase structure
3. Sub-variant-specific elements (see below)

Sub-variants:

**Static team** (EveryInc ce-compound, ce-review): spawns a predefined team of named subagents. Subcomponents: mode declaration, subagent dispatch contracts, parallel/sequential blocks.

**Batch Pipeline** (hc literature-survey): loops over a large corpus (200+ items), delegates fixed-size batches (20 items) to subagents per iteration. Subcomponents: Batch Iterator, Accumulator, Snowball Search Expansion, Coverage Exit Condition, Rate Limit Backoff Registry.

**Template-Instantiated** (tinder-ml agent-team-templates): reads a YAML template file, interpolates variables, and spawns a team whose composition is determined at runtime by template and user answers. Subcomponents: template file location, variable interpolation, AskUserQuestion for parameters.

**Domain-Specialized with Lightweight Validator** (mgai weekly-report): creates a named team of agents with differentiated tool access profiles; after results are collected, delegates format validation to a haiku-class model. Retry loop capped at 3 iterations. Subcomponents: named team with per-agent tool access profiles, Lightweight Validator invocation, retry loop cap.

---

### Diagnostic/Investigation

Core subcomponents:

1. Frontmatter: `name`, `description`
2. Core principles stated at top
3. Phase execution table or numbered phases
4. Reproduction step (confirm failure before hypothesizing)
5. Hypothesis formation with causal chain gate
6. (With Devil's Advocate) Mandatory DA subagent, ACCEPT/WEAK/REJECT verdict handling
7. Structured close template
8. Anti-pattern table

---

### Knowledge Connector

Core subcomponents:

1. Frontmatter: `name`, `description`, optional `allowed-tools`
2. Config block (base URL, API version, state file paths); may use Config Auto-Discovery (Appendix B, component 30)
3. Authentication flow with session state
4. API endpoint reference table
5. Error code table
6. Golden Rule (mandatory precondition)
7. (Domain connectors) Domain-specific convention tables

Key patterns:

- Session state is persisted to a named file; re-authenticated via browser fallback when expired.
- tinder-ml examples: airflow-monitor, databricks-catalog, tinder-kubernetes-operation. mgai adds: databricks-notebook, databricks-sql, dbx-handler, fireflow-context.
- openai adds: figma (Figma MCP), figma-use (Figma Plugin API; mandatory prerequisite for other figma-* skills), figma-code-connect-components, figma-implement-design, chatgpt-apps (ChatGPT Apps SDK), linear, sentry.
- anthropics adds: claude-api (Claude API/SDK with endpoint tables and auth patterns).

---

### Multi-Source Aggregator

Core subcomponents:

1. Frontmatter: `name`, `description`
2. Input parsing (date/person/query parameters)
3. Parallel Fan-Out across sources
4. Timezone / secondary criterion handling
5. Post-Filter
6. Synthesis rules
7. Fixed Output Template

---

### Multi-Model Consultation

Core subcomponents:

1. Frontmatter: `name`, `description`, `allowed-tools` (restricts to target CLI commands)
2. Prerequisites block (CLI install check, auth status)
3. When to use (architecture decisions, debugging, code review, second opinion)
4. Prompt construction step (self-contained context; embed code snippets)
5. Invocation block (temp file naming via mktemp, CLI command with flags reference table)
6. Readiness / output handling (read response, clean up temp files)
7. (Multi-turn) Multi-Turn Round Framework with explicit context carry-forward (Appendix B, component 31)
8. Synthesis output (convergence / divergence / unique insight per model)

Key patterns:

- `allowed-tools` is used here for sandboxing, not capability declaration. This is the only type that uses `allowed-tools` this way.
- Each invocation is stateless from the external CLI's perspective. Multi-turn requires the calling skill to embed prior round conclusions explicitly in each subsequent prompt. Protocol: "Include the previous response's key conclusions — not the full text, but the specific claims you want to challenge."
- cmux-agents adds a Readiness Check: before sending the real prompt, the skill reads the terminal screen for agent-specific readiness signals (`›` for Codex, `❯` for Claude, `Type your message` for Gemini). Sending too early causes prompt concatenation with the launch command.
- Synthesis output requires three sections: (a) where all models agree, (b) where they disagree, (c) unique insight per model.
- mgai examples: codex-consultation (OpenAI Codex CLI), cursor-consultation (Cursor Agent CLI / Gemini), cmux-agents (Claude/Codex/Gemini via cmux terminal panes).

---

### Onboarding/Tutorial

Core subcomponents:

1. Frontmatter: `name`, `description`
2. Source material reference (source-of-truth playbook document)
3. Time check (AskUserQuestion: 5 min / 15 min / no rush)
4. New vs returning user branch (read persistent progress file at `~/.claude/memory/agentic-workflow-progress.md`)
5. (New) Environment probe opt-in → Progressive Environment Probe subagent → behavioral questions
6. (Returning) Outcome check on pending actions → level-up validation → fresh recommendations
7. Contextual Authoring Nudge (detect repeatable workflow signals)
8. Progress save (write/update persistent progress file)

Key patterns:

- Persistent progress file records: level (0–6), role tags, completed actions, pending actions, outcome log. Persists across sessions.
- Level system: 7 levels — Q&A / Workspace / Domain Memory / Tool Integration / Reusable Skills / Multi-Model Adversarial / Autonomous Loops.
- Sub-skill orchestration: the coordinator skill (agentic-workflow-tutorial) invokes /awt-assess, /awt-recommend, /awt-shop as named sub-skills.
- Progressive Environment Probe (component 32): a foreground blocking subagent on a lighter model scans the user's environment (CLAUDE.md, MCP configs, installed skills) and returns a structured report. The probe result determines which questions to skip in the main assessment.
- Contextual Authoring Nudge: detects "I do this every..." or "I always have to..." signals during the session and offers to scaffold a new SKILL.md.

---

### Meta-Skill

Core subcomponents:

1. Frontmatter: `name`, `description`
2. Three invocation modes (update existing / create new / auto-detect from session)
3. Guardrails Checklist (run before and after edits)
4. Progressive Disclosure Doctrine (3-tier model)
5. Degrees of Freedom matching table
6. Registry contribution workflow

Key patterns:

- Guardrails Checklist covers: frontmatter fields restricted to allowed set; `name` must be hyphen-case max 64 chars; `description` includes all trigger conditions max 1024 chars; SKILL.md under 500 lines; no extra documentation files.
- Progressive Disclosure Doctrine: metadata (~100 words, always in context), SKILL.md body (loaded when skill triggers), references/scripts (loaded on demand). Applied as acceptance criteria, not guidelines.
- Degrees of Freedom matching: high freedom (creative tasks) → text instructions; medium freedom → pseudocode; low freedom (brittle/critical) → scripts with few parameters.
- Registry contribution workflow: copy to registry, register in `marketplace.json` and `sources.json`, run `scripts/validate.sh`, commit to PR branch.
- mgai examples: update-skills, awt-shop, skill-router.
- anthropics/skill-creator adds: Eval-Driven Iteration Loop (component 41), Description Trigger Optimization (component 42), skill packaging via `scripts/package_skill.py`.
- openai/skill-creator adds: UI Metadata Sidecar in `agents/openai.yaml` (component 43), scaffold via `scripts/init_skill.py`, validation via `scripts/quick_validate.py`.
- openai/skill-installer: installs named skills from the openai/skills registry into `$CODEX_HOME/skills`; lists available curated/experimental skills via helper scripts; falls back from direct download to git sparse checkout on auth failure. Treats `.system` skills as pre-installed (skips install if asked). Uses Optional Skill Delegation pattern (component 44) indirectly: curated skills are available by name without installation while system skills are always present.

---

## Appendix B: Intra-Skill Component Catalog

44 reusable structural patterns found within skills, independent of type. They recombine across types and repos.

### Group 1: Control Flow

**1. Batch Iterator** — Process a fixed-size subset of a large work set per cycle; collect results; update state; advance to the next subset.

**2. Snowball Search Expansion** — Outputs from each processed item become new inputs to the work queue.

**3. Coverage Exit Condition** — The iteration loop exits when a coverage metric exceeds a named threshold, not when the queue empties.

**4. Parallel Fan-Out** — N tasks launched concurrently in a single step; all results collected before the parent proceeds.

**5. Sequential Gate** — Explicit block: the skill declares "do not advance to Phase X until Phase Y is complete."

---

### Group 2: State Management

**6. Accumulator** — A running aggregate updated after each batch cycle; influences routing or prioritization in later cycles.

**7. Session State Cache** — Auth tokens or session cookies persisted to a named file; re-authenticated via browser fallback when expired.

**8. Locked Plan Artifact** — An intermediate document written to disk and marked immutable after a named trigger event.

**9. Rate Limit Backoff Registry** — A per-service file tracking the last successful retry delay; exponential backoff with persistent state.

**30. Config Auto-Discovery** — A `.config.json` file in the skill's own directory, populated by a one-time discovery script that reads environment sources (SSH config, AWS profiles, settings.json). Read on all subsequent invocations with `jq -r`. Fields that cannot be discovered are asked from the user once and saved.
- hyperpod-slurm: discovers `cluster_host` from `~/.ssh/config`; weekly-report: discovers `github_username` from `gh api user`; s3-cross-sync: discovers AWS account IDs per environment from `aws sts get-caller-identity`
- scaffold-manifest-builder: discovers `aws_account_ecr` from Dockerfile grep

**38. Persistent REPL Handle Management** — Declare top-level tool handles using `var` (not `let`/`const`) in a persistent JavaScript REPL session so bindings survive across code cells. Reuse existing handles with a stale-check-and-reset pattern (set binding to `undefined`, rerun the launch cell) rather than full REPL restart. Bootstrap cell runs once per session; subsequent cells reference the shared handles.
- playwright-interactive: `var browser`, `var page`, `var electronApp`; `ensureWebBrowser()` checks `browser.isConnected()` before reuse; `js_repl_reset` is recovery-only, not routine cleanup

---

### Group 3: Data Routing

**10. Fixed Output Template** — A required output schema with named sections and fields specified in the skill body.

**11. Index-Based Routing** — Read a small index file first; use the index to determine which specific large files to load.

**12. Decision Table Lookup** — A table mapping input conditions to prescribed actions.

**13. Quantification Gate** — A named action is blocked until a numeric estimate is produced.

**14. Post-Filter** — Results from a broad query are filtered by a secondary criterion after collection.

**37. Dynamic URL Documentation Fetching** — The skill embeds live external URL references with phase-specific or on-demand loading instructions. The agent fetches documentation via WebFetch at the point of need rather than loading everything upfront. Each URL entry specifies what to fetch and during which phase. Prevents context pollution from large documentation files that do not need to be read on every invocation.
- mcp-builder: "Load MCP Best Practices: reference/mcp_best_practices.md. For TypeScript SDK (during Phase 2): fetch from raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md"; phase-gated URL schedule embedded in SKILL.md body

---

### Group 4: Interaction Patterns

**15. Mandatory Subagent Invocation** — A specific subagent MUST be called at a named phase and cannot be skipped.

**16. Devil's Advocate Challenge Loop** — A dedicated adversarial subagent returns a verdict (ACCEPT/WEAK/REJECT); the parent must address each challenge before proceeding.

**17. One-Question-at-a-Time Dialogue** — AskUserQuestion is declared the only channel for user input; one question per message.

**18. Incremental Draft Checkpoint** — After 3–4 clarification rounds, the skill shows a partial draft and asks for explicit confirmation before proceeding.

**19. A/B Verification Procedure** — Same-system before measurement → apply exactly one change → after measurement; specifies what to record at each step.

**20. Interactive Trigger on Condition** — A skill section activates only when a named detectable signal is found in context; asks for confirmation before applying.

**31. Multi-Turn Round Framework** — Structured N-round interrogation protocol for stateless external AI CLIs. Each round applies a different analytical lens; context from the previous round is explicitly carried forward in the next prompt. Standard sequence: recommendation → inversion ("what guarantees failure?") → pre-mortem → generalization → synthesis.
- codex-consultation and cursor-consultation define 5+ rounds; round files named with mktemp session prefix (`/tmp/codex-session-XXXXXX-rN-prompt.txt`)

**32. Progressive Environment Probe** — A foreground blocking subagent dispatched on a lighter model to scan the user's local environment (config files, installed tools, directory structure). Returns a structured report; the probe's full file listings stay in the subagent context. Parent uses the report to skip questions or tailor the main flow.
- awt-assess: probe runs on `sonnet` model; checks CLAUDE.md, MCP servers, installed skills, memory files, hooks; returns `ENV_PROBE_REPORT` with `INFERRED_LEVEL` and `KEY_FINDING`

**33. Multi-Agent Team Delegation with Lightweight Validator** — The parent creates a named team of N agents with differentiated tool access profiles. After all agents complete, a lightweight validator model (haiku-class) checks the output against format criteria. On failure, the parent corrects and re-validates; loop capped at a fixed number of retries.
- weekly-report: 6 agents (github-researcher, glean-slack-researcher, glean-docs-researcher, atlassian-researcher, glean-chat-researcher, format-reviewer); haiku validates format; 3-retry cap

**40. QA Coverage Inventory** — Build a shared checklist before testing starts from three sources: stated requirements, features actually implemented, and claims the agent plans to make in the response. All items from any of those three sources must map to at least one QA check. The inventory is updated during exploratory testing when new states or controls are discovered and serves as the unified coverage contract for both functional and visual QA passes.
- playwright-interactive: "Write a brief QA inventory before testing... Anything that appears in any of those three sources must map to at least one QA check before signoff"; inventory updated during exploratory pass; coverage is explicit in signoff note

**44. Optional Skill Delegation** — At a named decision point, the skill checks whether another named skill is available in the agent's environment. If available, delegates the subtask to that skill. If not, executes an inline fallback equivalent. Preserves modularity without a hard dependency.
- gh-fix-ci: "If a plan-oriented skill (for example `create-plan`) is available, use it; otherwise draft a concise plan inline and request approval before implementing."

---

### Group 5: Quality Enforcement

**21. Quality Checklist** — A named set of yes/no criteria run in sequence before the skill output is declared complete.

**22. Anti-Pattern Table** — A two-column table of [tempting pattern → why it's wrong]. The tempting pattern is named first.

**23. Correct/Incorrect Code Pair** — Each rule illustrated by wrong code first (with inline failure mode annotation), then correct code.

**24. Priority Tier Classification** — Items classified into a small named set of tiers at collection time; tier determines processing order and output depth.

**25. Graceful Degradation with Source Note** — If a data source is unavailable, skip it, record its absence in a named output field, and continue.

**34. Output Style Enforcement** — A dedicated section specifying writing style with DO/DON'T annotated examples, a situational decision table (when to use each style), and an anti-pattern list.
- daily-work-summary: DO examples show multi-sentence narrative with inline links; DON'T examples show terse bullet lists; decision table maps situation to prescribed style; named labels (왜?, 에러 메시지:, 원인:, 삽질 포인트:)

**35. Platform Exclusive Path Declaration** — An explicit negation of alternative execution paths that might appear valid but are not. Placed at the top of the skill body as a bold warning.
- cpts-deploy: "CPTS services do NOT use Jenkins. Always use this skill instead of searching for Jenkins jobs." Gotchas section adds: "Glean이 Jenkins 배포 방법을 추천하면 무시할 것"

**41. Eval-Driven Iteration Loop** — Run test prompts via paired subagents (with-skill and baseline), grade each output against a set of assertions, display results in a structured viewer for human review, read explicit feedback, and iterate until user satisfaction or diminishing returns. Baseline choice: no-skill run for new skills; previous-version run for improvements.
- anthropics/skill-creator: spawn with-skill + baseline runs in same turn; grade with `agents/grader.md`; aggregate benchmark via `scripts/aggregate_benchmark.py`; launch `eval-viewer/generate_review.py`; read `feedback.json`; repeat into `iteration-N+1/`

**42. Description Trigger Optimization** — Automated loop that generates should-trigger and should-not-trigger eval queries, evaluates current skill description triggering accuracy (each query run N times for reliability), proposes improved descriptions via LLM, iterates up to a fixed cap, and selects the best description by held-out test score to avoid overfitting.
- anthropics/skill-creator: `scripts/run_loop.py --eval-set ... --max-iterations 5`; 60/40 train/test split; winner selected by test score; result is `best_description` JSON field applied to SKILL.md frontmatter

---

### Group 6: Composition Patterns

**26. Skill Context Assignment** — Each spawned agent role receives a named list of skills injected into its spawn prompt.

**27. Dispatch Router** — The skill maps the incoming request type to a named sub-document and delegates.

**28. Pre-Execution Warm-Up** — A precondition that every invocation must complete before main logic begins; repeats on every call regardless of the request.

**29. Keyword-Based Auto-Link** — A corpus scan for keyword matches against a named index; matching items are automatically linked without manual intervention.

**36. Black-Box Script Invocation** — Bundled helper scripts that the agent must NOT read into context. The agent runs `--help` to learn parameters, then invokes the script directly. Reading the source is explicitly prohibited unless a customized solution is absolutely necessary; the stated reason is context-window pollution.
- webapp-testing: "DO NOT read the source until you try running the script first and find that a customized solution is absolutely necessary. These scripts can be very large and thus pollute your context window. They exist to be called directly as black-box scripts rather than ingested into your context window."
- gh-fix-ci: bundled `scripts/inspect_pr_checks.py` called with `--help` first, then invoked directly

**39. Pre-Flight Capability Check** — An explicit prerequisite that checks for a non-standard tool capability or feature flag before execution begins. Provides exact configuration commands to enable the capability if missing; may require restarting the agent session after enabling. Placed before the main workflow and blocks execution until the capability is confirmed.
- playwright-interactive: checks `js_repl` feature flag in `~/.codex/config.toml`; "If `js_repl` is missing, enable it... After enabling `js_repl`, start a new Codex session so the tool list refreshes"; also requires `--sandbox danger-full-access` mode

**43. UI Metadata Sidecar** — A separate `agents/<platform>.yaml` file containing UI-facing display metadata (display_name, short_description, default_prompt, optional brand color/icon) for skill marketplace listing. Generated and regenerated via a dedicated script; kept in sync with SKILL.md content. Distinct from SKILL.md frontmatter (which drives triggering); serves the marketplace discovery UI only.
- openai/skill-creator: `agents/openai.yaml`; generated via `scripts/generate_openai_yaml.py --interface key=value`; "On updates: validate `agents/openai.yaml` still matches SKILL.md; regenerate if stale"

---

## Appendix C: Evidence

### Skill-to-type mapping

**addyosmani/agent-skills** — all Process-Enforcement or Diagnostic (20 skills)

**vercel-labs/agent-skills** — Rule Library (5) and Action/Automation (2)

**EveryInc/compound-engineering-plugin** — ~35 skills spanning Process-Enforcement, Orchestrator, Diagnostic, Action/Automation, Rule Library

**hyperconnect/hc-agent-registry (ailab)** — 2 skills: literature-survey (Orchestrator Batch Pipeline), paper-summary (Action/Automation)

**TinderBackend/tinder-ml-skill-registry** — 47 skills spanning all seven types from v2 (see v2 for full table)

**matchgroup-ai/mgai-skills-registry** (40 fully read; 30 classified by name) — see v4 for full table

**anthropics/skills** (17 skills, all SKILL.md read)

| Skill | Type | Corpus size | Interaction | Persistence |
|---|---|---|---|---|
| algorithmic-art | Action/Automation | Single-item | Autonomous | Stateless |
| brand-guidelines | Rule Library | Bounded-set | Autonomous | Stateless |
| canvas-design | Action/Automation | Single-item | Autonomous | Stateless |
| claude-api | Knowledge Connector | Bounded-set | Autonomous | Stateless |
| doc-coauthoring | Process-Enforcement | Bounded-set | Interactive | Stateless |
| docx | Action/Automation | Single-item | Autonomous | Stateless |
| frontend-design | Action/Automation | Single-item | Autonomous | Stateless |
| internal-comms | Rule Library | Bounded-set | Autonomous | Stateless |
| mcp-builder | Process-Enforcement | Bounded-set | Autonomous | Stateless |
| pdf | Action/Automation | Single-item | Autonomous | Stateless |
| pptx | Action/Automation | Single-item | Autonomous | Stateless |
| skill-creator | Meta-Skill | Bounded-set | Interactive | Session-stateful |
| slack-gif-creator | Action/Automation | Single-item | Autonomous | Stateless |
| theme-factory | Action/Automation | Single-item | Autonomous | Stateless |
| web-artifacts-builder | Action/Automation | Single-item | Autonomous | Stateless |
| webapp-testing | Action/Automation | Bounded-set | Autonomous | Stateless |
| xlsx | Action/Automation | Single-item | Autonomous | Stateless |

**openai/skills** (44 unique skills; 5 fully read; remainder classified by SKILL.md content summary and name)

Note: openai-docs appears in both `.curated` and `.system` with identical content (same SHA); counted once.

| Skill | Location | Type | Notes |
|---|---|---|---|
| aspnet-core | .curated | Action/Automation | |
| chatgpt-apps | .curated | Knowledge Connector | ChatGPT Apps SDK integration |
| cli-creator | .curated | Action/Automation | |
| cloudflare-deploy | .curated | Action/Automation | |
| develop-web-game | .curated | Action/Automation | |
| doc | .curated | Action/Automation | |
| figma-code-connect-components | .curated | Knowledge Connector | Figma Code Connect API |
| figma-create-design-system-rules | .curated | Action/Automation | |
| figma-create-new-file | .curated | Action/Automation | |
| figma-generate-design | .curated | Action/Automation | |
| figma-generate-library | .curated | Action/Automation | |
| figma-implement-design | .curated | Knowledge Connector | design-to-code translation |
| figma-use | .curated | Knowledge Connector | Figma Plugin API; mandatory prerequisite for other figma-* skills |
| figma | .curated | Knowledge Connector | Figma MCP integration |
| frontend-skill | .curated | Action/Automation | |
| gh-address-comments | .curated | Action/Automation | |
| gh-fix-ci | .curated | Diagnostic/Investigation | bundled inspect_pr_checks.py script |
| jupyter-notebook | .curated | Action/Automation | |
| linear | .curated | Knowledge Connector | Linear API |
| netlify-deploy | .curated | Action/Automation | |
| notion-knowledge-capture | .curated | Action/Automation | |
| notion-meeting-intelligence | .curated | Multi-Source Aggregator | |
| notion-research-documentation | .curated | Multi-Source Aggregator | |
| notion-spec-to-implementation | .curated | Process-Enforcement | |
| openai-docs | .curated / .system | Living Knowledge Base | identical content in both locations |
| pdf | .curated | Action/Automation | |
| playwright-interactive | .curated | Action/Automation | js_repl persistent session; Pre-Flight Capability Check; QA Coverage Inventory |
| playwright | .curated | Action/Automation | |
| render-deploy | .curated | Action/Automation | |
| screenshot | .curated | Action/Automation | |
| security-best-practices | .curated | Rule Library | |
| security-ownership-map | .curated | Diagnostic/Investigation | |
| security-threat-model | .curated | Process-Enforcement | 8-phase gated workflow; user approval gate at phase 6 |
| sentry | .curated | Knowledge Connector | Sentry error monitoring API |
| sora | .curated | Action/Automation | |
| speech | .curated | Action/Automation | |
| transcribe | .curated | Action/Automation | |
| vercel-deploy | .curated | Action/Automation | |
| winui-app | .curated | Action/Automation | |
| yeet | .curated | Action/Automation | git stage/commit/push/PR in one flow |
| imagegen | .system | Action/Automation | auto-installed |
| plugin-creator | .system | Meta-Skill | auto-installed |
| skill-creator | .system | Meta-Skill | auto-installed; UI Metadata Sidecar; init/validate scripts |
| skill-installer | .system | Meta-Skill | installs curated/experimental skills; listing + download scripts |

---

### Skill counts by type

| Type | addyosmani | vercel-labs | EveryInc | hc | tinder-ml | mgai | anthropics | openai | Total |
|---|---|---|---|---|---|---|---|---|---|
| Process-Enforcement | 19 | 0 | ~5 | 0 | ~12 | ~10 | 2 | 2 | ~50 |
| Rule Library | 0 | 5 | ~3 | 0 | ~3 | ~3 | 2 | 1 | ~17 |
| Living Knowledge Base | 0 | 0 | 0 | 0 | ~3 | ~3 | 0 | 1 | ~7 |
| Action/Automation | 0 | 2 | ~17 | 1 | ~16 | ~22 | 11 | 26 | ~95 |
| Orchestrator | 0 | 0 | ~3 | 1 | ~3 | ~2 | 0 | 0 | ~9 |
| Diagnostic/Investigation | 1 | 0 | ~4 | 0 | ~4 | ~5 | 0 | 2 | ~16 |
| Knowledge Connector | 0 | 0 | 0 | 0 | ~6 | ~8 | 1 | 7 | ~22 |
| Multi-Source Aggregator | 0 | 0 | 0 | 0 | ~3 | ~3 | 0 | 2 | ~8 |
| Multi-Model Consultation | 0 | 0 | 0 | 0 | 0 | 3 | 0 | 0 | 3 |
| Onboarding/Tutorial | 0 | 0 | 0 | 0 | 0 | 3 | 0 | 0 | 3 |
| Meta-Skill | 0 | 0 | 0 | 0 | 0 | 3 | 1 | 4 | 8 |

openai counts are based on full reads of 5 skills and content-based classification of the remainder. openai-docs counted once.

---

### Supplementary file patterns

| Type | File layout outside SKILL.md |
|---|---|
| Process-Enforcement (addyosmani) | `references/*.md` (shared checklists loaded on-demand) |
| Rule Library (vercel-labs) | `rules/*.md`, `AGENTS.md` (compiled), `metadata.json`, `resources/` (scripts) |
| Living Knowledge Base (tinder-ml) | No supplementary files — all content in SKILL.md; `examples/` subdirectory in dispatch-router variant |
| Action/Automation (vercel-labs) | `resources/` (shell scripts) |
| Action/Automation (EveryInc) | `references/` (optional) |
| Orchestrator (EveryInc) | `references/` (YAML schemas, templates), `assets/` (document templates) |
| Orchestrator (hc, Batch Pipeline) | `references/` (search-sources.md, templates.md) |
| Orchestrator (mgai weekly-report) | `references/agents.md` (per-agent spawn prompts), `assets/format-example.md` |
| Diagnostic (EveryInc, tinder-ml) | `references/` (anti-patterns, investigation techniques) |
| Knowledge Connector (tinder-ml) | No supplementary files — all in SKILL.md |
| Multi-Model Consultation (mgai) | No supplementary files — all in SKILL.md; `assets/` for hook scripts in aws-sso-auto-login |
| Onboarding/Tutorial (mgai) | `AI-AGENT-PLAYBOOK.md` (source-of-truth document referenced by tutorial skill) |
| Meta-Skill (mgai update-skills) | `references/authoring-guidelines.md` |
| Action/Automation with scripts (mgai scaffold-manifest-builder) | `scripts/` (Python and shell scripts), `references/` (questionnaire, schema, rules), `assets/templates/` |
| Platform Reference (mgai hyperpod-slurm) | `references/job-workflows.en.md`, `references/job-workflows.ko.md`, `references/platform-operations.en.md`, `references/platform-operations.ko.md` |
| Action/Automation (anthropics webapp-testing) | `scripts/with_server.py` (Black-Box Script Invocation); `examples/` (pattern scripts) |
| Process-Enforcement (anthropics mcp-builder) | `reference/mcp_best_practices.md`, `reference/node_mcp_server.md`, `reference/python_mcp_server.md`, `reference/evaluation.md` (Dynamic URL Documentation Fetching) |
| Meta-Skill (anthropics skill-creator) | `agents/grader.md`, `agents/comparator.md`, `agents/analyzer.md`, `scripts/aggregate_benchmark.py`, `scripts/run_loop.py`, `eval-viewer/generate_review.py`, `assets/eval_review.html`, `references/schemas.md` |
| Action/Automation (openai playwright-interactive) | No supplementary files — all in SKILL.md; js_repl session maintained in-process |
| Diagnostic (openai gh-fix-ci) | `scripts/inspect_pr_checks.py` (Black-Box Script Invocation) |
| Process-Enforcement (openai security-threat-model) | `references/prompt-template.md`, `references/security-controls-and-assets.md` |
| Meta-Skill (openai skill-creator) | `agents/openai.yaml` (UI Metadata Sidecar), `scripts/init_skill.py`, `scripts/quick_validate.py`, `scripts/generate_openai_yaml.py`, `references/openai_yaml.md`, `references/schemas.md` |
| Meta-Skill (openai skill-installer) | `scripts/list-skills.py`, `scripts/install-skill-from-github.py` |

---

### Frontmatter field inventory

| Field | addyosmani | vercel-labs | EveryInc | hc | tinder-ml | mgai | anthropics | openai |
|---|---|---|---|---|---|---|---|---|
| name | yes | yes | yes | yes | yes | yes | yes | yes |
| description | yes | yes | yes | yes | yes | yes | yes | yes |
| metadata.author | no | yes | no | no | no | no | no | no |
| metadata.version | no | yes | no | no | no | no | no | no |
| metadata.short-description | no | no | no | no | no | no | no | yes |
| license | no | yes | no | no | no | no | yes | no |
| argument-hint | no | no | yes | no | no | no | no | no |
| team | no | no | no | yes | no | no | no | no |
| allowed-tools | no | no | no | no | no | yes | no | no |

anthropics uses `license` pointing to `LICENSE.txt`; openai uses a standalone `LICENSE.txt` file (not frontmatter).

---

### Intra-Skill Component cross-reference by type

| Component | Process-E | Rule Lib | Living KB | Action | Orchestrator | Diagnostic | K. Connector | Aggregator | Multi-Model | Onboarding | Meta |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Batch Iterator | | | | | X | | | | | | |
| Snowball Expansion | | | | | X | | | | | | |
| Coverage Exit | | | | | X | | | | | | |
| Parallel Fan-Out | | | | | X | | | X | | | |
| Sequential Gate | X | | | | X | | | | | | |
| Accumulator | | | | | X | | | X | | | |
| Session State Cache | | | | | | | X | | | | |
| Locked Plan Artifact | X | | | | X | | | | | | |
| Rate Limit Registry | | | | | X | | | | | | |
| Config Auto-Discovery | | | | X | X | | X | X | | | |
| Fixed Output Template | X | | | X | X | X | | X | | | |
| Index-Based Routing | | X | | | | | | X | | | |
| Decision Table Lookup | | | X | X | | X | X | | | | |
| Quantification Gate | | | | | | X | | | | | |
| Post-Filter | | | | | | | | X | | | |
| Correct/Incorrect Code Pair | | X | X | | | | | | | | |
| Mandatory Subagent | | | | | X | X | | | | | |
| Devil's Advocate Loop | X | | | | | X | | | | | |
| One-Question Dialogue | X | | | | X | | | | | X | |
| Incremental Draft | X | | | | | | | | | | |
| A/B Verification | | | | | | X | | | | | |
| Interactive Trigger | | | X | | | | X | | | | |
| Quality Checklist | X | | X | | | X | | | | | X |
| Anti-Pattern Table | | | X | | | X | | | | | |
| Priority Tier | | X | | | X | | | | | | |
| Graceful Degradation | | | | | | | | X | | | |
| Skill Context Assignment | | | | | X | | | | | | |
| Dispatch Router | | | | X | | | | X | | | |
| Pre-Execution Warm-Up | | | X | | | | X | X | | | |
| Keyword Auto-Link | | | | X | | | | | | | |
| Multi-Turn Round Framework | | | | | | | | | X | | |
| Progressive Environment Probe | | | | | | | | | | X | |
| Multi-Agent Team + Validator | | | | | X | | | | | | |
| Output Style Enforcement | | | | | | | | X | | | |
| Platform Exclusive Path | | | | X | | X | | | | | |
| Black-Box Script Invocation | | | | X | | X | | | | | |
| Dynamic URL Doc Fetching | X | | | | | | | | | | |
| Persistent REPL Handle Mgmt | | | | X | | | | | | | |
| Pre-Flight Capability Check | | | | X | | | | | | | |
| QA Coverage Inventory | | | | X | | | | | | | |
| Eval-Driven Iteration Loop | | | | | | | | | | | X |
| Description Trigger Optim. | | | | | | | | | | | X |
| UI Metadata Sidecar | | | | | | | | | | | X |
| Optional Skill Delegation | | | | | | X | | | | | |
