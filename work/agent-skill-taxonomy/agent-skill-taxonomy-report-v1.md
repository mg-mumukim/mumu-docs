# Agent Skill Taxonomy: Common Dimensions, Types, and Core Subcomponents

Sources:
- https://github.com/addyosmani/agent-skills (skills/, README.md, AGENTS.md, CONTRIBUTING.md, docs/skill-anatomy.md)
- https://github.com/vercel-labs/agent-skills (skills/, README.md, AGENTS.md)
- https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills (all skill directories)
- note/idea/how-to-write-skill.md

---

## Conclusion

Skills across all three repositories share three universal structural dimensions: a frontmatter identity block with a trigger-focused description, explicit activation conditions, and a structured process body. Beyond these universals, five distinct skill types emerge based on operational mode: **Process-Enforcement**, **Rule Library**, **Action/Automation**, **Orchestrator**, and **Diagnostic/Investigation**. Each type has a characteristic set of subcomponents that do not appear uniformly across types. The most fundamental division is between skills that encode reusable engineering practices (Process-Enforcement, Diagnostic) versus skills that encode knowledge (Rule Library) versus skills that automate discrete tasks (Action/Automation, Orchestrator).

Critical numbers: addyosmani contributes 20 skills of the Process-Enforcement type; vercel-labs contributes 7 skills spanning Rule Library and Action/Automation; EveryInc contributes ~35+ skills spanning all types including Orchestrator.

---

## Key Findings

### Finding 1: Three dimensions are universal across all skills and repos

Every skill examined, regardless of repo or type, contains:

1. **Frontmatter identity block** — a `name` field (kebab-case slug) and a `description` field (trigger-focused, beginning with "Use when…"). vercel-labs adds `metadata.author`, `metadata.version`, and `license`. EveryInc adds `argument-hint` for user-facing argument prompts. (Observed in all 20+ SKILL.md files read.)

2. **Activation conditions** — where and how the skill fires. This appears either in the frontmatter `description`, a dedicated "When to Use / When NOT to Use" section in the body, or both. The addyosmani format always includes a dedicated body section. The vercel-labs format puts trigger phrases in the description. EveryInc uses both description and an `<auto_invoke>` block with literal trigger phrases.

3. **Process body** — a sequential or structured set of steps that the agent follows. The depth and complexity vary greatly by type, but every skill has at minimum a numbered or phased sequence.

### Finding 2: Five types can be identified by operational mode

The following taxonomy spans all three repos (see Evidence section for skill-to-type mapping):

| Type | Operational mode | Primary repo |
|---|---|---|
| Process-Enforcement | Encodes a multi-step engineering practice with quality gates | addyosmani (all 20) |
| Rule Library | Encodes a prioritized catalog of domain rules with code examples | vercel-labs (5 skills) |
| Action/Automation | Automates a discrete operational task with environment-aware branching | vercel-labs, EveryInc |
| Orchestrator | Coordinates parallel subagents across phased workflows | EveryInc |
| Diagnostic/Investigation | Systematic root cause analysis through structured hypothesis testing | addyosmani, EveryInc |

The types are not mutually exclusive by name, but each has a distinct dominant subcomponent set. ce-debug (EveryInc) is diagnostic; debugging-and-error-recovery (addyosmani) is also diagnostic but lighter. deploy-to-vercel (vercel-labs) and git-commit (EveryInc) are action/automation. ce-compound and ce-review (EveryInc) are orchestrators.

### Finding 3: The Process-Enforcement type is defined by three exclusive subcomponents

The addyosmani format explicitly names its anatomy in docs/skill-anatomy.md and README.md. Process-Enforcement skills uniquely include: (1) an anti-rationalization table (common agent excuses mapped to rebuttals), (2) a red flags list (signs the agent is deviating), and (3) a verification checklist (evidence requirements to confirm the skill completed). None of these appear in Rule Library or Action/Automation skills. They exist specifically to combat agent tendency to shortcut process steps.

### Finding 4: The Rule Library type externalizes its content into individual rule files

vercel-labs Rule Library skills use a two-level document structure. The SKILL.md is an index: it lists rule categories in priority order (CRITICAL → HIGH → MEDIUM → LOW) with short slugs per rule. The actual rule detail lives in individual `rules/<slug>.md` files. A compiled `AGENTS.md` contains all rules expanded for agent loading. A `metadata.json` sidecar carries version, author, abstract, and source references. This structure allows agents to load only the relevant rule subset (progressive disclosure) rather than the full document.

### Finding 5: The Orchestrator type introduces execution-control XML and subagent contracts

EveryInc Orchestrator skills (ce-compound, ce-review) use XML tags not present in any other type: `<parallel_tasks>`, `<sequential_tasks>`, `<critical_requirement>`, `<preconditions>`, `<auto_invoke>`. These encode execution constraints that cannot be expressed in prose reliably. Each subagent is described with an explicit output contract (what data it returns, what it must not do). Mode variants (full/lightweight, autofix/report-only/headless) are declared at the top and alter which phases run.

### Finding 6: Action/Automation skills substitute decision trees for workflows

Unlike Process-Enforcement skills, Action/Automation skills (deploy-to-vercel, git-commit) begin with a state-detection block (gather preconditions), then branch into decision trees based on detected state. vercel-labs deploy-to-vercel has four distinct execution paths depending on whether the project is linked to Vercel and whether the CLI is authenticated. EveryInc git-commit checks branch state and derives commit convention from recent git log before writing. Both types include environment-specific notes that vary behavior by runtime (Claude Code, claude.ai sandbox, Codex).

### Finding 7: Supplementary files are structured by type

Each type has a characteristic set of files outside SKILL.md:

- Process-Enforcement: `references/*.md` (shared checklists loaded on-demand)
- Rule Library: `rules/*.md`, `AGENTS.md` (compiled), `metadata.json`, `resources/` (scripts)
- Action/Automation: `resources/` (shell scripts), `scripts/`
- Orchestrator: `references/*.md` (YAML schemas, mapping files, templates), `assets/` (document templates)
- Diagnostic: `references/` (anti-patterns, investigation techniques)

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

### Core subcomponent inventory by type

**Process-Enforcement**
1. Frontmatter: `name`, `description` (trigger-focused)
2. Overview: one-paragraph statement of what the skill does and why
3. When to Use / When NOT to Use: explicit include and exclude conditions
4. Gated workflow: named phases with explicit human-approval gates between them; "do not advance until" language
5. Anti-rationalization table: two-column (common excuse → rebuttal)
6. Red flags list: observable signals that the agent is skipping steps
7. Verification checklist: evidence requirements at task end (checkboxes)
8. (Optional) Reference pointers: inline links to supplementary reference docs

**Rule Library**
1. Frontmatter: `name`, `description`, `license`, `metadata` (author, version)
2. When to Apply: task-type trigger conditions
3. Priority-ranked rule index: rules organized by category with impact level (CRITICAL → LOW), each entry is a slug + one-line description
4. Per-rule files: `rules/<slug>.md` — explanation, wrong code example, correct code example
5. Compiled document: `AGENTS.md` — all rules expanded in one file for bulk loading
6. Metadata sidecar: `metadata.json` — version, organization, date, abstract, external references

**Action/Automation**
1. Frontmatter: `name`, `description`, `argument-hint` (EveryInc)
2. State detection block: pre-condition checks run before any action
3. Decision tree: branching paths keyed on detected state (linked/unlinked, authenticated/not)
4. Step sequences: numbered commands per path, with inline shell code blocks
5. Environment-specific notes: section naming the runtime (Claude Code, claude.ai, Codex) and adjusting behavior per runtime
6. Error handling / troubleshooting: named error conditions with prescribed responses
7. Fallback paths: alternative execution path when primary fails
8. (EveryInc) Pre-populated context: `!` shell commands embedded in SKILL.md to inject live data at load time

**Orchestrator**
1. Frontmatter: `name`, `description`, `argument-hint`
2. Mode declaration: named variants (full/lightweight, autofix/report-only/headless) with behavior differences per variant
3. Subagent dispatch contracts: numbered subagent tasks, each with role description, what data to return, what they must not do, which model tier to use (frontier vs mid-tier), foreground vs background flag
4. Parallel task blocks: `<parallel_tasks>` XML — agents that run concurrently
5. Sequential task gates: `<sequential_tasks>` XML with explicit WAIT directives between phases
6. Output assembly logic: orchestrator-level step that collects subagent outputs and writes the final artifact
7. `<critical_requirement>` tags: constraints enforced above all other instructions (e.g., "only the orchestrator writes files")
8. `<preconditions>` with enforcement attribute: advisory vs blocking precondition checks
9. Cross-skill invocation: explicit calls to other named skills (`/ce:compound-refresh`, `/proof`)
10. `<auto_invoke>` block: literal trigger phrases for automatic activation

**Diagnostic/Investigation**
1. Frontmatter: `name`, `description`, `argument-hint`
2. Core principles: small set of invariant rules stated at the top and repeated at decision points (e.g., "investigate before fixing", "one change at a time")
3. Phase execution table: tabular overview of all phases (phase number, name, purpose)
4. Reproduction step: explicit instruction to confirm the bug exists before hypothesizing
5. Hypothesis formation with causal chain gate: agent must state the full causal chain (trigger → symptom, with no gaps) before proceeding; predictions required for uncertain links
6. Smart escalation table: matrix of failure patterns → diagnosis → next move; activates after N failed attempts
7. Structured close/summary template: fixed-field output (Problem, Root Cause, Fix, Prevention, Confidence)
8. Handoff options: post-diagnosis choices presented to user (fix, document, post to issue tracker)

---

## Reference (Appendix)

### Frontmatter field inventory

| Field | addyosmani | vercel-labs | EveryInc |
|---|---|---|---|
| name | yes | yes | yes |
| description | yes | yes | yes |
| metadata.author | no | yes | no |
| metadata.version | no | yes | no |
| license | no | yes | no |
| argument-hint | no | no | yes |

### File layout per type

**Process-Enforcement (addyosmani)**
```
skills/<name>/
  SKILL.md
references/
  testing-patterns.md
  security-checklist.md
  performance-checklist.md
  accessibility-checklist.md
```

**Rule Library (vercel-labs)**
```
skills/<name>/
  SKILL.md
  AGENTS.md         (compiled, all rules)
  README.md
  metadata.json
  rules/
    <slug>.md       (one file per rule)
```

**Action/Automation (vercel-labs)**
```
skills/<name>/
  SKILL.md
  resources/
    deploy.sh
    deploy-codex.sh
```

**Action/Automation (EveryInc)**
```
skills/<name>/
  SKILL.md          (includes !-prefixed shell commands for live context)
  references/       (optional)
```

**Orchestrator (EveryInc)**
```
skills/<name>/
  SKILL.md
  references/
    schema.yaml
    yaml-schema.md
  assets/
    resolution-template.md
```

**Diagnostic (EveryInc)**
```
skills/<name>/
  SKILL.md
  references/
    anti-patterns.md
    investigation-techniques.md
```

### Skill counts by type

| Type | addyosmani | vercel-labs | EveryInc | Total |
|---|---|---|---|---|
| Process-Enforcement | 19 | 0 | ~5 | ~24 |
| Rule Library | 0 | 5 | ~3 | ~8 |
| Action/Automation | 0 | 2 | ~17 | ~19 |
| Orchestrator | 0 | 0 | ~3 | ~3 |
| Diagnostic/Investigation | 1 | 0 | ~4 | ~5 |

EveryInc counts are approximate; not all ~35 skills were fully read.
