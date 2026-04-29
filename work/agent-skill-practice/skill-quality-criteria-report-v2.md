Sources:
- obra/superpowers — skills/writing-skills/anthropic-best-practices.md (github.com/obra/superpowers)
- obra/superpowers — skills/writing-skills/persuasion-principles.md (github.com/obra/superpowers)
- obra/superpowers — skills/writing-skills/SKILL.md (github.com/obra/superpowers)
- Anthropic Agent Skills Best Practices (platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- philschmid.de/agent-skills-tips

---

**Changes from v1**

- Replaced arbitrary numerical thresholds (word counts, line counts, test prompt counts) with natural-language equivalents that convey the same intent without implying empirical precision.
- Hard runtime limits (64-character name cap, 1024-character description cap) are retained as-is since they are factual constraints, not recommendations.

---

# Skill Quality Criteria

## Conclusion

A good skill is one that Claude selects at the right moment, delivers only information Claude does not already have, and reliably produces the intended behavior. The criteria that distinguish good from bad skills fall into six MECE groups: (1) Identity & Discoverability, (2) Scope Validity, (3) Information Architecture, (4) Instruction Quality, (5) Compliance Reliability, and (6) Validation. Groups 1–4 and 6 apply to all skills. Group 5 applies only to discipline-enforcing skills.

---

## Key Findings

### 1. Identity & Discoverability

How Claude selects a skill from the full list at runtime.

**Name**

The name is always loaded into the context window alongside all other skills. It must be scannable at a glance and consistent with other skills in the collection.

| Criterion | Good | Bad |
|---|---|---|
| Form | Gerund preferred; action-oriented acceptable: processing-pdfs, process-pdfs | Generic or vague regardless of form: helper, pdf-tool, doc-utils |
| Character set | Lowercase letters, numbers, hyphens only | Spaces, parentheses, reserved words (anthropic, claude) |
| Specificity | Names the action: analyzing-spreadsheets | Generic: helper, utils, documents |
| Max length | Short enough to scan at a glance | Over 64 characters (rejected by runtime) |

**Description**

The description is the primary mechanism by which Claude decides whether to read the skill body. A bad description either fails to trigger the skill when it should or triggers it when it should not.

| Criterion | Good | Bad |
|---|---|---|
| Trigger focus | Describes triggering conditions and capability domain; omits workflow and procedure | Summarizes the skill's workflow or procedure |
| Third person | "Extracts text from PDFs. Use when..." | "I can help you extract text" |
| Specificity | Includes concrete triggers, symptoms, contexts | "Helps with documents" |
| Negative case | Implicitly or explicitly excludes unrelated tasks | So broad it fires on everything |
| Length (soft) | Concise; avoid padding | Verbose without necessity |
| Length (hard) | Under 1024 characters | Over 1024 characters (rejected by runtime) |

A description that summarizes workflow creates a shortcut: Claude follows the description instead of reading the full skill body. Observed failure: a description saying "code review between tasks" caused Claude to do one review; changing it to "Use when executing implementation plans with independent tasks" caused Claude to read and follow the full two-stage review flowchart.

---

### 2. Scope Validity

Whether the skill should exist at all.

| Criterion | Good | Bad |
|---|---|---|
| Novelty | Captures a technique that was not intuitively obvious | Documents standard practice Claude already knows |
| Reusability | Applicable across projects | One-off solution to a single specific problem |
| Generality | Pattern-level insight | Project-specific convention (belongs in CLAUDE.md) |
| Automatable? | Requires judgment | Mechanical constraint enforceable by linting/validation |
| Obsolescence | Retired when the model achieves the capability independently | Left in place indefinitely regardless of model improvement |

A skill that encodes something Claude already does reliably consumes context tokens without adding value.

---

### 3. Information Architecture

How content is organized within the skill for efficient access.

**File structure**

| Criterion | Good | Bad |
|---|---|---|
| Entry point | SKILL.md short enough to read without skimming | Single file long enough to require skimming |
| Reference depth | All reference files linked directly from SKILL.md (one level deep) | SKILL.md links to advanced.md which links to details.md (nested) |
| Large reference files | Table of contents at top of long reference files | No navigation; Claude reads head preview and misses content |
| Supporting files | Created only for heavy reference or reusable executable scripts | Created for content that fits inline |
| File names | Descriptive: form_validation_rules.md | Opaque: doc2.md, file1.md |

**Progressive disclosure**

Frontmatter (name + description) is always loaded. SKILL.md body is loaded when the skill becomes relevant. Reference files and scripts are loaded or executed only when needed. A good skill exploits this layering: large reference material does not consume context until accessed.

---

### 4. Instruction Quality

What to write in the skill body and how to write it.

**Degree of freedom**

| Situation | Appropriate freedom | Instruction form |
|---|---|---|
| Multiple valid approaches, context-dependent | High | Text-based heuristics |
| Preferred pattern exists, variation acceptable | Medium | Pseudocode or parameterized script |
| Fragile sequence, consistency critical | Low | Exact command, no parameters |

**Content rules**

| Criterion | Good | Bad |
|---|---|---|
| Baseline assumption | Only adds what Claude does not already know | Explains concepts Claude knows (what PDFs are, how pip works) |
| Directives vs. exposition | Specific instructions; code examples first | General background information; reasoning before action |
| Terminology | One term per concept used consistently throughout | Same concept called "field", "box", "element", "control" |
| Time sensitivity | "Current method" + "Old patterns" section | Date-conditional logic ("If you're doing this before August 2025...") |
| Examples | One complete, runnable example from a real scenario | Multiple mediocre examples across 5 languages |
| Flowcharts | Only for non-obvious decision points or branching loops | Linear instructions, reference material, code examples |
| Cross-references | Skill name + explicit requirement marker: "REQUIRED: Use X" | File path: @skills/testing/tdd/SKILL.md (force-loads content) |
| Options | One default approach with one escape hatch | Multiple options listed equally |
| Outcome vs. procedure | Describes desired outcome; constraints instead of steps | Step-by-step procedure for non-fragile tasks |

**Feedback loops**

For quality-critical tasks: include a validate → fix → repeat loop. For complex workflows: provide a copyable checklist Claude can check off. For scripts: plan-validate-execute pattern prevents errors in high-stakes operations.

**Token efficiency targets**

| Metric | Target |
|---|---|
| SKILL.md body length | Short enough that Claude reads the full body; split into separate files as it grows |
| Frequently loaded skill | Very brief — a handful of sentences |
| Standard skill | Brief enough to absorb in one reading |

---

### 5. Compliance Reliability (Discipline-Enforcing Skills)

Applies when a skill is intended to enforce a rule or practice that the agent might rationalize away under pressure.

**Instruction language**

| Criterion | Good | Bad |
|---|---|---|
| Rule statement | "YOU MUST. No exceptions." | "Consider doing X when feasible." |
| Loophole closures | Explicit: "Don't keep it as reference. Don't look at it. Delete means delete." | Implicit: "Delete it and start over." |
| Spirit vs. letter | State upfront: "Violating the letter is violating the spirit." | Leave unaddressed |
| Trigger pattern | "When X, do Y immediately." | "Generally do Y." |

**Principle combinations by skill type**

| Skill type | Principles to apply | Principles to avoid |
|---|---|---|
| Discipline-enforcing | Authority + Commitment + Social Proof | Liking, Reciprocity |
| Guidance / technique | Moderate Authority + Unity | Heavy authority |
| Collaborative | Unity + Commitment | Authority, Liking |
| Reference | Clarity only | All persuasion principles |

**Rationalization resistance**

A discipline-enforcing skill that survives a pressure test (time pressure + sunk cost + authority challenge combined) is more reliable than one that passes a calm scenario. Build a rationalization table from real test output: each excuse the agent produces in baseline testing becomes an explicit counter in the skill.

---

### 6. Validation

Whether the skill demonstrably produces the intended behavior.

| Criterion | Good | Bad |
|---|---|---|
| Evaluation timing | Created before writing skill content | Written after, to confirm existing content |
| Coverage | Multiple scenarios; triggering AND non-triggering cases | Happy path only |
| Volume | Enough prompts to cover trigger and non-trigger cases; run each scenario more than once | Single trial |
| Model coverage | Tested on all target models (Haiku, Sonnet, Opus) | Tested on one model only |
| Iteration signal | Behavior of Claude B on real tasks drives refinement | Intuition or re-reading the skill drives refinement |
| Fix priority | Description fixed first when activation is the problem | Body rewritten when description is the actual issue |

The standard development loop: (1) run representative task without the skill and document failures, (2) write minimal content to address those failures, (3) run with skill loaded and observe behavior, (4) refine based on observation. Skills that skip step 1 document imagined problems, not real ones.

---

## Evidence

### Description workflow trap (Source 3)
In documented testing, a skill description that summarized the workflow ("code review between tasks") caused Claude to execute only one review. The full skill body specified a two-stage process. Changing the description to triggering conditions only ("Use when executing implementation plans with independent tasks") restored full compliance. This confirms that the description is not just metadata — it actively competes with the skill body for Claude's attention.

### Persuasion research baseline (Source 2)
Meincke et al. (2025), N=28,000 LLM conversations: compliance increased from 33% to 72% (p < .001) when persuasion techniques were applied. Authority, commitment, and scarcity were most effective. Liking creates sycophancy and conflicts with honest feedback culture. This provides empirical grounding for the principle combinations in Key Finding 5.

### Token load mechanics (Source 4)
At startup only frontmatter (name + description) is loaded for all skills. SKILL.md body loads when the skill is selected. Additional files load on-demand. Scripts execute without loading into context. A large reference file costs zero tokens until actually read.

### 50% activation improvement from description alone (Source 5)
Philschmid reports 50% performance improvements observed from refining descriptions alone, with no changes to skill body content. This supports the priority in Key Finding 1: description quality is the highest-leverage single change.

### Skill existence criteria (Sources 3, 4)
Both sources converge on the same test for whether a skill should exist: did Claude fail at this task without the skill? Source 4: "Create evaluations BEFORE writing extensive documentation. Ensures your Skill solves real problems rather than documenting imagined ones." Source 3: "If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing."
