# Agent Skill Practice: Three External Repositories

Sources:
- https://github.com/garrytan/gbrain (skills/, AGENTS.md, RESOLVER.md, manifest.json; 8 SKILL.md files fully read)
- https://github.com/obra/superpowers (skills/, CLAUDE.md, README.md; 7 SKILL.md files fully read)
- https://github.com/mattpocock/skills (skills/engineering/, skills/productivity/; README.md, CLAUDE.md, CONTEXT.md; 12 SKILL.md files fully read)
- work/agent-skill-taxonomy/agent-skill-taxonomy-report-v5.md (taxonomy reference: type definitions, component catalog, structural dimensions)

---

## Changes from v1

- Added Type Taxonomy section with the 11-type/4-category table from v5, so the document is self-contained without requiring v5 as a reference
- Added Structural Dimensions section and applied corpus size / interaction model / persistence model to all 56 new skills
- Reorganized new components 45–56 into v5's 6 component groups (Control Flow, State Management, Data Routing, Interaction Patterns, Quality Enforcement, Composition Patterns)
- Extended Appendix C: cumulative type-count table now includes gbrain, obra, and mattpocock; component cross-reference matrix extended with rows 45–56
- Extended frontmatter field inventory with 7 new fields

---

## Conclusion

All 56 skills across the three repos map to the existing 11-type taxonomy — no new types emerge. The taxonomy is now validated across 11 vendor ecosystems (v5: 8 repos; this report: +3). The three repos contribute 12 new intra-skill structural components (components 45–56), raising the catalog total from 44 to 56 and extending all 6 component groups.

The most significant structural advances over the v5 corpus are: always-on ambient activation (gbrain, component 45), a resolver/dispatch routing layer above the skill level (gbrain, component 46), `<HARD-GATE>` XML execution blocks as a named gate type (obra, component 48), companion subagent prompt files bundled within skill directories (obra, component 50), and a cross-skill domain vocabulary layer (CONTEXT.md + ADRs) enforced by all engineering skills at invocation time (mattpocock, component 52).

Skill counts: gbrain 30, obra 14, mattpocock 12 (public buckets only); total 56. Cumulative corpus across 11 repos: ~290 skills. Process-Enforcement remains the most common type (gbrain 3, obra 7, mattpocock 4 = 14 new instances). Action/Automation leads cumulatively.

---

## Universal Structure

Every skill across all repos contains three universal components (from v5):

1. **Frontmatter** — `name` (kebab-case slug) and `description` (trigger-focused). The three new repos add 7 new optional fields: `triggers[]`, `tools[]`, `mutating`, `writes_pages`, `writes_to[]`, `version` (gbrain); `disable-model-invocation` (mattpocock).

2. **Activation conditions** — when the skill fires. Appears in frontmatter `description`, a "When to Use / When NOT to Use" body section, or both. New variant in gbrain: always-on activation (no trigger required; skill fires on every message).

3. **Process body** — a sequential or structured set of steps. New variant in obra: embedded Graphviz DOT diagrams as process documentation.

---

## Type Taxonomy

11 types in 4 functional categories (from v5). All 56 new skills map to existing types.

### A. Knowledge

| Type | What it does | What distinguishes it |
|---|---|---|
| Rule Library | Encodes domain rules as a prioritized catalog | Rules stored in separate files ranked CRITICAL to LOW; SKILL.md is an index |
| Living Knowledge Base | Encodes all domain rules in one large file | Wrong code shown first with failure mode annotation, then correct code |

### B. Execution

| Type | What it does | What distinguishes it |
|---|---|---|
| Action/Automation | Automates a discrete operational task | State detection block, decision tree, environment-specific branching |
| Knowledge Connector | Wraps an authenticated internal service | Session state, API endpoint table, error code table, mandatory precondition |
| Diagnostic/Investigation | Systematic root cause analysis | Confirms failure reproduction before forming a hypothesis |
| Process-Enforcement | Encodes a multi-step practice with human approval gates | Anti-rationalization table, red flags list, verification checklist prevent shortcuts |

### C. Coordination

| Type | What it does | What distinguishes it |
|---|---|---|
| Orchestrator | Spawns and coordinates named subagents | Four sub-variants: static team, batch pipeline, template-instantiated, domain-specialized |
| Multi-Source Aggregator | Fans out to N sources in parallel, synthesizes one output | Parallel Fan-Out, post-filter by secondary criterion, Fixed Output Template |
| Multi-Model Consultation | Wraps an external AI CLI for adversarial analysis | Multi-turn interrogation protocol; `allowed-tools` sandboxed to target CLI only |

### D. System

| Type | What it does | What distinguishes it |
|---|---|---|
| Onboarding/Tutorial | Progressive skill adoption with cross-session progress tracking | Persistent progress file, sub-skill orchestration, contextual authoring nudge |
| Meta-Skill | Creates, validates, and registers other skills | Guardrails checklist, progressive disclosure doctrine, degrees-of-freedom matching |

### Type distribution — three new repos

| Type | gbrain | obra | mattpocock | Subtotal |
|---|---|---|---|---|
| Process-Enforcement | 3 | 7 | 4 | 14 |
| Action/Automation | 9 | 2 | 3 | 14 |
| Knowledge Connector | 6 | 0 | 0 | 6 |
| Meta-Skill | 4 | 2 | 1 | 7 |
| Orchestrator | 1 | 2 | 0 | 3 |
| Diagnostic/Investigation | 0 | 1 | 4 | 5 |
| Multi-Source Aggregator | 2 | 0 | 0 | 2 |
| Rule Library | 1 | 0 | 1 | 2 |
| Onboarding/Tutorial | 1 | 0 | 1 | 2 |
| Multi-Model Consultation | 1 | 0 | 0 | 1 |
| Living Knowledge Base | 0 | 0 | 0 | 0 |

---

## Structural Dimensions

Three dimensions from v5 differentiate skill complexity:

| Dimension | Values |
|---|---|
| Corpus size | Single-item / Bounded-set / Large-corpus |
| Interaction model | Autonomous / Interactive / Headless |
| Persistence model | Stateless / Session-stateful / Cross-session-stateful |

### garrytan/gbrain — dimension profile

The dominant pattern is **Large-corpus × Autonomous × Stateless**: skills that scan or write to a large, growing knowledge graph without requiring per-run user interaction. The two ambient skills (`signal-detector`, `brain-ops`) add a fourth activation mode not present in v5: **always-on** (not user-triggered).

`minion-orchestrator` is the only Cross-session-stateful skill: jobs persist in a Postgres queue across agent restarts.

| Skill | Corpus | Interaction | Persistence |
|---|---|---|---|
| signal-detector | Large-corpus | Autonomous | Stateless |
| brain-ops | Large-corpus | Autonomous | Stateless |
| ingest | Large-corpus | Autonomous | Stateless |
| idea-ingest | Large-corpus | Autonomous | Stateless |
| media-ingest | Large-corpus | Autonomous | Stateless |
| meeting-ingestion | Large-corpus | Autonomous | Stateless |
| query | Large-corpus | Autonomous | Stateless |
| enrich | Large-corpus | Autonomous | Stateless |
| data-research | Large-corpus | Autonomous | Stateless |
| briefing | Bounded-set | Autonomous | Stateless |
| minion-orchestrator | Bounded-set | Interactive | Cross-session-stateful |
| cross-modal-review | Single-item | Autonomous | Stateless |
| daily-task-manager | Bounded-set | Interactive | Session-stateful |
| daily-task-prep | Bounded-set | Autonomous | Stateless |
| cron-scheduler | Bounded-set | Autonomous | Stateless |
| reports | Bounded-set | Autonomous | Stateless |
| publish | Single-item | Autonomous | Stateless |
| webhook-transforms | Single-item | Autonomous | Stateless |
| citation-fixer | Large-corpus | Autonomous | Stateless |
| maintain | Large-corpus | Autonomous | Stateless |
| frontmatter-guard | Large-corpus | Autonomous | Stateless |
| soul-audit | Single-item | Interactive | Session-stateful |
| setup | Single-item | Interactive | Stateless |
| migrate | Large-corpus | Autonomous | Stateless |
| repo-architecture | Bounded-set | Autonomous | Stateless |
| skillify | Bounded-set | Interactive | Stateless |
| skill-creator | Single-item | Interactive | Stateless |
| testing | Bounded-set | Autonomous | Stateless |
| skillpack-check | Bounded-set | Autonomous | Stateless |
| smoke-test | Bounded-set | Autonomous | Stateless |

### obra/superpowers — dimension profile

All skills are **Single-item × Interactive × Session-stateful**: each addresses one development task, requires ongoing human approval, and produces a persistent artifact (design doc, plan, git commit). The pipeline structure means Session-stateful output from one skill becomes input to the next.

Exception: `dispatching-parallel-agents` and `verification-before-completion` are Autonomous × Stateless — they run within a session but do not require dialogue.

| Skill | Corpus | Interaction | Persistence |
|---|---|---|---|
| brainstorming | Single-item | Interactive | Session-stateful |
| writing-plans | Single-item | Interactive | Session-stateful |
| executing-plans | Bounded-set | Interactive | Session-stateful |
| subagent-driven-development | Bounded-set | Interactive | Session-stateful |
| dispatching-parallel-agents | Bounded-set | Autonomous | Stateless |
| test-driven-development | Bounded-set | Autonomous | Stateless |
| systematic-debugging | Single-item | Interactive | Stateless |
| requesting-code-review | Single-item | Interactive | Session-stateful |
| receiving-code-review | Single-item | Interactive | Session-stateful |
| using-git-worktrees | Single-item | Autonomous | Stateless |
| finishing-a-development-branch | Single-item | Interactive | Session-stateful |
| verification-before-completion | Single-item | Autonomous | Stateless |
| writing-skills | Single-item | Interactive | Session-stateful |
| using-superpowers | Bounded-set | Interactive | Stateless |

### mattpocock/skills — dimension profile

Engineering skills are split between **Single-item × Interactive × Session-stateful** (grill-with-docs, improve-codebase-architecture, grill-me, diagnose) and **Bounded-set × Interactive × Stateless** (to-issues, triage). `caveman` is the only Cross-session-stateful skill: it persists across turns until explicitly disabled.

The persistent session-statefulness of `grill-with-docs` and `improve-codebase-architecture` comes from writing to CONTEXT.md and ADRs — they mutate the shared domain vocabulary layer mid-session.

| Skill | Corpus | Interaction | Persistence |
|---|---|---|---|
| diagnose | Single-item | Interactive | Stateless |
| grill-with-docs | Single-item | Interactive | Session-stateful |
| improve-codebase-architecture | Bounded-set | Interactive | Session-stateful |
| setup-matt-pocock-skills | Single-item | Interactive | Stateless |
| tdd | Bounded-set | Autonomous | Stateless |
| to-issues | Bounded-set | Interactive | Stateless |
| to-prd | Single-item | Autonomous | Stateless |
| triage | Bounded-set | Interactive | Stateless |
| zoom-out | Single-item | Autonomous | Stateless |
| caveman | Single-item | Interactive | Cross-session-stateful |
| grill-me | Single-item | Interactive | Stateless |
| write-a-skill | Single-item | Interactive | Stateless |

---

## Key Findings

### garrytan/gbrain — Knowledge brain with ambient skill activation

gbrain is a TypeScript personal knowledge management system targeting investors and founders. Its 30 skills divide into five functional groups: ingestion (ingest, idea-ingest, media-ingest, meeting-ingestion), knowledge operations (query, brain-ops, enrich, citation-fixer), operational health (maintain, frontmatter-guard, smoke-test, skillpack-check), orchestration/scheduling (minion-orchestrator, cron-scheduler, cross-modal-review), and meta/identity (skillify, skill-creator, testing, soul-audit).

The defining philosophy is **brain-first**: every message passes through the knowledge graph before generating a response. Two skills are permanently ambient: `signal-detector` fires on every message to detect entity mentions and original thinking, and `brain-ops` is the core read/write loop for all page operations. These are not invoked — they run as background tasks alongside any response. This is the first instance of an always-on activation model (component 45) in the surveyed corpus; all 233+ skills in v5 use either description-triggered or user-invoked activation.

gbrain uses a separate `RESOLVER.md` dispatch layer (component 46) that maps natural-language trigger phrases to skills. No v5 repo uses a standalone routing document above the skill level; the closest is mgai's `skill-router` Meta-Skill, which routes at the skill level rather than the ecosystem level. The manifest.json carries a `conformance_version: "1.0.0"` field, enabling cross-repo skill compatibility checks via `gbrain check-resolvable`.

The `skillify` meta-skill defines a 10-item completeness checklist (component 55) with a `scaffold` CLI command that creates stub files atomically. This is the most operationalized skill-quality gate in the corpus: v5's Meta-Skill type already defines a Guardrails Checklist (component 21), but gbrain's version adds unit tests, integration tests, LLM evals, and E2E tests as required artifacts, and pairs it with a CI command that validates the entire skill tree for reachability, MECE overlap, and DRY violations.

The Iron Law of citation and back-linking (component 54) is a cross-cutting mandatory convention embedded in every ingestion skill: every fact carries a `[Source: ...]` citation, and every entity mention creates a back-link from the entity's page to the referencing page. This is a Process-Enforcement pattern encoded at the Knowledge Connector level — combining enforcement (component 54) with connector semantics — not present in v5.

### obra/superpowers — Structured software development methodology

Superpowers is a multi-platform development skill set (Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI) providing a complete software development workflow. The 14 skills form a linear pipeline: `brainstorming` → `writing-plans` → `executing-plans` → `subagent-driven-development` / `dispatching-parallel-agents` → `requesting-code-review` → `finishing-a-development-branch`.

The defining design choice is **human supervision at every phase boundary**. The term "your human partner" appears throughout and is documented as deliberate (CLAUDE.md: "not interchangeable with 'the user'"). `brainstorming` carries a `<HARD-GATE>` XML block (component 48): "Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it." The `<HARD-GATE>` tag is structurally distinct from v5's Sequential Gate (component 5), which is stated in prose. Component 48 is an explicitly named XML element, making it machine-inspectable as a named gate type.

Several skills bundle companion prompt files (component 50). `subagent-driven-development` ships `implementer-prompt.md`, `spec-reviewer-prompt.md`, and `code-quality-reviewer-prompt.md` — verbatim prompts injected into dispatched subagents. `brainstorming` bundles `spec-document-reviewer-prompt.md`. In v5's Orchestrator type, the closest pattern is `references/agents.md` in mgai weekly-report, which also holds spawn prompts, but superpowers' companion files are distinct per subagent role, not a single combined file. This also differs from Mandatory Subagent Invocation (component 15) and Multi-Agent Team Delegation (component 33) in that the prompt content is authored separately and evolved independently.

Process flows are documented as embedded Graphviz DOT notation (component 51) rather than prose checklists. `brainstorming`, `dispatching-parallel-agents`, and `using-superpowers` each contain a `digraph { ... }` block. This is a new composition pattern not present in any v5 skill.

`verification-before-completion` encodes an Iron Law: "No completion claims without fresh verification evidence." It maintains a Red Flags table (15 entries), a Rationalization Prevention table (8 entries), and a Common Failures table (7 entries). This is the most elaborate anti-rationalization apparatus (component 22) in the corpus. By the v5 definition, `verification-before-completion` is a Process-Enforcement skill; its anti-rationalization components are the same in type but more extensive in scope.

`using-superpowers` includes a `<SUBAGENT-STOP>` tag (component 49) that exempts dispatched subagents from the meta-skill's requirements — the first observed conditional skip tag. It also enforces skill invocation before any response, making it structurally an enforcer of the Pre-Execution Warm-Up (component 28) for all other skills.

The project's CLAUDE.md states a 94% PR rejection rate and explicitly prohibits skill modifications without eval evidence. Skills are documented as "code that shapes agent behavior," not prose — changes require adversarial pressure testing across multiple sessions.

### mattpocock/skills — Domain-vocabulary-driven engineering workflow

With 39.3K GitHub stars, this is the most widely installed of the three repos. Its 12 public skills (9 engineering, 3 productivity) address four stated failure modes: agent misalignment, excessive verbosity, broken code, and architectural degradation.

The defining structural innovation is a **cross-skill domain vocabulary layer** (component 52). All engineering skills are instructed to read `CONTEXT.md` (project glossary) and `docs/adr/` (architecture decision records) before acting. This creates a skill-ecosystem-wide shared dependency: `grill-with-docs` and `improve-codebase-architecture` update `CONTEXT.md` inline as terms are resolved, making the domain model a living artifact that grows through normal skill use. In v5 terms, this is a Composition Pattern at the ecosystem level, analogous to Index-Based Routing (component 11) but applied to a domain vocabulary rather than a data index. No v5 repo uses a cross-skill shared state document in this way.

`diagnose` defines a 6-phase debugging loop with 10 ordered strategies for constructing a feedback loop. By v5's Diagnostic/Investigation type definition, it confirms failure reproduction before forming a hypothesis (component 4 requirement met). Phase 3 adds a non-v5 element: 3–5 ranked falsifiable hypotheses must be presented to the user before testing any of them — a checkpoint not required in v5's Diagnostic type. This is closest to One-Question-at-a-Time Dialogue (component 17) but applied to hypothesis ranking rather than clarification questions.

`tdd` introduces **vertical slices** (tracer bullets) as an explicit anti-pattern correction: write one test → pass it → repeat, as opposed to "horizontal slicing." The companion file `deep-modules.md` defines "module depth" as the ratio of implementation complexity to interface complexity. `improve-codebase-architecture` extends this with the "deletion test" and a locked vocabulary (Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality). These constitute a domain-specific quality vocabulary not present in v5's quality enforcement components.

`triage` implements a 5-role issue state machine (needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix) with an explicit `ready-for-agent` vs `ready-for-human` distinction. This is a new sub-pattern of Action/Automation: in v5, the triage function was not enumerated; here it is the primary skill purpose. The `ready-for-agent` definition ("fully specified, AFK-ready — an agent can pick it up with no human context") operationalizes the distinction between Interactive and Autonomous skills at the workflow level.

`caveman` is a Rule Library skill that persists across turns until explicitly disabled. This is the first Rule Library skill with Cross-session-stateful persistence in the corpus.

Some skills carry `disable-model-invocation: true` in frontmatter (component 53: `grill-with-docs`, `setup-matt-pocock-skills`, `zoom-out`). This flag prevents the skill from being triggered by model-invocation-like requests and is not present in any v5 frontmatter.

### Cross-cutting: new components added to the catalog

Components 45–56 extend the 44-component v5 catalog. Organized by group:

#### Group 1: Control Flow (adds 2)

**45. Always-On Activation** — The skill fires on every message without requiring a trigger phrase or user invocation. Runs alongside the primary response; does not block it. The skill's frontmatter may omit `description`-based trigger phrases entirely.
- gbrain: `signal-detector` (entity detection on every message), `brain-ops` (read/write on any brain operation)

**53. `disable-model-invocation` Flag** — A frontmatter boolean that prevents the skill from activating when the user's input resembles a model invocation. Used for skills whose process begins with codebase exploration rather than model generation.
- mattpocock: `grill-with-docs`, `setup-matt-pocock-skills`, `zoom-out`

#### Group 4: Interaction Patterns (adds 2)

**48. `<HARD-GATE>` XML Execution Block** — An explicitly named XML element placed at the top of the skill body that prohibits proceeding to implementation until a named condition is met. Stronger than the prose-based Sequential Gate (component 5): the tag name makes it machine-inspectable and visually distinct. Always paired with a named condition (user approval, design review, etc.).
- obra: `brainstorming` (gate: "do not write code until design is approved")

**49. `<SUBAGENT-STOP>` Conditional Skip Tag** — An XML tag at the skill entry point that instructs the skill to exit immediately when the agent was dispatched as a subagent. Prevents meta-skill or onboarding logic from executing in subagent context where it would be disruptive.
- obra: `using-superpowers`

#### Group 5: Quality Enforcement (adds 2)

**54. Iron Law Domain Constraint** — A named mandatory invariant embedded in skill contracts that applies across every invocation of a class of skills, regardless of content or context. Not a per-request checklist but a universally enforced rule that the agent must never violate (e.g., "every entity mention creates a back-link"). Named "Iron Law" explicitly in source.
- gbrain: `ingest`, `idea-ingest`, `media-ingest`, `meeting-ingestion` (Iron Law: every entity mention creates a back-link; every fact carries a citation)

**55. Skill Completeness Checklist with CI Gate** — A 10-item checklist that defines when a skill is "properly skilled": SKILL.md, code, unit tests, integration tests, LLM evals, resolver trigger, resolver trigger eval, check-resolvable, E2E test, brain filing. Paired with a CLI command (`gbrain check-resolvable`) that validates the entire skill tree for reachability, MECE overlap, and DRY violations. Fails CI until all stubs are replaced.
- gbrain: `skillify`

#### Group 6: Composition Patterns (adds 6)

**46. Resolver/Dispatch Routing Layer** — A standalone document (`RESOLVER.md`) outside individual skill files that maps natural-language trigger phrases to skills. Acts as the ecosystem-level dispatcher: agents read it before any task to determine which skill to invoke. Distinct from Dispatch Router (component 27), which is a pattern inside a single skill; the resolver is above the skill level. Paired with `check-resolvable` for MECE validation.
- gbrain: `RESOLVER.md`

**47. Skill Manifest with Conformance Version** — A `manifest.json` file at the skills root that lists all skills with paths, descriptions, and a `conformance_version` field. Enables cross-repo compatibility checks and ecosystem-level validation. Analogous to a package.json for the skill tree.
- gbrain: `skills/manifest.json`

**50. Companion Subagent Prompt Files** — Separate `.md` files bundled within a skill directory that contain verbatim agent prompts injected into dispatched subagents. Each file corresponds to a distinct subagent role (implementer, spec reviewer, code reviewer). Unlike `references/agents.md` in v5's mgai weekly-report (a single file with all spawn prompts), companion files are one-per-role, versioned and edited independently.
- obra: `subagent-driven-development` (implementer-prompt.md, spec-reviewer-prompt.md, code-quality-reviewer-prompt.md), `brainstorming` (spec-document-reviewer-prompt.md)

**51. Graphviz DOT Process Flow Diagram** — An embedded `digraph { ... }` block within the SKILL.md body that renders as a directed graph in compatible environments. Used to document decision trees and phase sequences visually. Complements or replaces prose checklists for complex multi-branch workflows.
- obra: `brainstorming`, `dispatching-parallel-agents`, `using-superpowers`

**52. Cross-Skill Domain Vocabulary Layer** — A shared `CONTEXT.md` (project glossary) and `docs/adr/` (architecture decision records) that all engineering skills are required to read before acting. Terms defined in CONTEXT.md constrain the vocabulary used in test names, issue titles, and architectural proposals. CONTEXT.md is mutated inline by `grill-with-docs` and `improve-codebase-architecture` as terms are resolved. ADRs record hard-to-reverse decisions to prevent re-litigation.
- mattpocock: read by `diagnose`, `tdd`, `improve-codebase-architecture`, `grill-with-docs`, `to-issues`, `triage`, `zoom-out`; mutated by `grill-with-docs`, `improve-codebase-architecture`

**56. Multi-Platform Plugin Adapter Directories** — A set of platform-specific adapter directories (`.claude-plugin/`, `.codex-plugin/`, `.codex/`, `.cursor-plugin/`, `.opencode/`) that translate the core skill content for each target agent runtime. Enables a single skill repository to be installed on 5+ platforms from one source.
- obra: root of repo

---

## Evidence

### Skill-to-type mapping

#### garrytan/gbrain (30 skills)

| Skill | Type | Corpus | Interaction | Persistence |
|---|---|---|---|---|
| signal-detector | Action/Automation (always-on) | Large-corpus | Autonomous | Stateless |
| brain-ops | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| ingest | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| idea-ingest | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| media-ingest | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| meeting-ingestion | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| query | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| enrich | Knowledge Connector | Large-corpus | Autonomous | Stateless |
| data-research | Multi-Source Aggregator | Large-corpus | Autonomous | Stateless |
| briefing | Multi-Source Aggregator | Bounded-set | Autonomous | Stateless |
| minion-orchestrator | Orchestrator | Bounded-set | Interactive | Cross-session-stateful |
| cross-modal-review | Multi-Model Consultation | Single-item | Autonomous | Stateless |
| daily-task-manager | Action/Automation | Bounded-set | Interactive | Session-stateful |
| daily-task-prep | Action/Automation | Bounded-set | Autonomous | Stateless |
| cron-scheduler | Action/Automation | Bounded-set | Autonomous | Stateless |
| reports | Action/Automation | Bounded-set | Autonomous | Stateless |
| publish | Action/Automation | Single-item | Autonomous | Stateless |
| webhook-transforms | Action/Automation | Single-item | Autonomous | Stateless |
| citation-fixer | Action/Automation | Large-corpus | Autonomous | Stateless |
| maintain | Process-Enforcement | Large-corpus | Autonomous | Stateless |
| frontmatter-guard | Process-Enforcement | Large-corpus | Autonomous | Stateless |
| soul-audit | Process-Enforcement | Single-item | Interactive | Session-stateful |
| setup | Onboarding/Tutorial | Single-item | Interactive | Stateless |
| migrate | Action/Automation | Large-corpus | Autonomous | Stateless |
| repo-architecture | Rule Library | Bounded-set | Autonomous | Stateless |
| skillify | Meta-Skill | Bounded-set | Interactive | Stateless |
| skill-creator | Meta-Skill | Single-item | Interactive | Stateless |
| testing | Meta-Skill | Bounded-set | Autonomous | Stateless |
| skillpack-check | Meta-Skill | Bounded-set | Autonomous | Stateless |
| smoke-test | Action/Automation | Bounded-set | Autonomous | Stateless |

#### obra/superpowers (14 skills)

| Skill | Type | Corpus | Interaction | Persistence |
|---|---|---|---|---|
| brainstorming | Process-Enforcement | Single-item | Interactive | Session-stateful |
| writing-plans | Process-Enforcement | Single-item | Interactive | Session-stateful |
| executing-plans | Action/Automation | Bounded-set | Interactive | Session-stateful |
| subagent-driven-development | Orchestrator | Bounded-set | Interactive | Session-stateful |
| dispatching-parallel-agents | Orchestrator | Bounded-set | Autonomous | Stateless |
| test-driven-development | Process-Enforcement | Bounded-set | Autonomous | Stateless |
| systematic-debugging | Diagnostic/Investigation | Single-item | Interactive | Stateless |
| requesting-code-review | Process-Enforcement | Single-item | Interactive | Session-stateful |
| receiving-code-review | Process-Enforcement | Single-item | Interactive | Session-stateful |
| using-git-worktrees | Action/Automation | Single-item | Autonomous | Stateless |
| finishing-a-development-branch | Process-Enforcement | Single-item | Interactive | Session-stateful |
| verification-before-completion | Process-Enforcement | Single-item | Autonomous | Stateless |
| writing-skills | Meta-Skill | Single-item | Interactive | Session-stateful |
| using-superpowers | Meta-Skill | Bounded-set | Interactive | Stateless |

#### mattpocock/skills (12 public skills)

| Skill | Type | Corpus | Interaction | Persistence |
|---|---|---|---|---|
| diagnose | Diagnostic/Investigation | Single-item | Interactive | Stateless |
| grill-with-docs | Process-Enforcement | Single-item | Interactive | Session-stateful |
| improve-codebase-architecture | Diagnostic/Investigation | Bounded-set | Interactive | Session-stateful |
| setup-matt-pocock-skills | Onboarding/Tutorial | Single-item | Interactive | Stateless |
| tdd | Process-Enforcement | Bounded-set | Autonomous | Stateless |
| to-issues | Action/Automation | Bounded-set | Interactive | Stateless |
| to-prd | Action/Automation | Single-item | Autonomous | Stateless |
| triage | Action/Automation | Bounded-set | Interactive | Stateless |
| zoom-out | Diagnostic/Investigation | Single-item | Autonomous | Stateless |
| caveman | Rule Library | Single-item | Interactive | Cross-session-stateful |
| grill-me | Process-Enforcement | Single-item | Interactive | Stateless |
| write-a-skill | Meta-Skill | Single-item | Interactive | Stateless |

### Cumulative type counts (all 11 repos)

| Type | v5 total | gbrain | obra | mattpocock | Cumulative |
|---|---|---|---|---|---|
| Action/Automation | ~95 | 9 | 2 | 3 | ~109 |
| Process-Enforcement | ~50 | 3 | 7 | 4 | ~64 |
| Knowledge Connector | ~22 | 6 | 0 | 0 | ~28 |
| Rule Library | ~17 | 1 | 0 | 1 | ~19 |
| Diagnostic/Investigation | ~16 | 0 | 1 | 4 | ~21 |
| Meta-Skill | ~8 | 4 | 2 | 1 | ~15 |
| Multi-Source Aggregator | ~8 | 2 | 0 | 0 | ~10 |
| Living Knowledge Base | ~7 | 0 | 0 | 0 | ~7 |
| Orchestrator | ~9 | 1 | 2 | 0 | ~12 |
| Onboarding/Tutorial | 3 | 1 | 0 | 1 | 5 |
| Multi-Model Consultation | 3 | 1 | 0 | 0 | 4 |

v5 totals are approximate for types with tilde (~). gbrain/obra/mattpocock counts are exact.

### Intra-skill component cross-reference (new components 45–56)

| Component | Process-E | Rule Lib | Living KB | Action | Orchestrator | Diagnostic | K. Connector | Aggregator | Multi-Model | Onboarding | Meta |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 45. Always-On Activation | | | | X | | | X | | | | |
| 46. Resolver/Dispatch Layer | | | | | | | | | | | |
| 47. Skill Manifest | | | | | | | | | | | X |
| 48. HARD-GATE XML Block | X | | | | | | | | | | |
| 49. SUBAGENT-STOP Tag | | | | | | | | | | | X |
| 50. Companion Subagent Prompts | | | | | X | | | | | | |
| 51. Graphviz DOT Diagram | X | | | | X | | | | | | X |
| 52. Cross-Skill Domain Layer | X | | | X | | X | | | | | |
| 53. disable-model-invocation | X | | | | | X | | | | X | |
| 54. Iron Law Constraint | | | | | | | X | | | | |
| 55. Skill Completeness Checklist | | | | | | | | | | | X |
| 56. Multi-Platform Plugin Dirs | | | | | | | | | | | |

Components 46 and 56 operate at the ecosystem level, not the per-skill type level; they are not categorized by type.

### Frontmatter field inventory (new fields, not in v5)

| Field | Repo | Meaning |
|---|---|---|
| triggers[] | gbrain | List of exact user phrases that route to this skill via RESOLVER.md |
| tools[] | gbrain | Allowlisted MCP/gbrain tool names the skill may call |
| mutating | gbrain | true if the skill writes brain pages |
| writes_pages | gbrain | true if the skill creates new pages |
| writes_to[] | gbrain | List of brain directory prefixes the skill may write to |
| version | gbrain | Semantic version of the skill (independent of manifest.json) |
| disable-model-invocation | mattpocock | true = skill not triggered by model-invocation-like requests |

### Notable skill size outliers

- gbrain `ingest/SKILL.md`: ~250 lines; most complex single skill in the corpus, covering 6 ingestion media types, entity detection protocol, raw source preservation, and quality rules.
- obra `brainstorming/SKILL.md`: ~200 lines + embedded DOT diagram; full design methodology encoded as a single skill.
- mattpocock `diagnose/SKILL.md`: ~150 lines; 6 phases + 10 feedback-loop construction strategies.
- mattpocock `zoom-out/SKILL.md`: 4 lines (frontmatter + one instruction sentence); the shortest functional skill in the corpus.

---

## Reference

### gbrain RESOLVER.md dispatch categories

- Always-on: signal-detector, brain-ops
- Brain operations: query, enrich, repo-architecture, citation-fixer, data-research, publish, frontmatter-guard
- Ingestion: idea-ingest, media-ingest, meeting-ingestion, ingest (generic router)
- Operational: daily-task-manager, daily-task-prep, briefing, cron-scheduler, reports, skill-creator, skillify, skillpack-check, smoke-test, cross-modal-review, testing, webhook-transforms, minion-orchestrator
- Setup/migration: setup, migrate, maintain, soul-audit
- Identity/access: ACCESS_POLICY.md, SOUL.md, USER.md, HEARTBEAT.md

### obra/superpowers pipeline sequence

brainstorming → writing-plans → executing-plans → subagent-driven-development / dispatching-parallel-agents → requesting-code-review → receiving-code-review → finishing-a-development-branch

Parallel skills (usable at any phase): test-driven-development, systematic-debugging, verification-before-completion, using-git-worktrees

### mattpocock triage state machine

Category roles: bug, enhancement
State roles: needs-triage → needs-info | ready-for-agent | ready-for-human | wontfix
ready-for-agent: "fully specified, AFK-ready, no human context needed"
ready-for-human: "same structure as agent brief, but reason explains why it cannot be delegated"

### skillify completeness checklist (gbrain)

1. SKILL.md with YAML frontmatter
2. Deterministic code (if applicable)
3. Unit tests for every branch
4. Integration tests against live endpoints
5. LLM evals (if any LLM call)
6. Resolver trigger in RESOLVER.md
7. Resolver trigger eval
8. gbrain check-resolvable passes
9. E2E test
10. brain/RESOLVER.md entry (if writes brain pages)
