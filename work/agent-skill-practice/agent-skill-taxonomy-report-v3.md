# Agent Skill Taxonomy: Common Dimensions, Types, Core Subcomponents, and Intra-Skill Components

Sources:
- https://github.com/addyosmani/agent-skills (skills/, README.md, AGENTS.md, CONTRIBUTING.md, docs/skill-anatomy.md)
- https://github.com/vercel-labs/agent-skills (skills/, README.md, AGENTS.md)
- https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills (all skill directories)
- https://github.com/hyperconnect/hc-agent-registry/tree/main/skills/ailab (literature-survey, paper-summary)
- https://github.com/TinderBackend/tinder-ml-skill-registry/tree/main/skills (47 skills listed; 15 fully read)
- https://github.com/matchgroup-ai/mgai-skills-registry/tree/main/skills (70 unique skills across 9 categories; ~40 fully read)
- note/idea/how-to-write-skill.md

---

## Changes from v2

- Added findings from mgai-skills-registry (70 unique skills across 9 categories; ~40 fully read)
- Added Finding 11: Multi-Model Consultation — a new skill type wrapping external AI CLIs
- Added Finding 12: Onboarding/Tutorial type with persistent progress state and sub-skill orchestration
- Added Finding 13: Meta-Skill type for skill authoring and registry management
- Added Finding 14: Config Auto-Discovery as a new structural pattern present in 5+ mgai skills
- Added Finding 15: Six new intra-skill components (components 30–35)
- Updated frontmatter field inventory with `allowed-tools` field
- Updated skill-to-type mapping and skill counts for mgai
- Updated Intra-Skill Component cross-reference table

---

## Conclusion

Skills across all six repositories share three universal structural dimensions: a frontmatter identity block, explicit activation conditions, and a structured process body. Beyond these universals, ten distinct skill types emerge: **Process-Enforcement**, **Rule Library**, **Living Knowledge Base**, **Action/Automation**, **Orchestrator**, **Diagnostic/Investigation**, **Knowledge Connector**, **Multi-Source Aggregator**, **Multi-Model Consultation**, **Onboarding/Tutorial**, and **Meta-Skill**. Three structural dimensions from v2 differentiate skill complexity: corpus size, interaction model, and persistence model. Two new dimensions appear in mgai: `allowed-tools` restriction (sandboxes which tools a skill may call) and bilingual documentation (parallel English/Korean reference files).

Beyond skill-level taxonomy, 35 reusable intra-skill components can be identified. Six new components (30–35) come from mgai: Config Auto-Discovery, Multi-Turn Round Framework, Multi-Agent Team Delegation with Lightweight Validator, Output Style Enforcement, Platform Exclusive Path Declaration, and Progressive Environment Probe.

Critical numbers: addyosmani contributes 20 skills (Process-Enforcement or Diagnostic); vercel-labs contributes 7 (Rule Library and Action/Automation); EveryInc contributes ~35 (spanning all types); hc-agent-registry/ailab contributes 2 (Orchestrator and Action/Automation); tinder-ml-skill-registry contributes ~47 spanning seven types; mgai-skills-registry contributes ~70 unique skills spanning all ten types, with ~40 new skills not present in any prior repo.

---

## Key Findings

### Finding 1: Three dimensions are universal across all skills and repos

Every skill examined, regardless of repo or type, contains:

1. **Frontmatter identity block** — a `name` field (kebab-case slug) and a `description` field (trigger-focused, beginning with "Use when…"). vercel-labs adds `metadata.author`, `metadata.version`, and `license`. EveryInc adds `argument-hint`. hc-agent-registry adds `team`. tinder-ml extends `description` with inline trigger-phrase lists. mgai adds `allowed-tools` to restrict which tool categories the skill may call.

2. **Activation conditions** — where and how the skill fires. Appears in the frontmatter `description`, a dedicated "When to Use / When NOT to Use" body section, or both. addyosmani always includes a dedicated body section. vercel-labs puts trigger phrases in the description. EveryInc uses both description and an `<auto_invoke>` block. tinder-ml embeds trigger phrase lists directly in the description field.

3. **Process body** — a sequential or structured set of steps. The depth and complexity vary greatly by type, but every skill has at minimum a numbered or phased sequence.

### Finding 2: Ten types can be identified by operational mode

The following taxonomy spans all six repos (see Evidence section for skill-to-type mapping):

| Type | Operational mode | Primary repo |
|---|---|---|
| Process-Enforcement | Encodes a multi-step engineering practice with quality gates | addyosmani (all 20), tinder-ml, mgai |
| Rule Library | Encodes a prioritized catalog of domain rules with code examples | vercel-labs (5 skills) |
| Living Knowledge Base | Single large file encoding domain knowledge as correct/incorrect code pairs, organized by domain section | tinder-ml, mgai |
| Action/Automation | Automates a discrete operational task with environment-aware branching | vercel-labs, EveryInc, tinder-ml, mgai |
| Orchestrator | Coordinates parallel subagents or batch pipelines across phased workflows | EveryInc, tinder-ml, hc, mgai |
| Diagnostic/Investigation | Systematic root cause analysis through structured hypothesis testing | addyosmani, EveryInc, tinder-ml, mgai |
| Knowledge Connector | Wraps an authenticated internal service with session state, API reference, and error table | tinder-ml, mgai |
| Multi-Source Aggregator | Fans out to N parallel sources, post-filters by secondary criterion, synthesizes unified output | tinder-ml, mgai |
| Multi-Model Consultation | Wraps an external AI CLI and sends structured prompts for adversarial or supplementary analysis | mgai |
| Onboarding/Tutorial | Progressive skill adoption coach with persistent user progress tracking across sessions | mgai |
| Meta-Skill | Creates, validates, and registers other skills; enforces structural guardrails | mgai |

### Finding 3: The Process-Enforcement type is defined by three exclusive subcomponents

(Unchanged from v1.) The addyosmani format explicitly names its anatomy in docs/skill-anatomy.md. Process-Enforcement skills uniquely include: (1) an anti-rationalization table, (2) a red flags list, and (3) a verification checklist. None appear in Rule Library or Action/Automation skills. They exist to combat agent tendency to shortcut process steps.

tinder-ml adds a fourth variant: **Interactive Process-Enforcement**, where AskUserQuestion is the mandatory channel for all user interaction. workplan and brainstorming declare "AskUserQuestion is MANDATORY — never ask questions via plain text." This prevents the agent from embedding questions inside prose responses that users may miss.

### Finding 4: The Rule Library type externalizes content into individual rule files; the Living Knowledge Base internalizes all rules into one file with code pairs

vercel-labs Rule Library uses a two-level structure: SKILL.md is an index with priority-ranked slugs; detail lives in individual `rules/<slug>.md` files. A compiled `AGENTS.md` and `metadata.json` sidecar complete the structure.

tinder-ml's Living Knowledge Base type (scaffold-context, kaizen) inverts this: all rules live in a single large SKILL.md, organized by domain section. Each rule follows a **Correct/Incorrect Code Pair** pattern — the wrong code is shown first with an annotation explaining the failure mode, then the correct code. Some sections include an **Interactive Trigger**: a list of detection signals that, when found in context, cause the skill to ask the user for confirmation before applying the section's rules.

### Finding 5: The Orchestrator type now has three distinct sub-variants

(Extending v1.) EveryInc Orchestrators (ce-compound, ce-review) spawn a **static, predefined team** of named subagents.

hc-agent-registry literature-survey introduces a **Batch Pipeline Orchestrator**: the parent loops over a large corpus (200+ papers), delegating 20-item batches to subagents per iteration.

tinder-ml agent-team-templates introduces a **Template-Instantiated Orchestrator**: reads a YAML template file, interpolates variables, and spawns a team whose composition is determined at runtime by the template and user answers.

mgai weekly-report introduces a **Domain-Specialized Orchestrator with Lightweight Validator**: the parent creates a named team of 6 agents each assigned different tool access profiles (github-researcher, glean-slack-researcher, glean-docs-researcher, atlassian-researcher, glean-chat-researcher, format-reviewer), collects results, then delegates final format validation to a haiku-class model rather than the primary model. The validator runs a retry loop (max 3 iterations) before finalization.

### Finding 6: The Knowledge Connector type introduces session state and API reference tables

(Unchanged from v2.) tinder-ml contributes the Knowledge Connector type: airflow-monitor, databricks-catalog, tinder-kubernetes-operation. Structural markers: session state file, re-authentication fallback, API endpoint reference table, error code table, Golden Rule. mgai extends this type with additional Knowledge Connector skills in ml-ops/ (databricks-notebook, databricks-sql, dbx-handler, fireflow-context).

### Finding 7: Supplementary files are structured by type

(Updated from v2 with mgai data.)

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
| Onboarding/Tutorial (mgai) | `AI-AGENT-PLAYBOOK.md` (source of truth document referenced by tutorial skill) |
| Meta-Skill (mgai update-skills) | `references/authoring-guidelines.md` |
| Action/Automation with scripts (mgai scaffold-manifest-builder) | `scripts/` (Python and shell scripts), `references/` (questionnaire, schema, rules), `assets/templates/` |
| Platform Reference (mgai hyperpod-slurm) | `references/job-workflows.en.md`, `references/job-workflows.ko.md`, `references/platform-operations.en.md`, `references/platform-operations.ko.md` |

### Finding 8: Two additional skill types emerge from tinder-ml and hc

(Unchanged from v2.) Knowledge Connector and Multi-Source Aggregator. See v2 for full descriptions.

### Finding 9: Three structural dimensions differentiate skill complexity

(Unchanged from v2.) Corpus size (Single-item / Bounded-set / Large-corpus), interaction model (Autonomous / Interactive / Headless), persistence model (Stateless / Session-stateful / Cross-session-stateful).

### Finding 10: Living Knowledge Base type extends Rule Library with code pairs and interactive triggers

(Unchanged from v2.)

### Finding 11: Multi-Model Consultation type wraps external AI CLIs with structured interrogation protocols

mgai introduces a skill type not present in any prior repo: the Multi-Model Consultation. Examples: codex-consultation (OpenAI Codex CLI), cursor-consultation (Cursor Agent CLI / Gemini), cmux-agents (Claude/Codex/Gemini via cmux terminal panes).

Structural markers:

- **`allowed-tools` restriction**: frontmatter declares `allowed-tools: Bash(codex *), Read, Write` (or equivalent) to constrain which tools the skill can invoke. This is the only type that uses `allowed-tools` for sandboxing rather than capability declaration.
- **Stateless external agent wrapper**: each invocation is ephemeral. The external CLI has no memory of previous calls.
- **Context carry-forward protocol**: multi-turn discussions require the calling skill to embed prior round conclusions in each subsequent prompt. The protocol is explicit: "Include the previous response's key conclusions — not the full text, but the specific claims you want to challenge."
- **Multi-Turn Round Framework**: structured 5-round interrogation — Round 1: initial recommendation; Round 2: inversion ("what would guarantee this fails?"); Round 3: pre-mortem (stress-test weak points); Round 4: generalization; Round 5+: synthesis and reconciliation.
- **Synthesis output**: final response requires three sections — (a) where all models agree (high confidence), (b) where they disagree (needs human judgment), (c) what unique insight each model brings.
- **Readiness check** (cmux-agents): before sending the real prompt, the skill reads the terminal screen and looks for agent-specific readiness signals (`›` for Codex, `❯` for Claude, `Type your message` for Gemini). Sending too early causes prompt concatenation with the launch command.

### Finding 12: Onboarding/Tutorial type uses persistent progress state and sub-skill orchestration

mgai introduces a skill cluster for progressive skill adoption: agentic-workflow-tutorial (coordinator), awt-assess (assessment), awt-recommend (recommendations), awt-shop (skill marketplace).

Structural markers:

- **Persistent progress file**: `~/.claude/memory/agentic-workflow-progress.md` records level (0–6), role tags, completed actions, pending actions, and outcome log. Persists across sessions as a cross-session-stateful resource.
- **Level system**: 7 levels (Q&A / Workspace / Domain Memory / Tool Integration / Reusable Skills / Multi-Model Adversarial / Autonomous Loops) with defined criteria and evidence-based leveling.
- **New/returning user branching**: coordinator reads the progress file; if it exists, the session begins with outcome check on pending actions and quick level-up validation; if not, runs full assessment.
- **Sub-skill orchestration**: the coordinator skill invokes /awt-assess, /awt-recommend, /awt-shop as named sub-skills rather than embedding their logic directly.
- **Progressive Environment Probe**: awt-assess dispatches a foreground blocking subagent (on a lighter model) to scan the user's environment (CLAUDE.md, MCP configs, installed skills) and return a structured report. The probe result informs question skipping in the main assessment flow. Context savings: probe file listings stay in the subagent context; only the structured report returns to the parent.
- **Contextual Authoring Nudge**: during the session, the skill detects "I do this every..." or "I always have to..." signals and offers to scaffold a new SKILL.md from the described workflow.

### Finding 13: Meta-Skill type enforces skill structural guardrails and manages registry contribution

mgai update-skills is a skill whose subject matter is other skills.

Structural markers:

- **Guardrails Checklist**: a structural validation checklist run before and after edits. Checks: frontmatter fields restricted to allowed set; `name` must be hyphen-case max 64 chars; `description` includes all trigger conditions max 1024 chars; SKILL.md under 500 lines; no extra documentation files.
- **Progressive Disclosure Doctrine**: three-tier model for skill content — metadata (~100 words, always in context, primary trigger), SKILL.md body (loaded when skill triggers, instructions only), references/scripts (loaded on demand). This doctrine is applied as acceptance criteria, not guidelines.
- **Degrees of Freedom matching**: skill specificity should match task fragility — high freedom (creative tasks) → text instructions; medium freedom → pseudocode; low freedom (critical/brittle) → specific scripts with few parameters.
- **Registry contribution workflow**: skill creation includes steps to copy to registry, register in `marketplace.json` and `sources.json`, run `scripts/validate.sh`, and commit to PR branch.
- **Three invocation modes**: (1) update existing skill by name, (2) create new skill, (3) auto-detect from current session what learnings are worth capturing.

### Finding 14: Config Auto-Discovery is a new structural pattern for skill-local configuration

Five mgai skills (scaffold-manifest-builder, hyperpod-slurm, weekly-report, daily-work-summary, s3-cross-sync) share a pattern not observed in prior repos:

1. Check for `.config.json` in the skill's own directory.
2. If missing, run a discovery script that reads environment sources (SSH config, `~/.claude/settings.json`, AWS profiles, git config) to populate field values.
3. Save the config once: `cat > .config.json <<EOF ... EOF`.
4. On all subsequent invocations, read values with `jq -r '.field_name' .config.json`.

Fields discovered vary by skill: `cluster_host` and `pgrok_host` from SSH config (hyperpod-slurm); `github_username` from `gh api user` (weekly-report); AWS account IDs from `aws sts get-caller-identity` per profile (s3-cross-sync). Fields that cannot be discovered automatically are left empty and the skill asks the user once, then saves the answer.

This pattern decouples environment-specific values from the skill body, allowing the same skill to work in different user environments without modification.

### Finding 15: Six new intra-skill components appear in mgai

Six components not observed in prior repos emerge from mgai skills. See the Intra-Skill Component Catalog section for full definitions (components 30–35).

---

## Intra-Skill Component Catalog

The following 35 components are reusable structural patterns found *within* skills, independent of skill type. They recombine across types and repos.

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
- scaffold-manifest-builder: discovers `aws_account_ecr` from Dockerfile grep; hyperpod-slurm: discovers `cluster_host` from `~/.ssh/config`; s3-cross-sync: discovers AWS account IDs per environment from `aws sts get-caller-identity`
- Analogy: lazy initialization / one-time setup constructor

---

### Group 3: Data Routing

**10. Fixed Output Template** — A required output schema with named sections and fields specified in the skill body.

**11. Index-Based Routing** — Read a small index file first; use the index to determine which specific large files to load.

**12. Decision Table Lookup** — A table mapping input conditions to prescribed actions.

**13. Quantification Gate** — A named action is blocked until a numeric estimate is produced.

**14. Post-Filter** — Results from a broad query are filtered by a secondary criterion after collection.

---

### Group 4: Interaction Patterns

**15. Mandatory Subagent Invocation** — A specific subagent MUST be called at a named phase and cannot be skipped.

**16. Devil's Advocate Challenge Loop** — A dedicated adversarial subagent returns a verdict (ACCEPT/WEAK/REJECT); the parent must address each challenge before proceeding.

**17. One-Question-at-a-Time Dialogue** — AskUserQuestion is declared the only channel for user input; one question per message.

**18. Incremental Draft Checkpoint** — After 3–4 clarification rounds, the skill shows a partial draft and asks for explicit confirmation before proceeding.

**19. A/B Verification Procedure** — Same-system before measurement → apply exactly one change → after measurement; specifies what to record at each step.

**20. Interactive Trigger on Condition** — A skill section activates only when a named detectable signal is found in context; asks for confirmation before applying.

**31. Multi-Turn Round Framework** — Structured N-round interrogation protocol for stateless external AI CLIs. Each round applies a different analytical lens; context from the previous round is explicitly carried forward in the next prompt. Standard lens sequence: recommendation → inversion ("what guarantees failure?") → pre-mortem (stress-test weak points) → generalization → synthesis.
- codex-consultation and cursor-consultation define 5+ rounds; round files named with mktemp session prefix (`/tmp/codex-session-XXXXXX-rN-prompt.txt`)
- Analogy: structured adversarial iteration with explicit state handoff

**32. Progressive Environment Probe** — A foreground blocking subagent dispatched on a lighter model to scan the user's local environment (config files, installed tools, directory structure). Returns a structured report to the parent; the probe's full file listings stay in the subagent context (context savings). Parent uses the report to skip questions or tailor the main flow.
- awt-assess: probe runs on `sonnet` model; checks CLAUDE.md, MCP servers, installed skills, memory files, hooks; returns `ENV_PROBE_REPORT` with `INFERRED_LEVEL` and `KEY_FINDING`
- Analogy: environment introspection with context budget delegation

**33. Multi-Agent Team Delegation with Lightweight Validator** — The parent creates a named team of N agents with differentiated tool access profiles (each agent authorized for different MCPs/CLIs). After all agents complete, a lightweight validator model (haiku-class) checks the assembled output against format criteria. On failure, the parent corrects and re-validates; loop capped at a fixed number of retries.
- weekly-report: 6 agents (github-researcher, glean-slack-researcher, glean-docs-researcher, atlassian-researcher, glean-chat-researcher, format-reviewer); haiku validates format; 3-retry cap
- Analogy: parallel map-reduce with a cheap final linter

---

### Group 5: Quality Enforcement

**21. Quality Checklist** — A named set of yes/no criteria run in sequence before the skill output is declared complete.

**22. Anti-Pattern Table** — A two-column table of [tempting pattern → why it's wrong]. The tempting pattern is named first.

**23. Correct/Incorrect Code Pair** — Each rule illustrated by wrong code first (with inline failure mode annotation), then correct code.

**24. Priority Tier Classification** — Items classified into a small named set of tiers at collection time; tier determines processing order and output depth.

**25. Graceful Degradation with Source Note** — If a data source is unavailable, skip it, record its absence in a named output field, and continue.

**34. Output Style Enforcement** — A dedicated section specifying writing style with DO/DON'T annotated examples, a situational decision table (when to use each style), and an anti-pattern list with ❌ markers. Applied to skills whose outputs must conform to a specific prose style (narrative vs bullet, context depth requirements, named sub-item labels).
- daily-work-summary: DO examples show multi-sentence narrative with inline links; DON'T examples show terse bullet lists; decision table maps situation (single bug fix, technical deep dive, WIP progress) to prescribed style; named labels (왜?, 에러 메시지:, 원인:, 삽질 포인트:)
- Analogy: style guide enforced as acceptance criteria within the skill body

**35. Platform Exclusive Path Declaration** — An explicit negation of alternative execution paths that might appear valid but are not. Placed at the top of the skill body as a bold warning. Guards against hallucination-based drift toward wrong tool or method. Often triggered by presence of similarly-named but inapplicable alternatives in the organization's tooling.
- cpts-deploy: "CPTS services do NOT use Jenkins. Always use this skill instead of searching for Jenkins jobs." Gotchas section adds: "Glean이 Jenkins 배포 방법을 추천하면 무시할 것"
- Analogy: compile-time exclusion / negative assertion at entry point

---

### Group 6: Composition Patterns

**26. Skill Context Assignment** — Each spawned agent role receives a named list of skills injected into its spawn prompt.

**27. Dispatch Router** — The skill maps the incoming request type to a named sub-document and delegates.

**28. Pre-Execution Warm-Up** — A precondition that every invocation must complete before main logic begins; repeats on every call regardless of the request.

**29. Keyword-Based Auto-Link** — A corpus scan for keyword matches against a named index; matching items are automatically linked without manual intervention.

---

## Evidence / Details

### Skill-to-type mapping

**addyosmani/agent-skills** — all Process-Enforcement or Diagnostic (20 skills; unchanged from v2)

**vercel-labs/agent-skills** — Rule Library (5) and Action/Automation (2) (unchanged from v2)

**EveryInc/compound-engineering-plugin** — ~35 skills spanning Process-Enforcement, Orchestrator, Diagnostic, Action/Automation, Rule Library (unchanged from v2)

**hyperconnect/hc-agent-registry (ailab)** — 2 skills: literature-survey (Orchestrator Batch Pipeline), paper-summary (Action/Automation) (unchanged from v2)

**TinderBackend/tinder-ml-skill-registry** — 47 skills spanning all seven types from v2 (unchanged from v2; see v2 for full table)

**matchgroup-ai/mgai-skills-registry** (40 fully read; 30 classified by name)

| Skill | Category | Type | Corpus size | Interaction | Persistence |
|---|---|---|---|---|---|
| codex-consultation | consultation | Multi-Model Consultation | Single-item | Autonomous | Stateless |
| cursor-consultation | consultation | Multi-Model Consultation | Single-item | Autonomous | Stateless |
| cmux-agents | consultation | Multi-Model Consultation | Single-item | Autonomous | Stateless |
| agentic-workflow-tutorial | onboarding | Onboarding/Tutorial | Bounded-set | Interactive | Cross-session-stateful |
| awt-assess | onboarding | Onboarding/Tutorial | Single-item | Interactive | Cross-session-stateful |
| awt-recommend | onboarding | Onboarding/Tutorial | Bounded-set | Interactive | Cross-session-stateful |
| update-skills | authoring | Meta-Skill | Bounded-set | Interactive | Stateless |
| awt-shop | tooling | Meta-Skill | Bounded-set | Interactive | Stateless |
| skill-router | tooling | Meta-Skill | Bounded-set | Autonomous | Stateless |
| weekly-report | reporting | Orchestrator (Domain-Specialized) | Bounded-set | Interactive | Cross-session-stateful |
| daily-work-summary | reporting | Multi-Source Aggregator | Bounded-set | Autonomous | Stateless |
| agent-browser | tooling | Action/Automation (Tool Reference) | Single-item | Autonomous | Stateless |
| using-worktree | tooling | Action/Automation | Bounded-set | Autonomous | Stateless |
| scaffold-manifest-builder | infra | Action/Automation (Script-Gated) | Bounded-set | Interactive | Stateless |
| hyperpod-slurm | infra | Knowledge Connector | Single-item | Autonomous | Cross-session-stateful |
| cpts-deploy | infra | Action/Automation | Single-item | Autonomous | Stateless |
| server-error-triage | infra | Diagnostic/Investigation | Single-item | Autonomous | Stateless |
| okta-alb-session-triage | infra | Diagnostic/Investigation | Single-item | Autonomous | Stateless |
| s3-cross-sync | infra | Action/Automation | Bounded-set | Autonomous | Stateless |
| aws-sso-auto-login | automation | Action/Automation (Hook Config) | Single-item | Autonomous | Stateless |
| mcp-atlassian | tooling | Rule Library (reference) | Single-item | Autonomous | Stateless |
| translation | authoring | Process-Enforcement | Bounded-set | Autonomous | Stateless |
| airflow-monitor | ml-ops | Knowledge Connector | Single-item | Autonomous | Cross-session-stateful |
| databricks-notebook | ml-ops | Knowledge Connector | Single-item | Autonomous | Stateless |
| databricks-sql | ml-ops | Knowledge Connector | Single-item | Autonomous | Stateless |
| dbx-handler | ml-ops | Knowledge Connector | Bounded-set | Autonomous | Stateless |
| scaffold-context | infra | Living Knowledge Base | N/A | Autonomous | Stateless |
| multi-replica-operations | infra | Living Knowledge Base | N/A | Autonomous | Stateless |
| kaizen | authoring | Living Knowledge Base | N/A | Autonomous | Stateless |
| workplan | authoring | Process-Enforcement (Interactive) | Bounded-set | Interactive | Stateless |
| brainstorming | authoring | Process-Enforcement (Interactive) | Bounded-set | Interactive | Stateless |
| meta-review | authoring | Process-Enforcement (Gated) | Bounded-set | Autonomous | Session-stateful |
| blast-radius-analyzer | infra | Diagnostic/Investigation | Single-item | Autonomous | Stateless |

mgai also contains ~37 additional skills classified by name: automation (approve-pr, file-organizer, git-hygiene, launchd-cron, mg-software-capitalization, pr-creation-context, slack-notify, sync-jira-ticket, terminal-ambiance, workday-expense → Action/Automation), ml-ops (fireflow-context, mmdd-description-generator, photoenhancer-logs, uctm-health, visualize-experiment-results → Knowledge Connector or Action/Automation), infra (scaffold-alphabetical-order, scaffold-empty-manifest, scaffold-kms-access, scaffold-mesh-http2-tuning, scaffold-metrics, system-debugging, system-optimization, tinder-kubernetes-operation → Diagnostic or Action/Automation), authoring (content-research-writer, humanizer, internal-comms, ml-skills-guide, prompt-engineering, software-architecture, writing-plans → Process-Enforcement or Rule Library), reporting (daily-recap, ship-learn-next → Multi-Source Aggregator), frontend (hands-free-mode, mobile-layout → Action/Automation), tooling (refresh-all, slack-interact → Action/Automation).

### Core subcomponent inventory by type

**Process-Enforcement**
1. Frontmatter: `name`, `description` (trigger-focused)
2. Overview: one-paragraph statement of purpose
3. When to Use / When NOT to Use: explicit include and exclude conditions
4. Gated workflow: named phases with explicit human-approval gates
5. Anti-rationalization table
6. Red flags list
7. Verification checklist
8. (Interactive variant) AskUserQuestion mandatory channel declaration
9. (Interactive variant) Incremental Draft Checkpoint

**Rule Library (vercel-labs)**
1. Frontmatter: `name`, `description`, `license`, `metadata` (author, version)
2. Priority-ranked rule index (CRITICAL → LOW)
3. Per-rule files: `rules/<slug>.md`
4. Compiled document: `AGENTS.md`
5. Metadata sidecar: `metadata.json`

**Living Knowledge Base (tinder-ml / mgai)**
1. Frontmatter: `name`, `description` (trigger phrases embedded)
2. Domain sections (one `##` section per topic)
3. Problem statement per section
4. Correct/Incorrect Code Pair
5. Per-section Quick Checklist
6. (Some sections) Interactive Trigger

**Action/Automation**
1. Frontmatter: `name`, `description`, `argument-hint` (EveryInc) / `allowed-tools` (mgai)
2. State detection block
3. Decision tree
4. Step sequences with inline shell code blocks
5. Environment-specific notes
6. Error handling / troubleshooting
7. Fallback paths

**Orchestrator**
1. Frontmatter: `name`, `description`
2. Phase structure
3. (Static team) Mode declaration, subagent dispatch contracts, parallel/sequential blocks
4. (Batch pipeline) Batch Iterator, Accumulator, Snowball Expansion, Coverage Exit Condition, Rate Limit Registry
5. (Template-Instantiated) Template location, variable interpolation, AskUserQuestion for parameters
6. (Domain-Specialized) Named team with differentiated tool access profiles, Lightweight Validator, retry loop cap

**Diagnostic/Investigation**
1. Frontmatter: `name`, `description`
2. Core principles stated at top
3. Phase execution table or numbered phases
4. Reproduction step (confirm failure before hypothesizing)
5. Hypothesis formation with causal chain gate
6. (With Devil's Advocate) Mandatory DA subagent, ACCEPT/WEAK/REJECT verdict handling
7. Structured close template
8. Anti-pattern table

**Knowledge Connector**
1. Frontmatter: `name`, `description`, optional `allowed-tools`
2. Config block (base URL, API version, state file paths); may use Config Auto-Discovery
3. Authentication flow with session state
4. API endpoint reference table
5. Error code table
6. Golden Rule (mandatory precondition)
7. (Domain connectors) Domain-specific convention tables

**Multi-Source Aggregator**
1. Frontmatter: `name`, `description`
2. Input parsing (date/person/query parameters)
3. Parallel Fan-Out across sources
4. Timezone / secondary criterion handling
5. Post-Filter
6. Synthesis rules
7. Fixed Output Template

**Multi-Model Consultation (mgai)**
1. Frontmatter: `name`, `description`, `allowed-tools` (restricts to target CLI commands)
2. Prerequisites block (CLI install check, auth status)
3. When to use (architecture decisions, debugging, code review, second opinion)
4. Prompt construction step (self-contained context; embed code snippets)
5. Invocation block (temp file naming via mktemp, CLI command with flags reference table)
6. Readiness / output handling (read response, clean up temp files)
7. (Multi-turn) Multi-Turn Round Framework with explicit context carry-forward
8. Synthesis output (convergence / divergence / unique insight per model)

**Onboarding/Tutorial (mgai)**
1. Frontmatter: `name`, `description`
2. Source material reference (source-of-truth playbook document)
3. Time check (AskUserQuestion: 5 min / 15 min / no rush)
4. New vs returning user branch (read persistent progress file)
5. (New) Environment probe opt-in → Progressive Environment Probe subagent → behavioral questions
6. (Returning) Outcome check on pending actions → level-up validation → fresh recommendations
7. Contextual Authoring Nudge (detect repeatable workflow signals)
8. Progress save (write/update `~/.claude/memory/agentic-workflow-progress.md`)

**Meta-Skill (mgai)**
1. Frontmatter: `name`, `description`
2. Three invocation modes (update existing / create new / auto-detect from session)
3. Guardrails Checklist (run before and after changes)
4. Progressive Disclosure Doctrine (3-tier model)
5. Degrees of Freedom matching table
6. Registry contribution workflow

---

## Reference (Appendix)

### Frontmatter field inventory

| Field | addyosmani | vercel-labs | EveryInc | hc | tinder-ml | mgai |
|---|---|---|---|---|---|---|
| name | yes | yes | yes | yes | yes | yes |
| description | yes | yes | yes | yes | yes | yes |
| metadata.author | no | yes | no | no | no | no |
| metadata.version | no | yes | no | no | no | no |
| license | no | yes | no | no | no | no |
| argument-hint | no | no | yes | no | no | no |
| team | no | no | no | yes | no | no |
| allowed-tools | no | no | no | no | no | yes |

### Skill counts by type

| Type | addyosmani | vercel-labs | EveryInc | hc | tinder-ml | mgai | Total |
|---|---|---|---|---|---|---|---|
| Process-Enforcement | 19 | 0 | ~5 | 0 | ~12 | ~10 | ~46 |
| Rule Library | 0 | 5 | ~3 | 0 | ~3 | ~3 | ~14 |
| Living Knowledge Base | 0 | 0 | 0 | 0 | ~3 | ~3 | ~6 |
| Action/Automation | 0 | 2 | ~17 | 1 | ~16 | ~22 | ~58 |
| Orchestrator | 0 | 0 | ~3 | 1 | ~3 | ~2 | ~9 |
| Diagnostic/Investigation | 1 | 0 | ~4 | 0 | ~4 | ~5 | ~14 |
| Knowledge Connector | 0 | 0 | 0 | 0 | ~6 | ~8 | ~14 |
| Multi-Source Aggregator | 0 | 0 | 0 | 0 | ~3 | ~3 | ~6 |
| Multi-Model Consultation | 0 | 0 | 0 | 0 | 0 | 3 | 3 |
| Onboarding/Tutorial | 0 | 0 | 0 | 0 | 0 | 3 | 3 |
| Meta-Skill | 0 | 0 | 0 | 0 | 0 | 3 | 3 |

mgai counts are approximate; ~30 of 70 skills were classified by name inference.

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
