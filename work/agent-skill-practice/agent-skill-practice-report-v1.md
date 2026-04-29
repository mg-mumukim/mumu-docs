# Agent Skill Practice: Three External Repositories

Sources:
- https://github.com/garrytan/gbrain (skills/, AGENTS.md, RESOLVER.md, manifest.json; 8 SKILL.md files fully read)
- https://github.com/obra/superpowers (skills/, CLAUDE.md, README.md; 7 SKILL.md files fully read)
- https://github.com/mattpocock/skills (skills/engineering/, skills/productivity/; README.md, CLAUDE.md, CONTEXT.md; 12 SKILL.md files fully read)
- work/agent-skill-taxonomy/agent-skill-taxonomy-report-v5.md (taxonomy reference)

---

## Conclusion

All three repos confirm the existing 11-type taxonomy — no new types emerge. The combined 56 skills (gbrain 30, superpowers 14, mattpocock 12) distribute across 7 of the 11 existing types. What the three repos contribute is not new types but 12 new intra-skill structural components, raising the known component count from 44 to 56. The most significant new patterns are: ambient always-on activation (gbrain), a resolver/dispatch routing layer above the skill level (gbrain), XML-tag execution gates (obra), companion subagent prompt files bundled within a skill directory (obra), and a cross-skill domain vocabulary layer enforced by all skills (mattpocock).

Skill counts: gbrain 30, obra 14, mattpocock 12 (public buckets only). Type distribution in these three repos: Process-Enforcement 14, Action/Automation 9, Orchestrator 5, Diagnostic/Investigation 5, Meta-Skill 5, Knowledge Connector 4, Multi-Source Aggregator 2, Multi-Model Consultation 1, Rule Library 1. The three types not represented — Living Knowledge Base, Onboarding/Tutorial, and Multi-Model Consultation — appear only as a single instance (one Multi-Model Consultation in gbrain `cross-modal-review`).

---

## Key Findings

### garrytan/gbrain — Knowledge brain with ambient skill activation

gbrain is a TypeScript personal knowledge management system targeting investors and founders. Its 30 skills divide into five functional groups: ingestion (ingest, idea-ingest, media-ingest, meeting-ingestion), knowledge operations (query, brain-ops, enrich, citation-fixer), operational health (maintain, frontmatter-guard, smoke-test, skillpack-check), orchestration/scheduling (minion-orchestrator, cron-scheduler, cross-modal-review), and meta/identity (skillify, skill-creator, testing, soul-audit).

The defining philosophy is **brain-first**: every message passes through the knowledge graph before generating a response. Two skills are permanently ambient: `signal-detector` fires on every message to detect entity mentions and original thinking, and `brain-ops` is the core read/write loop for all page operations. These are not invoked — they run as background tasks alongside any response. This is the first known instance of an always-on activation model in the surveyed corpus.

gbrain uses a separate `RESOLVER.md` dispatch layer that maps natural-language trigger phrases to skills. No other surveyed repo uses a standalone routing document above the skill level. The manifest.json carries a `conformance_version: "1.0.0"` field, enabling cross-repo skill compatibility checks via `gbrain check-resolvable`.

The `skillify` meta-skill defines a 10-item completeness checklist (SKILL.md, code, unit tests, integration tests, LLM evals, resolver trigger, resolver trigger eval, check-resolvable, E2E test, brain filing) and a `scaffold` CLI command that creates stub files atomically. This is qualitatively more demanding than any skill-authoring guide in the v5 corpus. `skillify` + `check-resolvable` together form a testable CI gate on the skill tree.

The Iron Law of citation and back-linking is a cross-cutting mandatory convention embedded in every ingestion skill: every fact carries a `[Source: ...]` citation, and every entity mention creates a back-link from the entity's page to the referencing page. This is an example of domain-specific invariants encoded as skill-level contracts.

### obra/superpowers — Structured software development methodology

Superpowers is a multi-platform development skill set (Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI) providing a complete software development workflow. The 14 skills form a linear pipeline: `brainstorming` → `writing-plans` → `executing-plans` → `subagent-driven-development` / `dispatching-parallel-agents` → `requesting-code-review` → `finishing-a-development-branch`.

The defining design choice is **human supervision at every phase boundary**. The term "your human partner" appears throughout and is documented as deliberate (CLAUDE.md: "not interchangeable with 'the user'"). `brainstorming` carries a `<HARD-GATE>` XML block: "Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it." The `<HARD-GATE>` tag is a structural element stronger than the prose anti-rationalization tables seen in v5 skills — it is an explicit named gate type embedded in the skill body.

Several skills bundle companion prompt files. `subagent-driven-development` ships `implementer-prompt.md`, `spec-reviewer-prompt.md`, and `code-quality-reviewer-prompt.md` — verbatim prompts injected into dispatched subagents. `brainstorming` bundles `spec-document-reviewer-prompt.md`. These files are not referenced instructions; they are the actual prompt content forwarded to child agents. This companion-file pattern is not present in any v5 corpus repo.

Process flows are documented as embedded Graphviz DOT notation rather than prose checklists. `brainstorming`, `dispatching-parallel-agents`, and `using-superpowers` each contain a `digraph { ... }` block rendered in compatible environments.

`verification-before-completion` encodes an Iron Law: "No completion claims without fresh verification evidence." It maintains a Red Flags table (15 entries), a Rationalization Prevention table (8 entries), and a Common Failures table (7 entries). This is the most elaborate anti-rationalization apparatus in the corpus.

`using-superpowers` is an onboarding/meta skill that enforces skill invocation before any response. It includes a `<SUBAGENT-STOP>` tag that exempts dispatched subagents from the meta-skill's requirements — the first observed conditional skip tag.

The project's CLAUDE.md states a 94% PR rejection rate and explicitly prohibits skill modifications without eval evidence. Skills are documented as "code that shapes agent behavior," not prose — changes require adversarial pressure testing across multiple sessions.

### mattpocock/skills — Domain-vocabulary-driven engineering workflow

With 39.3K GitHub stars, this is the most widely installed of the three repos. Its 12 public skills (9 engineering, 3 productivity) address four stated failure modes: agent misalignment, excessive verbosity, broken code, and architectural degradation.

The defining structural innovation is a **cross-skill domain vocabulary layer**. All engineering skills (`diagnose`, `tdd`, `improve-codebase-architecture`, `grill-with-docs`, `to-issues`, `triage`, `zoom-out`) are instructed to read `CONTEXT.md` (project glossary) and `docs/adr/` (architecture decision records) before acting. This creates a skill-ecosystem-wide shared dependency not present in any v5 repo. `grill-with-docs` updates `CONTEXT.md` inline as terms are resolved, and `improve-codebase-architecture` does the same; ADRs are written when decisions are hard to reverse, surprising without context, and the result of a real trade-off. The ADR authorship criteria are explicit across multiple skills.

`diagnose` defines a 6-phase debugging loop (feedback loop → reproduce → hypothesise → instrument → fix → cleanup) with 10 ordered strategies for constructing a feedback loop (failing test, curl/HTTP script, CLI invocation, headless browser, trace replay, throwaway harness, property/fuzz loop, bisection, differential, HITL bash script). "Build the right feedback loop, and the bug is 90% fixed" is the central claim. Phase 3 requires 3–5 ranked falsifiable hypotheses before testing any of them.

`tdd` introduces **vertical slices** (tracer bullets) as explicit anti-pattern correction: write one test → pass it → repeat — as opposed to "horizontal slicing" (all tests first, then all code). It connects to a `deep-modules.md` companion document defining "module depth" as the ratio of implementation complexity to interface complexity. This is a specific interface design philosophy not present elsewhere in the corpus.

`improve-codebase-architecture` introduces the "deletion test": imagine deleting a module — if complexity vanishes, it was a pass-through. The skill defines a precise vocabulary (Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality) and prohibits using synonyms like "component," "service," or "boundary."

`triage` implements a 5-role issue state machine (needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix) with two category roles (bug, enhancement). The `ready-for-agent` vs `ready-for-human` distinction is explicit: `ready-for-agent` means "fully specified, AFK-ready — an agent can pick it up with no human context."

`caveman` is a Rule Library skill that activates a persistent token-compression mode (~75% reduction), persists across turns until explicitly disabled, and includes an Auto-Clarity Exception for security warnings, irreversible actions, and multi-step sequences.

Some skills carry `disable-model-invocation: true` in frontmatter (`grill-with-docs`, `setup-matt-pocock-skills`, `zoom-out`). This flag prevents the skill from being triggered by model-invocation-like requests. It is not present in any v5 corpus skill.

### Cross-cutting: what changes in the intra-skill component catalog

New components confirmed across the three repos:

| # | Component | Type | Repo |
|---|---|---|---|
| 45 | Always-on activation flag | Activation model | gbrain |
| 46 | Resolver/dispatch routing layer (RESOLVER.md) | Ecosystem structure | gbrain |
| 47 | Skill manifest (manifest.json) with conformance_version | Ecosystem structure | gbrain |
| 48 | `<HARD-GATE>` XML execution block | Gate mechanism | obra |
| 49 | `<SUBAGENT-STOP>` conditional skip tag | Gate mechanism | obra |
| 50 | Companion subagent prompt files (in skill directory) | Subagent delegation | obra |
| 51 | Graphviz DOT process flow diagram | Process documentation | obra |
| 52 | Cross-skill domain vocabulary layer (CONTEXT.md + ADRs) | Ecosystem structure | mattpocock |
| 53 | `disable-model-invocation` frontmatter flag | Activation model | mattpocock |
| 54 | Iron Law domain constraint (citation / back-link) | Process enforcement | gbrain |
| 55 | Skill completeness checklist (10-item, with CI gate) | Meta-skill tooling | gbrain |
| 56 | Multi-platform plugin adapter directories | Deployment | obra |

Components 45–56 are additions to the v5 catalog of 44. Total known components: 56.

---

## Evidence

### Skill-to-type mapping

#### garrytan/gbrain (30 skills)

| Skill | Type |
|---|---|
| signal-detector | Action/Automation (always-on variant) |
| brain-ops | Knowledge Connector |
| ingest | Knowledge Connector |
| idea-ingest | Knowledge Connector |
| media-ingest | Knowledge Connector |
| meeting-ingestion | Knowledge Connector |
| query | Knowledge Connector |
| enrich | Knowledge Connector |
| data-research | Multi-Source Aggregator |
| briefing | Multi-Source Aggregator |
| minion-orchestrator | Orchestrator |
| cross-modal-review | Multi-Model Consultation |
| daily-task-manager | Action/Automation |
| daily-task-prep | Action/Automation |
| cron-scheduler | Action/Automation |
| reports | Action/Automation |
| publish | Action/Automation |
| webhook-transforms | Action/Automation |
| citation-fixer | Action/Automation |
| maintain | Process-Enforcement |
| frontmatter-guard | Process-Enforcement |
| soul-audit | Process-Enforcement |
| setup | Onboarding/Tutorial |
| migrate | Action/Automation |
| repo-architecture | Rule Library |
| skillify | Meta-Skill |
| skill-creator | Meta-Skill |
| testing | Meta-Skill |
| skillpack-check | Meta-Skill |
| smoke-test | Action/Automation |

#### obra/superpowers (14 skills)

| Skill | Type |
|---|---|
| brainstorming | Process-Enforcement |
| writing-plans | Process-Enforcement |
| executing-plans | Action/Automation |
| subagent-driven-development | Orchestrator |
| dispatching-parallel-agents | Orchestrator |
| test-driven-development | Process-Enforcement |
| systematic-debugging | Diagnostic/Investigation |
| requesting-code-review | Process-Enforcement |
| receiving-code-review | Process-Enforcement |
| using-git-worktrees | Action/Automation |
| finishing-a-development-branch | Process-Enforcement |
| verification-before-completion | Process-Enforcement |
| writing-skills | Meta-Skill |
| using-superpowers | Meta-Skill |

#### mattpocock/skills (12 public skills)

| Skill | Type |
|---|---|
| diagnose | Diagnostic/Investigation |
| grill-with-docs | Process-Enforcement |
| improve-codebase-architecture | Diagnostic/Investigation |
| setup-matt-pocock-skills | Onboarding/Tutorial |
| tdd | Process-Enforcement |
| to-issues | Action/Automation |
| to-prd | Action/Automation |
| triage | Action/Automation |
| zoom-out | Diagnostic/Investigation |
| caveman | Rule Library |
| grill-me | Process-Enforcement |
| write-a-skill | Meta-Skill |

### Type distribution across all three repos

| Type | Count | Repos |
|---|---|---|
| Process-Enforcement | 14 | gbrain 3, obra 7, mattpocock 4 |
| Action/Automation | 11 | gbrain 9, obra 2, mattpocock 3 |
| Meta-Skill | 6 | gbrain 4, obra 2, mattpocock 1 |
| Orchestrator | 5 | gbrain 1, obra 2 |
| Diagnostic/Investigation | 5 | obra 1, mattpocock 4 |
| Knowledge Connector | 6 | gbrain 6 |
| Multi-Source Aggregator | 2 | gbrain 2 |
| Rule Library | 2 | gbrain 1, mattpocock 1 |
| Onboarding/Tutorial | 2 | gbrain 1, mattpocock 1 |
| Multi-Model Consultation | 1 | gbrain 1 |
| Living Knowledge Base | 0 | — |

All 56 skills map to existing 11 types. No new types added.

### Frontmatter field inventory (new fields not in v5)

| Field | Seen in | Meaning |
|---|---|---|
| triggers[] | gbrain | List of exact user phrases that route to this skill (feeds RESOLVER.md) |
| tools[] | gbrain | Allowlisted MCP/gbrain tool names the skill is permitted to call |
| mutating | gbrain | Boolean; true if skill writes brain pages |
| writes_pages | gbrain | Boolean; true if skill creates new pages |
| writes_to[] | gbrain | List of brain directory prefixes the skill is allowed to write to |
| version | gbrain | Semantic version of the skill |
| disable-model-invocation | mattpocock | true = skill is not invoked via model invocation |

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
