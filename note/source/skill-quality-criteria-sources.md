# Skill Quality Criteria — Source Collection

Collected: 2026-04-29
Sources:
1. obra/superpowers — skills/writing-skills/anthropic-best-practices.md
2. obra/superpowers — skills/writing-skills/persuasion-principles.md
3. obra/superpowers — skills/writing-skills/SKILL.md
4. platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
5. philschmid.de/agent-skills-tips

---

## Source 1: obra/superpowers — anthropic-best-practices.md

(This file is a mirror/adaptation of the Anthropic official best practices. Content is substantively identical to Source 4 below; see Source 4 for full text.)

---

## Source 2: obra/superpowers — persuasion-principles.md

# Persuasion Principles for Skill Design

## Overview

LLMs respond to the same persuasion principles as humans. Understanding this psychology helps you design more effective skills - not to manipulate, but to ensure critical practices are followed even under pressure.

**Research foundation:** Meincke et al. (2025) tested 7 persuasion principles with N=28,000 AI conversations. Persuasion techniques more than doubled compliance rates (33% → 72%, p < .001).

## The Seven Principles

### 1. Authority
**What it is:** Deference to expertise, credentials, or official sources.

**How it works in skills:**
- Imperative language: "YOU MUST", "Never", "Always"
- Non-negotiable framing: "No exceptions"
- Eliminates decision fatigue and rationalization

**When to use:**
- Discipline-enforcing skills (TDD, verification requirements)
- Safety-critical practices
- Established best practices

**Example:**
```
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. Commitment
**What it is:** Consistency with prior actions, statements, or public declarations.

**How it works in skills:**
- Require announcements: "Announce skill usage"
- Force explicit choices: "Choose A, B, or C"
- Use tracking: TodoWrite for checklists

**When to use:**
- Ensuring skills are actually followed
- Multi-step processes
- Accountability mechanisms

**Example:**
```
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. Scarcity
**What it is:** Urgency from time limits or limited availability.

**How it works in skills:**
- Time-bound requirements: "Before proceeding"
- Sequential dependencies: "Immediately after X"
- Prevents procrastination

**When to use:**
- Immediate verification requirements
- Time-sensitive workflows
- Preventing "I'll do it later"

**Example:**
```
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. Social Proof
**What it is:** Conformity to what others do or what's considered normal.

**How it works in skills:**
- Universal patterns: "Every time", "Always"
- Failure modes: "X without Y = failure"
- Establishes norms

**When to use:**
- Documenting universal practices
- Warning about common failures
- Reinforcing standards

**Example:**
```
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. Unity
**What it is:** Shared identity, "we-ness", in-group belonging.

**How it works in skills:**
- Collaborative language: "our codebase", "we're colleagues"
- Shared goals: "we both want quality"

**When to use:**
- Collaborative workflows
- Establishing team culture
- Non-hierarchical practices

### 6. Reciprocity
Use sparingly — can feel manipulative. Rarely needed in skills.

### 7. Liking
**DON'T USE for compliance.** Conflicts with honest feedback culture. Creates sycophancy.

## Principle Combinations by Skill Type

| Skill Type | Use | Avoid |
|------------|-----|-------|
| Discipline-enforcing | Authority + Commitment + Social Proof | Liking, Reciprocity |
| Guidance/technique | Moderate Authority + Unity | Heavy authority |
| Collaborative | Unity + Commitment | Authority, Liking |
| Reference | Clarity only | All persuasion |

## Why This Works: The Psychology

**Bright-line rules reduce rationalization:**
- "YOU MUST" removes decision fatigue
- Absolute language eliminates "is this an exception?" questions
- Explicit anti-rationalization counters close specific loopholes

**Implementation intentions create automatic behavior:**
- Clear triggers + required actions = automatic execution
- "When X, do Y" more effective than "generally do Y"
- Reduces cognitive load on compliance

**LLMs are parahuman:**
- Trained on human text containing these patterns
- Authority language precedes compliance in training data
- Commitment sequences (statement → action) frequently modeled
- Social proof patterns (everyone does X) establish norms

## Ethical Use

**Legitimate:** Ensuring critical practices are followed; creating effective documentation; preventing predictable failures.

**Illegitimate:** Manipulating for personal gain; creating false urgency; guilt-based compliance.

**The test:** Would this technique serve the user's genuine interests if they fully understood it?

## Research Citations

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania. Compliance increased 33% → 72% with persuasion techniques. Authority, commitment, scarcity most effective.

---

## Source 3: obra/superpowers — skills/writing-skills/SKILL.md (excerpts)

### What is a Skill?
A skill is a reference guide for proven techniques, patterns, or tools. **Skills are NOT narratives about how you solved a problem once.**

Skill types: Technique (concrete method with steps), Pattern (way of thinking about problems), Reference (API docs, syntax guides).

### SKILL.md Frontmatter
- `name`: letters, numbers, hyphens only; no parentheses, special chars; max 1024 chars total
- `description`: Third-person; describes ONLY when to use (NOT what it does); start with "Use when..."; NEVER summarize the skill's process or workflow; keep under 500 chars if possible

**Critical finding on description:** Testing revealed that when a description summarizes the skill's workflow, Claude may follow the description instead of reading the full skill content. A description saying "code review between tasks" caused Claude to do ONE review, even though the skill's flowchart clearly showed TWO reviews. When changed to just "Use when executing implementation plans with independent tasks" (no workflow summary), Claude correctly read and followed the full content.

```yaml
# ❌ BAD: Summarizes workflow
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ✅ GOOD: Just triggering conditions
description: Use when executing implementation plans with independent tasks in the current session
```

### Claude Search Optimization (CSO)
- Rich description field: answer "Should I read this skill right now?"
- Keyword coverage: error messages, symptoms, synonyms, tool names
- Descriptive naming: verb-first, active voice (creating-skills not skill-creation)
- Token efficiency targets: frequently-loaded <150 words, others <500 words

### Token Efficiency Techniques
- Move details to tool help (`--help` reference instead of documenting all flags)
- Use cross-references instead of repeating workflow details
- Compress examples (20 words vs 42 words for same pattern)
- Eliminate redundancy

### Cross-Referencing Other Skills
- Use skill name only with explicit requirement markers: `**REQUIRED SUB-SKILL:** Use superpowers:test-driven-development`
- No `@` file links (force-loads files, burns context before needed)

### Flowchart Usage
Use ONLY for non-obvious decision points, process loops where you might stop early, or "when to use A vs B" decisions.
Never for reference material (use tables/lists), code examples, linear instructions, or labels without semantic meaning.

### Code Examples
One excellent example beats many mediocre ones. Complete and runnable. Well-commented explaining WHY. From real scenario.
Don't implement in 5+ languages. Don't create fill-in-the-blank templates.

### When to Create a Skill
**Create when:** technique wasn't intuitively obvious to you; you'd reference this again across projects; pattern applies broadly.
**Don't create for:** one-off solutions; standard practices well-documented elsewhere; project-specific conventions (put in CLAUDE.md); mechanical constraints enforceable by automation.

### Directory Structure
```
skill-name/
  SKILL.md              # Main reference (required)
  supporting-file.*     # Only if needed (heavy reference 100+ lines OR reusable tools)
```

### Bulletproofing Against Rationalization (Discipline Skills)
- Close every loophole explicitly — don't just state the rule, forbid specific workarounds
- Address "Spirit vs Letter" arguments upfront: "Violating the letter of the rules is violating the spirit"
- Build rationalization table from baseline testing
- Create red flags list for self-checking

### Anti-Patterns
- Narrative examples ("In session 2025-10-03, we found...")
- Multi-language dilution (example-js.js, example-py.py, example-go.go)
- Code in flowcharts
- Generic labels (helper1, step3, pattern4)

---

## Source 4: platform.claude.com — Agent Skills Best Practices

# Skill authoring best practices

Good Skills are concise, well-structured, and tested with real usage.

## Core principles

### Concise is key
Context window is a public good. Shared with system prompt, conversation history, other Skills' metadata, actual request. Only metadata (name + description) pre-loaded at startup. SKILL.md read only when relevant, additional files only as needed.

**Default assumption: Claude is already very smart.** Only add context Claude doesn't already have.

Good (50 tokens): Direct code example with library import.
Bad (150 tokens): Explains what PDFs are, why to use the library, history of alternatives.

### Set appropriate degrees of freedom
- **High freedom** (text instructions): Multiple approaches valid, decisions depend on context
- **Medium freedom** (pseudocode/scripts with parameters): Preferred pattern exists, some variation OK
- **Low freedom** (specific scripts, no/few parameters): Fragile operation, consistency critical, exact sequence required

Analogy: Narrow bridge (cliffs on both sides) = low freedom; Open field = high freedom.

### Test with all models you plan to use
Haiku needs more guidance; Opus avoid over-explaining; Sonnet balanced. Aim for instructions that work across all target models.

## Skill structure

### YAML Frontmatter
- `name`: max 64 characters, lowercase letters/numbers/hyphens only, no XML tags, no reserved words ("anthropic", "claude")
- `description`: non-empty, max 1024 characters, no XML tags; should describe what AND when

### Naming conventions
Preferred: gerund form (verb + -ing) — `processing-pdfs`, `analyzing-spreadsheets`
Acceptable: noun phrases, action-oriented
Avoid: vague (helper, utils, tools), overly generic (documents, data, files), reserved words

### Writing effective descriptions
Always write in third person (injected into system prompt).
Good: "Processes Excel files and generates reports"
Avoid: "I can help you process Excel files"

Be specific and include key terms. Description must answer: "Should I select this Skill?"

Effective:
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

Avoid vague:
```yaml
description: Helps with documents
description: Processes data
```

### Progressive disclosure patterns
- SKILL.md body under 500 lines
- Split content into separate files when approaching limit
- Reference files loaded only when needed
- Keep references one level deep from SKILL.md (avoid nested: SKILL → advanced → details)
- Reference files >100 lines: include table of contents at top

Directory example:
```
pdf/
├── SKILL.md              # Main instructions
├── FORMS.md              # Loaded as needed
├── reference.md          # Loaded as needed
└── scripts/
    └── analyze_form.py   # Executed, not loaded
```

### Avoid deeply nested references
Nested references cause Claude to partially read files (head -100 preview), resulting in incomplete information.
All reference files should link directly from SKILL.md.

## Workflows and feedback loops

### Use workflows for complex tasks
Clear sequential steps. Checklists that Claude can copy and check off.

### Implement feedback loops
Common pattern: Run validator → fix errors → repeat. Greatly improves output quality.

## Content guidelines

### Avoid time-sensitive information
Don't include date-conditional logic. Use "old patterns" / "current method" sections instead.

### Use consistent terminology
Choose one term per concept and use it throughout. Mix = confusion.

## Common patterns

### Template pattern
For strict output requirements: exact templates. For flexible guidance: sensible defaults with judgment call permitted.

### Examples pattern
Input/output pairs for output-quality-dependent Skills. Concrete, not abstract.

### Conditional workflow pattern
Decision trees for routing to different sub-workflows.

## Evaluation and iteration

### Build evaluations first
Create evaluations BEFORE writing extensive documentation. Ensures Skill solves real problems not imagined ones.

Process: Identify gaps → Create 3+ evaluations → Establish baseline → Write minimal instructions → Iterate.

Evaluation structure: JSON with `skills`, `query`, `files`, `expected_behavior` fields.

### Develop Skills iteratively with Claude
Claude A (expert) writes Skill; Claude B (agent) tests it on real tasks. Iterate based on Claude B's observed behavior, not assumptions.

### Observe how Claude navigates Skills
- Unexpected exploration paths → structure not intuitive
- Missed connections → links need to be more explicit
- Overreliance on certain sections → move content to SKILL.md
- Ignored content → unnecessary or poorly signaled

## Anti-patterns to avoid
- Windows-style paths (use forward slashes)
- Offering too many options (provide a default, one escape hatch)

## Advanced: Skills with executable code

### Solve, don't punt
Scripts handle errors explicitly. No "magic numbers" / voodoo constants — all values justified and documented.

### Provide utility scripts
Pre-made scripts: more reliable, save tokens, save time, ensure consistency.
Distinction: "Run script.py" (execute) vs. "See script.py for algorithm" (read as reference).

### Create verifiable intermediate outputs
Plan-validate-execute pattern for complex tasks. Create plan file → validate with script → execute.

### Avoid assuming tools are installed
List required packages explicitly with install commands.

### MCP tool references
Always use fully qualified names: `ServerName:tool_name`.

## Checklist for effective Skills

Core quality:
- [ ] Description specific, includes key terms, includes what AND when
- [ ] SKILL.md body under 500 lines
- [ ] No time-sensitive information
- [ ] Consistent terminology
- [ ] Concrete examples
- [ ] File references one level deep
- [ ] Progressive disclosure used
- [ ] Workflows have clear steps

Code and scripts:
- [ ] Scripts solve problems (not punt to Claude)
- [ ] Explicit error handling
- [ ] No voodoo constants
- [ ] Required packages listed
- [ ] No Windows-style paths
- [ ] Validation/verification steps for critical operations
- [ ] Feedback loops for quality-critical tasks

Testing:
- [ ] At least three evaluations
- [ ] Tested with Haiku, Sonnet, and Opus
- [ ] Tested with real usage scenarios
- [ ] Team feedback incorporated

---

## Source 5: philschmid.de — 8 Tips for Writing Agent Skills

**1. Understand Skill Structure**
Skills: required SKILL.md + optional supporting folders. Three layers: frontmatter metadata (when to use), markdown instructions (how to use), optional assets. Two categories: capability skills (extend model abilities), preference skills (encode specific workflows).

**2. Perfect the Description**
The skill description triggers activation. Vague descriptions prevent activation; overly broad ones cause constant firing. Be specific about what AND when. 50% performance improvements observed from refining descriptions alone.

**3. Prioritize Directives Over Exposition**
Focus on what agents don't inherently know. Use specific instructions rather than general information. Lead with code examples. Explain reasoning when rules matter. Avoid overfitting skills to narrow test cases.

**4. Maintain Lean Design**
Information loads in layers — frontmatter always available, skill body when triggered, reference materials on-demand. Conserves context for actual work.

**5. Balance Constraint and Autonomy**
Describe desired outcomes rather than prescribing steps. Provide constraints instead of procedures. Reserve step-by-step instructions for scripts handling fragile sequences.

**6. Account for Negative Cases**
Define when skills should NOT activate. Test both triggering and non-triggering scenarios. Prevents skills from hijacking unrelated requests.

**7. Rigorous Testing Protocol**
Manual testing. Clear success metrics. 10-20 test prompts. Multiple trials per prompt. Isolated test environments. Prioritize description fixes over instruction rewrites when activation is the problem.

**8. Recognize Obsolescence**
Retire skills when models achieve capability independently. Applies especially as language models advance.
