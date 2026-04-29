# Agent Skill Taxonomy: Common Dimensions, Types, Core Subcomponents, and Intra-Skill Components

Sources:
- https://github.com/addyosmani/agent-skills (skills/, README.md, AGENTS.md, CONTRIBUTING.md, docs/skill-anatomy.md)
- https://github.com/vercel-labs/agent-skills (skills/, README.md, AGENTS.md)
- https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills (all skill directories)
- https://github.com/hyperconnect/hc-agent-registry/tree/main/skills/ailab (literature-survey, paper-summary)
- https://github.com/TinderBackend/tinder-ml-skill-registry/tree/main/skills (47 skills listed; 15 fully read: agent-team-templates, airflow-monitor, blast-radius-analyzer, brainstorming, daily-recap, databricks-catalog, inferentia-neuron-compilation, internal-comms, kaizen, meta-review, multi-replica-operations, scaffold-context, system-debugging-with-devils-advocate, tinder-ml-brain-manage, tinder-ml-brain-query, tinder-kubernetes-operation, workplan)
- note/idea/how-to-write-skill.md

---

## Changes from v1

- Added findings from 2 new repos (hc-agent-registry/ailab: 2 skills; tinder-ml-skill-registry: 47 skills total, 15+ fully read)
- Added Finding 8: Two additional skill types — Knowledge Connector and Multi-Source Aggregator
- Added Finding 9: Three new structural dimensions — corpus size, interaction model, persistence model
- Added Finding 10: Living Knowledge Base type extends Rule Library with correct/incorrect code pairs and interactive triggers
- Added major new section: Intra-Skill Component Catalog (29 components in 6 categories)
- Updated skill-to-type mapping, skill counts, and file layout appendix for new repos

---

## Conclusion

Skills across all five repositories share three universal structural dimensions: a frontmatter identity block, explicit activation conditions, and a structured process body. Beyond these universals, seven distinct skill types emerge: **Process-Enforcement**, **Rule Library**, **Living Knowledge Base**, **Action/Automation**, **Orchestrator**, **Diagnostic/Investigation**, and **Knowledge Connector**. A new aggregation mode, **Multi-Source Aggregator**, appears in tinder-ml and hc skills and may warrant designation as an eighth type. Three additional structural dimensions differentiate skill complexity: corpus size (single-item / bounded-set / large-corpus), interaction model (autonomous / interactive / headless), and persistence model (stateless / session-stateful / cross-session-stateful).

Beyond skill-level taxonomy, 29 reusable intra-skill components can be identified — patterns that appear *within* skills and recombine across types. These are organized into six categories: Control Flow, State Management, Data Routing, Interaction Patterns, Quality Enforcement, and Composition Patterns. They function analogously to programming constructs (for loop, factory pattern, accumulator): each component solves a recurring structural problem that spans skill types and repositories.

Critical numbers: addyosmani contributes 20 skills (all Process-Enforcement or Diagnostic); vercel-labs contributes 7 (Rule Library and Action/Automation); EveryInc contributes ~35+ (spanning all types); hc-agent-registry/ailab contributes 2 (Orchestrator and Action/Automation); tinder-ml-skill-registry contributes ~47 spanning all seven types.

---

## Key Findings

### Finding 1: Three dimensions are universal across all skills and repos

Every skill examined, regardless of repo or type, contains:

1. **Frontmatter identity block** — a `name` field (kebab-case slug) and a `description` field (trigger-focused, beginning with "Use when…"). vercel-labs adds `metadata.author`, `metadata.version`, and `license`. EveryInc adds `argument-hint`. hc-agent-registry adds `team`. tinder-ml extends `description` with inline trigger-phrase lists.

2. **Activation conditions** — where and how the skill fires. Appears in the frontmatter `description`, a dedicated "When to Use / When NOT to Use" body section, or both. addyosmani always includes a dedicated body section. vercel-labs puts trigger phrases in the description. EveryInc uses both description and an `<auto_invoke>` block. tinder-ml embeds trigger phrase lists directly in the description field.

3. **Process body** — a sequential or structured set of steps. The depth and complexity vary greatly by type, but every skill has at minimum a numbered or phased sequence.

### Finding 2: Seven types can be identified by operational mode

The following taxonomy spans all five repos (see Evidence section for skill-to-type mapping):

| Type | Operational mode | Primary repo |
|---|---|---|
| Process-Enforcement | Encodes a multi-step engineering practice with quality gates | addyosmani (all 20), tinder-ml |
| Rule Library | Encodes a prioritized catalog of domain rules with code examples | vercel-labs (5 skills) |
| Living Knowledge Base | Single large file encoding domain knowledge as correct/incorrect code pairs, organized by domain section | tinder-ml (scaffold-context, kaizen, multi-replica-operations) |
| Action/Automation | Automates a discrete operational task with environment-aware branching | vercel-labs, EveryInc, tinder-ml |
| Orchestrator | Coordinates parallel subagents or batch pipelines across phased workflows | EveryInc, tinder-ml, hc |
| Diagnostic/Investigation | Systematic root cause analysis through structured hypothesis testing | addyosmani, EveryInc, tinder-ml |
| Knowledge Connector | Wraps an authenticated internal service with session state, API reference, and error table | tinder-ml |

A Multi-Source Aggregator mode (fan-out to N parallel sources, post-filter, synthesize) appears in tinder-ml (daily-recap, tinder-ml-brain-query) and hc (literature-survey Phase 2). It shares traits with Orchestrator but differs in that its subagents are queries against external systems rather than task-executing agents.

### Finding 3: The Process-Enforcement type is defined by three exclusive subcomponents

(Unchanged from v1.) The addyosmani format explicitly names its anatomy in docs/skill-anatomy.md. Process-Enforcement skills uniquely include: (1) an anti-rationalization table, (2) a red flags list, and (3) a verification checklist. None appear in Rule Library or Action/Automation skills. They exist to combat agent tendency to shortcut process steps.

tinder-ml adds a fourth variant: **Interactive Process-Enforcement**, where AskUserQuestion is the mandatory channel for all user interaction. workplan and brainstorming declare "AskUserQuestion is MANDATORY — never ask questions via plain text." This prevents the agent from embedding questions inside prose responses that users may miss.

### Finding 4: The Rule Library type externalizes content into individual rule files; the Living Knowledge Base internalizes all rules into one file with code pairs

vercel-labs Rule Library uses a two-level structure: SKILL.md is an index with priority-ranked slugs; detail lives in individual `rules/<slug>.md` files. A compiled `AGENTS.md` and `metadata.json` sidecar complete the structure.

tinder-ml's Living Knowledge Base type (scaffold-context, kaizen) inverts this: all rules live in a single large SKILL.md, organized by domain section. Each rule follows a **Correct/Incorrect Code Pair** pattern — the wrong code is shown first with an annotation explaining the failure mode, then the correct code. Some sections include an **Interactive Trigger**: a list of detection signals that, when found in context, cause the skill to ask the user for confirmation before applying the section's rules. scaffold-context adds a "Quick Checklist" at the end of each domain section.

### Finding 5: The Orchestrator type now has two distinct sub-variants

(Extending v1.) EveryInc Orchestrators (ce-compound, ce-review) spawn a **static, predefined team** of named subagents. The team composition, roles, and model tiers are hardcoded.

hc-agent-registry literature-survey introduces a **Batch Pipeline Orchestrator**: the parent loops over a large corpus (200+ papers), delegating 20-item batches to subagents per iteration. The team composition is not fixed — each batch spawns N subagents determined by batch size. The parent accumulates state across batches (method-tracker) and reorders the work queue after each batch.

tinder-ml agent-team-templates introduces a **Template-Instantiated Orchestrator**: reads a YAML template file, interpolates variables, and spawns a team whose composition is determined at runtime by the template and user answers.

### Finding 6: The Knowledge Connector type introduces session state and API reference tables

tinder-ml contributes a skill type not present in earlier repos: the Knowledge Connector. Examples include airflow-monitor, databricks-catalog, and tinder-kubernetes-operation.

Structural markers:
- **Session state file** (`~/.airflow/session-state.json`): auth cookies persisted to disk and reloaded on each invocation
- **Re-authentication fallback**: on 401/302, the skill opens a browser via `agent-browser`, waits for user MFA completion, and saves the new session
- **API endpoint reference table**: `| Task | Command |` mapping of operations to curl commands or CLI commands
- **Error code table**: `| HTTP Code | Meaning | Action |` prescribing the response to each failure mode
- **Golden Rule**: a "always do this first" instruction that every invocation must follow (e.g., "Always describe a table before querying it")

databricks-catalog adds **domain-specific convention tables** (e.g., Tinder-specific column semantics, partition column rules, id type distinctions) that encode knowledge needed to write correct queries against internal schemas.

### Finding 7: Supplementary files are structured by type

(Updated from v1 with new repo data.)

| Type | File layout outside SKILL.md |
|---|---|
| Process-Enforcement (addyosmani) | `references/*.md` (shared checklists loaded on-demand) |
| Rule Library (vercel-labs) | `rules/*.md`, `AGENTS.md` (compiled), `metadata.json`, `resources/` (scripts) |
| Living Knowledge Base (tinder-ml) | No supplementary files — all content in SKILL.md; `examples/` subdirectory in dispatch-router variant |
| Action/Automation (vercel-labs) | `resources/` (shell scripts) |
| Action/Automation (EveryInc) | `references/` (optional) |
| Orchestrator (EveryInc) | `references/` (YAML schemas, templates), `assets/` (document templates) |
| Orchestrator (hc, Batch Pipeline) | `references/` (search-sources.md, templates.md) |
| Diagnostic (EveryInc, tinder-ml) | `references/` (anti-patterns, investigation techniques) |
| Knowledge Connector (tinder-ml) | No supplementary files — all in SKILL.md |

### Finding 8: Two additional skill types emerge from tinder-ml and hc

**Knowledge Connector** (see Finding 6 above): wraps an authenticated internal service. Structurally distinguished by session state management, browser-based re-auth fallback, API endpoint table, and error-code-to-action table. Not present in addyosmani, vercel-labs, or EveryInc.

**Multi-Source Aggregator**: fans out to multiple data sources in parallel, post-filters by a secondary criterion, and synthesizes a unified output. tinder-ml daily-recap fans out to 7 parallel Glean searches plus 2 local history files, then post-filters every result's UTC timestamp to the user's local timezone. tinder-ml tinder-ml-brain-query reads a team index plus person profiles in parallel, then routes to the specific recap files indicated. hc literature-survey Phase 2 fans out to multiple academic search sources. The defining structural feature is the **post-filter step**: broad query → collect all → filter by secondary criterion.

### Finding 9: Three new structural dimensions differentiate skill complexity

Beyond skill type, three dimensions vary independently and predict structural choices:

| Dimension | Values | Effect on structure |
|---|---|---|
| Corpus size | Single-item / Bounded-set / Large-corpus | Single-item: no loop. Bounded-set: fixed phases. Large-corpus: Batch Iterator + Accumulator + Coverage Exit |
| Interaction model | Autonomous / Interactive / Headless | Autonomous: no AskUserQuestion. Interactive: mandatory AskUserQuestion with one-question-at-a-time rule. Headless: no user interaction at all |
| Persistence model | Stateless / Session-stateful / Cross-session-stateful | Stateless: no files read/written outside output. Session-stateful: accumulator or locked plan written within one invocation. Cross-session-stateful: shared brain repo, session-state.json, rate-limit-backoff.md read/written across invocations |

### Finding 10: Living Knowledge Base type extends Rule Library with code pairs and interactive triggers

The Living Knowledge Base type (scaffold-context, kaizen, multi-replica-operations) differs from the vercel-labs Rule Library in four ways:

1. **Single file**: all rules in one large SKILL.md, not an index + per-rule files
2. **Correct/Incorrect Code Pair**: each rule shows the wrong code first with explicit failure mode annotation, then the correct code. The annotation explains *why* the wrong code fails, not just that it does.
3. **Interactive Trigger**: some domain sections begin with a "When to Trigger" block listing detectable signals. When any signal is present, the skill asks the user for confirmation before applying the section's rules. scaffold-context GPU thread pool section triggers on `scheduling.node_workload_taint: gpu` or GPU instance families in node_selector.
4. **Per-section Quick Checklist**: each domain section ends with a numbered checklist specific to that domain (4-7 items), separate from any skill-level quality checklist.

---

## Intra-Skill Component Catalog

The following 29 components are reusable structural patterns found *within* skills, independent of skill type. They recombine across types and repos. Each component name, definition, and example is drawn directly from observed skill content.

### Group 1: Control Flow

**1. Batch Iterator**
Process a fixed-size subset of a large work set per cycle; collect results; update state; advance to the next subset. Repeat until queue is exhausted or exit condition fires.
- literature-survey: 20 papers per batch; subagents read each group; results fed into method-tracker and queue
- Analogy: for loop with fixed stride

**2. Snowball Search Expansion**
Outputs from each processed item become new inputs to the work queue. The queue grows from its own results; the skill does not pre-enumerate all items at the start.
- literature-survey: each paper's related-works section yields new candidate papers added to the queue; queue re-sorted after each batch
- Analogy: BFS traversal where nodes produce new nodes

**3. Coverage Exit Condition**
The iteration loop exits when a coverage metric exceeds a named threshold, not when the queue empties. The metric is computed by comparing output against a requirements document.
- literature-survey: computes "requirements 20개 중 18개 = 90%"; exits at 95%; if below, adds 100 papers and repeats Phase 3-5
- Analogy: while (coverage < threshold) rather than while (queue not empty)

**4. Parallel Fan-Out**
N tasks launched concurrently in a single step; all results collected before the parent proceeds. Distinct from sequential branching in that order of completion does not matter.
- literature-survey: 4-5 subagents per batch launched in parallel; daily-recap: 7 Glean searches launched in parallel; EveryInc `<parallel_tasks>` block
- Analogy: Promise.all() / scatter-gather

**5. Sequential Gate**
Explicit block: the skill declares "do not advance to Phase X until Phase Y is complete." Often combined with a named artifact that must exist before the gate lifts.
- literature-survey: "Phase 4 starts only after Phase 3.7 complete"; meta-review: "DO NOT spawn dev agents before execution plan is written"; EveryInc WAIT directives in `<sequential_tasks>`
- Analogy: barrier / join point

---

### Group 2: State Management

**6. Accumulator**
A running aggregate (table, score, counter) updated after each batch cycle. The accumulated state influences routing or prioritization decisions in later cycles.
- literature-survey method-tracker: records per-method baseline-citation count, variant count, performance consistency; weights used to rerank queue after each batch
- tinder-ml-brain-query rollup: reads pre-aggregated rollup files before daily recaps for long time ranges
- Analogy: reduce() with mutable accumulator object

**7. Session State Cache**
Auth tokens or session cookies persisted to a named file after first authentication; read at the start of each invocation; re-authenticated via browser fallback when expired.
- airflow-monitor: `~/.airflow/session-state.json` + `~/.airflow/cookies.txt`; jq extracts cookies; 401/302 triggers Chrome re-auth via agent-browser; new session saved after success
- Analogy: token refresh with persistent store

**8. Locked Plan Artifact**
An intermediate document written to disk during execution and marked immutable ("LOCKED — do not modify after agents spawn") after a named trigger event. All subsequent agents reference this artifact.
- meta-review: writes `/tmp/execution-plan-<team-name>.md`; locked after GO decision; each spawned agent receives "Read this plan and follow it exactly"
- literature-survey: `[주제]-requirements.md` approved by user in Phase 1; not modified in later phases
- Analogy: write-once configuration / frozen spec

**9. Rate Limit Backoff Registry**
A per-service file tracking the last successful retry delay. On HTTP 429 or empty response, the skill reads the registry file, waits the recorded delay, retries, then writes back on success. Delay doubles on failure (30→60→120→300s max). On 300s failure, switches to alternate source.
- literature-survey: `[출력폴더]/rate-limit-backoff.md`; Google blocked → Semantic Scholar; Semantic Scholar blocked → arxiv
- Analogy: exponential backoff with persistent state

---

### Group 3: Data Routing

**10. Fixed Output Template**
A required output schema with named sections and fields specified in the skill body. The skill contract declares which sections are required vs. optional and what each field must contain.
- paper-summary: 14-section template (meta, elevator pitch, three-line summary, method, limitations, community reaction, etc.)
- daily-recap: Timeline + Project Updates + Action Items (Red/Yellow/Green) + Key Summary; icons prescribed
- workplan: Motivation + Delivery Date + Expected Outcome + Work Overview + Risks; template literal in skill body
- Analogy: schema / struct definition for output

**11. Index-Based Routing**
Read a small index file first; use the index to determine which specific large files to load. The index encodes routing metadata (slugs, dates, person IDs, project keywords) to avoid scanning the full corpus.
- tinder-ml-brain-query: reads `team-members.yaml` and `projects/INDEX.md` in parallel before reading any person's recap files
- internal-comms: maps communication type to `examples/<type>.md` file; loads only the matching file
- Analogy: database index lookup before full table scan

**12. Decision Table Lookup**
A table mapping input conditions (HTTP status code, error message, resource type, detected signal) to prescribed actions. The agent selects the row matching current state and executes the corresponding action.
- airflow-monitor: `| 200 | Success | Parse response |`, `| 401 | Unauthorized | Re-authenticate via browser |`
- inferentia-neuron-compilation: errors table with 11 rows mapping error string to cause and fix
- blast-radius-analyzer: overengineering patterns table; system-debugging: anti-patterns table
- Analogy: switch/case or dispatch table

**13. Quantification Gate**
A named action is blocked until a numeric estimate is produced. The gate forces measurement before recommendation; if quantities cannot be produced, the skill declares the change unscoped.
- blast-radius-analyzer: requires `size_per_user × inactive_user_count`, `risk = cold_start_cost / memory_savings`, and percentage of total memory before classifying any resource as Must-have/Nice-to-have/Unnecessary
- Analogy: precondition assertion on numeric input

**14. Post-Filter**
Results from a broad query are filtered by a secondary criterion after collection. The filter is applied explicitly in code, not in the query itself, because the query API does not support the required criterion.
- daily-recap: Glean returns UTC timestamps; each result is converted to the user's local timezone (detected at runtime via `date +%z`); results outside the local target date are discarded
- tinder-ml-brain-query: recaps stored with local-timezone dates; search uses a broad UTC range then filters by local date
- Analogy: filter() applied to a pre-fetched result set

---

### Group 4: Interaction Patterns

**15. Mandatory Subagent Invocation**
A specific subagent MUST be called at a named phase and cannot be skipped. The skill body uses explicit "MANDATORY", "do NOT skip this step", or "never skip" language at the invocation site.
- system-debugging-with-devils-advocate: "MANDATORY: Launch the `system-debugging-devils-advocate` agent. Do NOT skip this step."
- workplan Phase 5.5: "Always spawn the subagent — your own blind spots are the problem"
- literature-survey Phase 3: subagent delegation for each batch group is structural, not optional
- Analogy: required function call / non-optional dependency

**16. Devil's Advocate Challenge Loop**
A dedicated adversarial subagent receives the parent's hypothesis or plan and returns a verdict: ACCEPT, WEAK, or REJECT with specific counter-evidence. The parent must address each challenge before proceeding. WEAK → run the requested tests and re-submit. REJECT → return to an earlier phase.
- system-debugging-with-devils-advocate: parent presents hypothesis + full evidence chain + discarded hypotheses; DA returns verdict; parent responds with evidence, acknowledged limitation, new test, or "separate issue"
- workplan: devil's advocate subagent challenges each deliverable; parent presents each identified risk to user for CONFIRM/NOT A RISK/ALREADY MITIGATED; dismissed risks are not included in the document
- Analogy: adversarial review with structured response protocol

**17. One-Question-at-a-Time Dialogue**
AskUserQuestion is declared the only channel for user input. One question per message. Multiple-choice format is preferred; open-ended only when necessary. If a topic requires multiple questions, they are serialized across messages.
- workplan: "Every question to the user MUST use the AskUserQuestion tool. Never ask questions via plain text output."
- brainstorming: "Only one question per message — if a topic needs more exploration, break it into multiple questions. Prefer multiple choice questions when possible."
- Analogy: form wizard with single-field-per-page constraint

**18. Incremental Draft Checkpoint**
After 3-4 clarification rounds, the skill shows a partial draft (specific named sections only) and asks for explicit confirmation before proceeding to the next section. Not "show everything at the end."
- workplan Phase 4: shows Motivation + Expected Outcome only; asks "What needs to change? Or is this solid enough?"
- brainstorming: "Present the design in small sections (200-300 words), checking after each section whether it looks right"
- Analogy: streaming output with mid-stream approval gates

**19. A/B Verification Procedure**
Same-system before measurement → apply exactly one change → after measurement. The procedure specifies what to record at each step (pod name, image tag, restart count, exact commands, timestamps) to prove causality rather than correlation.
- system-debugging-with-devils-advocate Phase 4: "Same pod, same image, same environment. Before fix: demonstrate failure (with logs). Apply fix: change ONLY the hypothesized variable. After fix: demonstrate success."
- inferentia-neuron-compilation Step 5: GPU endpoint vs Neuron endpoint; identical inputs; pass criteria (Pearson r > 0.9999, max diff < 0.02, zero classification flips at production threshold)
- Analogy: controlled experiment / A/B test with evidence requirements

**20. Interactive Trigger on Condition**
A skill section activates only when a named detectable signal is found in context. When the signal is present, the skill asks the user for confirmation before applying the section's rules; if the user declines, the section is skipped.
- scaffold-context GPU thread pool section: trigger signals are `scheduling.node_workload_taint: gpu`, GPU instance families in node_selector, `autoscale.metric: gpu_usage`, service name containing `serve`/`triton`/`vllm`/`llm`; if present, AskUserQuestion: "Does it run PyTorch, HuggingFace, or BLAS-linked models?"
- Analogy: conditional code path guarded by runtime detection + user confirmation

---

### Group 5: Quality Enforcement

**21. Quality Checklist**
A named set of criteria run in sequence before the skill output is declared complete. Each criterion is a yes/no question. The checklist is placed at the end of the skill body and covers the output produced by all preceding phases.
- workplan: 14-item checklist (motivation structure, delivery date format, outcome specificity, risk completeness, language, etc.)
- blast-radius-analyzer: 6-item checklist (scope reduction, naming, percentage threshold, cold-start cost, consistency motive check, partial rollout)
- addyosmani Process-Enforcement skills: verification checklist with evidence requirements
- Analogy: pre-commit hook list

**22. Anti-Pattern Table**
A two-column table of [tempting pattern → why it's wrong (or dangerous)]. The tempting pattern is named first to make it recognizable. Present in both diagnostic and knowledge base type skills.
- blast-radius-analyzer: "Unify all configs → Forces migration on working state"; "TTL on everything → Tiny states don't need TTL"
- system-debugging-with-devils-advocate: "Changing multiple things between tests → Can't isolate which change fixed it"; "Stopping at the first plausible explanation → The first explanation is often wrong"
- kaizen: 4 red-flag sections, one per pillar
- Analogy: linting rules with explanations

**23. Correct/Incorrect Code Pair**
Each rule is illustrated by two code blocks: the wrong code first (with an inline annotation explaining the failure mode), then the correct code (with annotation explaining why it works). The wrong code is shown before the correct code to make the failure mode salient.
- scaffold-context: every section (DynamoDB TTL, service mesh egress, autoscale select_policy, health check port/path, etc.) follows this pattern; wrong code is labeled `#### Incorrect` with a failure mode note; correct code is labeled `#### Correct`
- kaizen: Good/Bad code pairs for each pillar (iteration, type safety, error handling, abstraction)
- Analogy: ESLint rule documentation format (bad → good example)

**24. Priority Tier Classification**
Items are classified into a small named set of tiers at collection time. The tier determines processing order (higher tiers processed first) and output detail depth (higher tiers analyzed in depth; lower tiers summarized briefly).
- literature-survey: 4 tiers (1순위 NeurIPS/ICML/bigtech, 2순위 CVPR/KDD, 3순위 ICCV/WWW/arXiv+prominent, 4순위 others); tier assignment uses venue + citation count + affiliation
- literature-survey relevance: Core/Related/Peripheral classification applied before tier; Peripheral items demoted two tiers
- blast-radius-analyzer: Must-have / Nice-to-have / Unnecessary; only Must-have changes implemented
- Analogy: priority queue with named levels

**25. Graceful Degradation with Source Note**
If a data source is unavailable or returns no data, the skill skips that source, records its absence in a named output field, and continues with the remaining sources. The output is partial but valid, not an error.
- daily-recap: "If neither history file exists, skip Step 1 entirely and proceed to Step 2. Note: 'No local session history available (history files not found)'."
- tinder-ml-brain-query: "If a person has no recaps, say 'No context uploaded by {person} yet' instead of erroring"
- Analogy: optional dependency with degraded-mode output

---

### Group 6: Composition Patterns

**26. Skill Context Assignment**
Each spawned agent role receives a named list of skills injected into its spawn prompt. Different roles receive different skill sets. The orchestrating skill declares which skills each role needs in its agent-team template or spawn prompt.
- agent-team-templates: Architect → scaffold-context, software-architecture, kaizen; QA Engineer → superpowers:test-driven-development, superpowers:verification-before-completion; Meta Reviewer → skills from entire session
- EveryInc ce-compound: each subagent prompt names the skills it should apply

**27. Dispatch Router**
The skill maps the incoming request type to a named sub-document and delegates. The skill body encodes no domain knowledge — it is a pure routing layer that reads the matching document and applies its instructions.
- internal-comms: identifies communication type → loads `examples/3p-updates.md`, `examples/company-newsletter.md`, etc.; all formatting, tone, and content rules are in the loaded file, not in SKILL.md
- tinder-ml-brain-query: routes person-activity queries to recap files, project-status queries to project files, team-overview queries to team member recaps; routing logic is in SKILL.md; content is in the loaded files
- Analogy: router / dispatcher pattern; strategy pattern

**28. Pre-Execution Warm-Up**
A precondition that every invocation must complete before main logic begins. Not a conditional phase — a mandatory prefix that repeats on every call regardless of the request.
- tinder-ml-brain-query: `cd $BRAIN_PATH && git pull origin main --quiet` — always pull latest before any query
- airflow-monitor: load session state and test cookie validity before any API call
- databricks-catalog: "Always describe a table before querying it. Never guess column names or types." (Golden Rule repeated at top)
- Analogy: constructor initialization / setup() method

**29. Keyword-Based Auto-Link**
A corpus scan for keyword matches against a named index. Matching items are automatically linked to the project file in the index without manual intervention. New matches are appended; existing links are not overwritten.
- tinder-ml-brain-manage enrich command: reads `projects/INDEX.md` to get all project keywords; scans all person recaps matching the date range; for each recap, checks content for keyword matches against the index; adds matching recaps to each project's "Related Recaps" section with a 5-10 word description
- Analogy: tag-based auto-categorization / inverted index update

---

## Evidence / Details

### Skill-to-type mapping

**addyosmani/agent-skills** (all Process-Enforcement or Diagnostic)

| Skill | Type | Lifecycle phase |
|---|---|---|
| idea-refine | Process-Enforcement | Define |
| spec-driven-development | Process-Enforcement | Define |
| planning-and-task-breakdown | Process-Enforcement | Plan |
| incremental-implementation | Process-Enforcement | Build |
| test-driven-development | Process-Enforcement | Build |
| context-engineering | Process-Enforcement | Build |
| source-driven-development | Process-Enforcement | Build |
| frontend-ui-engineering | Process-Enforcement | Build |
| api-and-interface-design | Process-Enforcement | Build |
| browser-testing-with-devtools | Process-Enforcement | Verify |
| debugging-and-error-recovery | Diagnostic/Investigation | Verify |
| code-review-and-quality | Process-Enforcement | Review |
| code-simplification | Process-Enforcement | Review |
| security-and-hardening | Process-Enforcement | Review |
| performance-optimization | Process-Enforcement | Review |
| git-workflow-and-versioning | Process-Enforcement | Ship |
| ci-cd-and-automation | Process-Enforcement | Ship |
| deprecation-and-migration | Process-Enforcement | Ship |
| documentation-and-adrs | Process-Enforcement | Ship |
| shipping-and-launch | Process-Enforcement | Ship |

**vercel-labs/agent-skills**

| Skill | Type |
|---|---|
| react-best-practices | Rule Library |
| web-design-guidelines | Rule Library |
| react-native-skills | Rule Library |
| react-view-transitions | Rule Library |
| composition-patterns | Rule Library |
| deploy-to-vercel | Action/Automation |
| vercel-cli-with-tokens | Action/Automation |

**EveryInc/compound-engineering-plugin** (selected)

| Skill | Type |
|---|---|
| ce-compound | Orchestrator |
| ce-review | Orchestrator |
| ce-compound-refresh | Orchestrator |
| ce-debug | Diagnostic/Investigation |
| ce-plan | Process-Enforcement |
| ce-ideate | Process-Enforcement |
| ce-brainstorm | Process-Enforcement |
| ce-optimize | Process-Enforcement |
| git-commit | Action/Automation |
| git-commit-push-pr | Action/Automation |
| git-worktree | Action/Automation |
| git-clean-gone-branches | Action/Automation |
| changelog | Action/Automation |
| ce-setup | Action/Automation |
| ce-sessions | Action/Automation |
| deploy-docs | Action/Automation |
| todo-create | Action/Automation |
| todo-resolve | Action/Automation |
| todo-triage | Action/Automation |
| ce-slack-research | Action/Automation |
| document-review | Diagnostic/Investigation |
| agent-native-audit | Diagnostic/Investigation |
| onboarding | Rule Library (persona) |
| dhh-rails-style | Rule Library (persona) |
| andrew-kane-gem-writer | Rule Library (persona) |

**hyperconnect/hc-agent-registry (ailab)**

| Skill | Type | Corpus size | Interaction | Persistence |
|---|---|---|---|---|
| literature-survey | Orchestrator (Batch Pipeline) | Large-corpus (200+) | Autonomous (with approval gate at Phase 1) | Cross-session-stateful |
| paper-summary | Action/Automation | Bounded-set (wip.md list) | Autonomous | Stateless |

**TinderBackend/tinder-ml-skill-registry** (15 fully read)

| Skill | Type | Corpus size | Interaction | Persistence |
|---|---|---|---|---|
| agent-team-templates | Orchestrator (Template-Instantiated) | Bounded-set | Interactive | Stateless |
| airflow-monitor | Knowledge Connector | Single-item | Autonomous | Cross-session-stateful |
| blast-radius-analyzer | Diagnostic/Investigation | Single-item | Autonomous | Stateless |
| brainstorming | Process-Enforcement (Interactive) | Bounded-set | Interactive | Stateless |
| daily-recap | Multi-Source Aggregator | Bounded-set | Autonomous | Cross-session-stateful |
| databricks-catalog | Knowledge Connector | Single-item | Autonomous | Stateless |
| inferentia-neuron-compilation | Action/Automation (Runbook) | Single-item | Autonomous | Stateless |
| internal-comms | Dispatch Router | Single-item | Autonomous | Stateless |
| kaizen | Living Knowledge Base | N/A (reference) | Autonomous | Stateless |
| meta-review | Process-Enforcement (Gated) | Bounded-set | Autonomous | Session-stateful |
| multi-replica-operations | Living Knowledge Base | N/A (reference) | Autonomous | Stateless |
| scaffold-context | Living Knowledge Base | N/A (reference) | Autonomous (with Interactive Triggers) | Stateless |
| system-debugging-with-devils-advocate | Diagnostic/Investigation | Single-item | Autonomous | Stateless |
| tinder-kubernetes-operation | Knowledge Connector (minimal) | Single-item | Autonomous | Stateless |
| tinder-ml-brain-manage | Action/Automation | Bounded-set | Interactive | Cross-session-stateful |
| tinder-ml-brain-query | Multi-Source Aggregator | Bounded-set | Autonomous | Cross-session-stateful |
| workplan | Process-Enforcement (Interactive) | Bounded-set | Interactive | Stateless |

tinder-ml-skill-registry has 47 skills total. The ~32 skills not fully read are classified by name inference:
- Knowledge Connector: ~6 (databricks-notebook, databricks-sql, dbx-handler, fireflow-context, scaffold-metrics, photoenhancer-logs)
- Action/Automation: ~14 (git-hygiene, file-organizer, tinder-ml-brain-upload, tinder-ml-brain-rollup, visualize-experiment-results, mmdd-description-generator, slack-notify, pr-creation-context, approve-pr, agentic-workflow-tutorial, scaffold-empty-manifest, scaffold-alphabetical-order, ship-learn-next, uctm-health)
- Living Knowledge Base / Rule Library: ~5 (scaffold-mesh-http2-tuning, prompt-engineering, ml-skills-guide, software-architecture, content-research-writer)
- Process-Enforcement: ~5 (writing-plans, awt-assess, awt-recommend, awt-shop, system-optimization)
- Diagnostic: ~2 (infra-cost-analysis, awt-assess may overlap)

### Core subcomponent inventory by type

**Process-Enforcement**
1. Frontmatter: `name`, `description` (trigger-focused)
2. Overview: one-paragraph statement of purpose
3. When to Use / When NOT to Use: explicit include and exclude conditions
4. Gated workflow: named phases with explicit human-approval gates; "do not advance until" language
5. Anti-rationalization table: two-column (common excuse → rebuttal)
6. Red flags list: observable signals that the agent is skipping steps
7. Verification checklist: evidence requirements at task end
8. (Interactive variant) AskUserQuestion mandatory channel declaration
9. (Interactive variant) Incremental Draft Checkpoint after core sections

**Rule Library (vercel-labs)**
1. Frontmatter: `name`, `description`, `license`, `metadata` (author, version)
2. When to Apply: task-type trigger conditions
3. Priority-ranked rule index: CRITICAL → LOW, each entry is a slug + one-line description
4. Per-rule files: `rules/<slug>.md` — explanation, wrong example, correct example
5. Compiled document: `AGENTS.md` — all rules in one file for bulk loading
6. Metadata sidecar: `metadata.json`

**Living Knowledge Base (tinder-ml)**
1. Frontmatter: `name`, `description` (trigger phrases embedded)
2. Domain sections: one `##` section per topic area
3. Problem statement per section: narrative of the failure mode
4. Correct/Incorrect Code Pair: wrong code first with annotation, then correct code
5. Per-section Quick Checklist: 4-7 numbered items
6. (Some sections) Interactive Trigger: detection signals + AskUserQuestion before applying

**Action/Automation**
1. Frontmatter: `name`, `description`, `argument-hint` (EveryInc)
2. State detection block: pre-condition checks
3. Decision tree: branching paths keyed on detected state
4. Step sequences: numbered commands per path, with inline shell code blocks
5. Environment-specific notes: runtime-aware behavior
6. Error handling / troubleshooting: named error conditions with prescribed responses
7. Fallback paths

**Orchestrator**
1. Frontmatter: `name`, `description`, `argument-hint`
2. (Static team) Mode declaration, subagent dispatch contracts, parallel/sequential XML blocks, output assembly, `<critical_requirement>` tags, `<auto_invoke>` block
3. (Batch pipeline) Phase structure, Batch Iterator, Accumulator (method-tracker), Snowball Expansion, Coverage Exit Condition, rate limit backoff registry
4. (Template-Instantiated) Template location, variable interpolation spec, AskUserQuestion for required parameters, team creation + task creation + agent spawn sequence

**Diagnostic/Investigation**
1. Frontmatter: `name`, `description`
2. Core principles: invariant rules stated at top
3. Phase execution table or numbered phases
4. Reproduction step: confirm the failure exists before hypothesizing
5. Hypothesis formation with causal chain gate
6. (With Devil's Advocate) Mandatory DA subagent invocation, ACCEPT/WEAK/REJECT verdict handling
7. Structured close template: fixed-field output (Problem, Root Cause, Fix, Prevention)
8. Anti-pattern table

**Knowledge Connector**
1. Frontmatter: `name`, `description` (trigger phrases for system names)
2. Configuration table: base URL, API version, state file paths
3. Authentication flow: session state load → API call → 401/302 → browser re-auth → save → retry
4. API endpoint reference table: operation → command mapping
5. Error code table: HTTP code / error type → action
6. Golden Rule: mandatory precondition before any operation
7. (Domain connectors) Domain-specific convention tables encoding internal schema semantics

**Multi-Source Aggregator**
1. Frontmatter: `name`, `description`
2. Input parsing: extract date/person/query parameters from user message
3. Parallel Fan-Out: N queries launched simultaneously across different sources and strategies
4. Timezone / secondary criterion handling: detect offset at runtime; convert all source timestamps before filtering
5. Post-Filter: discard results outside the local target criterion
6. Synthesis rules: fact vs. inference distinction, conflict handling, recency priority
7. Fixed Output Template with named sections

---

## Reference (Appendix)

### Frontmatter field inventory

| Field | addyosmani | vercel-labs | EveryInc | hc | tinder-ml |
|---|---|---|---|---|---|
| name | yes | yes | yes | yes | yes |
| description | yes | yes | yes | yes | yes |
| metadata.author | no | yes | no | no | no |
| metadata.version | no | yes | no | no | no |
| license | no | yes | no | no | no |
| argument-hint | no | no | yes | no | no |
| team | no | no | no | yes | no |

### Skill counts by type

| Type | addyosmani | vercel-labs | EveryInc | hc (ailab) | tinder-ml | Total |
|---|---|---|---|---|---|---|
| Process-Enforcement | 19 | 0 | ~5 | 0 | ~12 | ~36 |
| Rule Library | 0 | 5 | ~3 | 0 | ~3 | ~11 |
| Living Knowledge Base | 0 | 0 | 0 | 0 | ~3 | ~3 |
| Action/Automation | 0 | 2 | ~17 | 1 | ~16 | ~36 |
| Orchestrator | 0 | 0 | ~3 | 1 | ~3 | ~7 |
| Diagnostic/Investigation | 1 | 0 | ~4 | 0 | ~4 | ~9 |
| Knowledge Connector | 0 | 0 | 0 | 0 | ~6 | ~6 |
| Multi-Source Aggregator | 0 | 0 | 0 | 0 | ~3 | ~3 |

tinder-ml counts are approximate; ~32 of 47 skills were classified by name inference rather than full read.

### Intra-Skill Component cross-reference by type

| Component | Process-E | Rule Lib | Living KB | Action | Orchestrator | Diagnostic | K. Connector | Aggregator |
|---|---|---|---|---|---|---|---|---|
| Batch Iterator | | | | | X | | | |
| Snowball Expansion | | | | | X | | | |
| Coverage Exit | | | | | X | | | |
| Parallel Fan-Out | | | | | X | | | X |
| Sequential Gate | X | | | | X | | | |
| Accumulator | | | | | X | | | X |
| Session State Cache | | | | | | | X | |
| Locked Plan Artifact | X | | | | X | | | |
| Rate Limit Registry | | | | | X | | | |
| Fixed Output Template | X | | | X | X | X | | X |
| Index-Based Routing | | X | | | | | | X |
| Decision Table Lookup | | | X | X | | X | X | |
| Quantification Gate | | | | | | X | | |
| Post-Filter | | | | | | | | X |
| Correct/Incorrect Code Pair | | X | X | | | | | |
| Mandatory Subagent | | | | | X | X | | |
| Devil's Advocate Loop | X | | | | | X | | |
| One-Question Dialogue | X | | | | X | | | |
| Incremental Draft | X | | | | | | | |
| A/B Verification | | | | | | X | | |
| Interactive Trigger | | | X | | | | X | |
| Quality Checklist | X | | X | | | X | | |
| Anti-Pattern Table | | | X | | | X | | |
| Priority Tier | | X | | | X | | | |
| Graceful Degradation | | | | | | | | X |
| Skill Context Assignment | | | | | X | | | |
| Dispatch Router | | | | X | | | | X |
| Pre-Execution Warm-Up | | | X | | | | X | X |
| Keyword Auto-Link | | | | X | | | | |
