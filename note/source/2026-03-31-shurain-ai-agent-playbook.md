# AI Agent Playbook: From ChatGPT User to Agentic Workflow
**Authors**: Shurain Ha + Claude Opus 4.6
**Date**: 2026-03-30
**Audience**: MG AI team — engineers, PMs, and leads

---

## Why This Exists

Most of the team uses ChatGPT or similar tools for Q&A — asking questions, getting answers, maybe generating some code snippets. That's Level 0. It's useful, but it leaves 90% of the value on the table.

This playbook documents how Shurain went from Level 0 to a fully agentic workflow in ~4 weeks (Feb 28 – Mar 28, 2026), extracted from actual session logs. It's not theory — it's a reconstruction of what actually happened, in what order, and why each step mattered.

The core insight: **the gap between "AI chatbot" and "AI agent" is not about the model getting smarter — it's about you giving it persistent context, structured access to your tools, and a feedback loop that compounds.**

---

## The Progression Model

| Level | Name | What It Looks Like | Time to Reach |
|-------|------|--------------------|---------------|
| 0 | **Q&A** | ChatGPT in browser. Ask questions, get answers. Copy-paste results. | You're here |
| 1 | **Workspace** | AI has access to your files. Reads/writes docs, code, notes. | Day 1-2 |
| 2 | **Domain Memory** | AI knows your project context, team, org, and can build on prior work. | Week 1 |
| 3 | **Tool Integration** | AI connects to your actual work systems (Notion, Jira, Databricks, Calendar). | Week 1-2 |
| 4 | **Reusable Skills** | Recurring workflows codified as repeatable commands. Morning routine, meeting prep, etc. | Week 2-3 |
| 5 | **Multi-Model Adversarial** | AI checks its own work by consulting other models. Pre-mortems on its own analysis. | Week 3-4 |
| 6 | **Autonomous Loops** | AI runs multi-step workflows end-to-end with minimal intervention. | Week 4+ |

**You don't need to reach Level 6 to get value.** Level 2 alone — persistent context — is a 5x productivity multiplier for anyone who works with documents, data, or code.

---

## Phase 1: First Week — "Replicate Something That Already Exists"

### The Strategy

Don't start by building something new. Start by **replicating an existing report or analysis that someone else already did.** This is the single most important piece of advice in this playbook.

Why replication:
1. **You already know what the answer should look like** — so you can tell if the AI is working correctly
2. **It forces you to set up data access** — permissions, table names, query patterns
3. **It builds domain knowledge as a side effect** — the AI learns your schema, your metrics, your terminology
4. **It's low-risk** — if the replication fails, you've lost nothing; if it succeeds, you've gained a working pipeline

### What Shurain Actually Did (Day 1-3)

**Day 1: Article editing + knowledge synthesis.** Started with a Korean essay he'd already written and had feedback on. Asked the AI to iterate on it. Not a cold start — warm start with existing material. This established the basic interaction pattern: propose → approve → execute.

**Day 2: Org intelligence gathering.** Mike (SVP) asked for visibility into how MG AI works. Instead of writing from scratch, Shurain had the AI:
1. Read his existing 1-on-1 notes, meeting minutes, and newsletters
2. Pull 6 months of meeting titles from Notion
3. Cross-reference with skip-level notes and workshop PDFs
4. Synthesize into a single document

The document went through **7 correction rounds** in one session — each triggered by a short question like "Did you also check the rituals?" or "What about our efforts to support people's growth?" Each correction pushed the AI from synthesis back to primary sources.

**Day 3: Newsletter translation + curation.** 21 Korean newsletters translated to English. First attempt crashed (too many parallel requests). Second attempt succeeded using sequential processing. Then curated which ones to share with Mike, considering political risk (what to redact) and audience fit.

### The Pattern

```
1. Pick something that already exists (report, analysis, doc)
2. Give the AI access to the source material
3. Ask it to reproduce it
4. Correct what it gets wrong (this is where the learning happens)
5. The corrections become persistent knowledge for next time
```

### Starter Exercises by Role

**For Engineers:**
- Replicate a past A/B test analysis. Give the AI the Databricks tables and the final report. Ask it to reproduce the numbers. You'll discover which tables matter, which joins are tricky, and which metrics have gotchas.
- Replicate a PR review. Give it the diff and your previous review comments. See if it catches what you caught.

**For PMs:**
- Replicate a PRD or spec you already wrote. Give it the meeting notes and stakeholder inputs. See how close it gets — the gaps reveal what you added that wasn't in the inputs.
- Replicate a competitor analysis. Give it the raw data and see if it reaches the same conclusions.

**For Leads/Managers:**
- Replicate your weekly update. Give it access to your team's Jira board, recent meeting notes, and last week's update. See what it includes vs. what you wrote.

---

## Phase 2: Building Persistent Context — "The AI Remembers"

### Why This Matters

The difference between ChatGPT and an agentic workflow is **memory**. A browser chatbot starts from zero every conversation. An agent with persistent context starts from where you left off.

### What to Build

**1. A working directory (your "wiki")**

Create a directory where the AI can read and write. This is your shared workspace. Structure:

```
my-workspace/
├── Daily/              # Daily notes (YYYY-MM-DD.md)
├── 1on1/               # 1-on-1 meeting notes
├── context/            # Domain context files
├── CLAUDE.md           # Instructions for the AI
└── [project files]     # Whatever you're working on
```

**2. A CLAUDE.md file**

This is the AI's instruction manual. Start minimal and grow it:

```markdown
# CLAUDE.md

## About Me
- [Your name], [role] at [team]
- Working on: [current projects]

## Conventions
- Daily notes go in Daily/YYYY-MM-DD.md
- Append with timestamps, never overwrite

## Context
- [Team name] does [what]
- Key systems: [list]
- Key people: [list with roles]
```

Every time the AI does something wrong or you correct its approach, **add that correction to CLAUDE.md**. This is how the AI learns your preferences. From Shurain's logs, rules like "no OAuth for Google services" and "never run Python on the HyperPod login node" were discovered through natural task execution and immediately codified.

**3. Domain context files**

After your first few replication exercises, extract what the AI learned into context files:

```markdown
# context/our-data.md

## Key Tables
- `hubble_core.screen_sessions` — client-side events
- `hubble.c_r_s_v2_*` — server-side CRS events

## Gotchas
- `chats_day` has multiple rows per chat_id (one per active day)
- `n_way_conversation` changes across rows — dedup by DESC to get final state
```

This is the compound interest of agentic work: **every investigation makes the next one faster.**

---

## Phase 3: Connecting to Your Real Systems

### The Unlock

The jump from "AI reads my files" to "AI queries my databases and pulls from Notion" is where most people stall. But it's also where the biggest productivity gain lives.

### What Shurain Connected (in order)

| System | How | What It Enabled |
|--------|-----|-----------------|
| **Local files** | Built-in (Claude Code reads filesystem) | Document synthesis, daily notes |
| **Notion** | MCP server | Meeting notes, project docs, team wikis |
| **Google Calendar** | Chrome DOM scraping (no OAuth) | Daily schedule awareness, meeting prep |
| **Databricks** | MCP server via Agentis Gateway | Data analysis, metric verification |
| **Jira** | MCP server via Agentis Gateway | Sprint tracking, ticket context |
| **Glean** | MCP server | Cross-system search of internal docs |

### Start With One

Don't try to connect everything at once. Pick the system you spend the most time context-switching into:
- **Engineers**: Databricks or your code repos
- **PMs**: Notion or Jira
- **Managers**: Calendar + 1-on-1 notes

### The Replication Trick, Again

Once connected, replicate an existing artifact that required that system:
- Connected Databricks? Replicate a past analysis.
- Connected Notion? Pull meeting notes and summarize what happened last week.
- Connected Jira? Generate last sprint's status update.

---

## Phase 4: Codifying Workflows — "Skills"

### What This Means

A "skill" is a repeatable workflow that you've done enough times to standardize. Instead of explaining the task from scratch each time, you encode it once and invoke it with a short command.

### Shurain's Skill Evolution (chronological)

| Week | Skills Built | Trigger |
|------|-------------|---------|
| 1 | `/gcal`, `/gdoc`, `/link`, `/learn` | Needed to pull calendar, download docs, link files to daily notes |
| 1 | `/pm-tl-sync`, `/leaders-sync` | Needed meeting notes from Notion regularly |
| 2 | `/morning`, `/evening` | Daily routine was predictable enough to codify |
| 2 | `/prep` | Meeting prep followed the same pattern every time |
| 2 | `/decide`, `/feedback` | Management decisions and feedback followed structured frameworks |
| 3 | `/premortem`, `/deep-dive`, `/investigate` | Analysis work needed adversarial review |
| 3 | `/weekly-review`, `/monthly-review` | Periodic synthesis was time-consuming but formulaic |
| 4 | `/consult`, `/codex`, `/gemini-cli` | Multi-model verification became standard practice |

### The Rule: Three Times Then Codify

If you do something three times and it follows roughly the same pattern, turn it into a skill. Signs you should codify:
- You find yourself explaining the same context repeatedly
- The output format is predictable
- The input sources are the same

### Starter Skills for the Team

**For everyone:**
- **Daily standup prep**: Pull yesterday's commits/tickets, summarize what happened
- **Meeting prep**: Pull relevant context before a meeting (notes from last time, open action items)

**For engineers:**
- **Code review assistant**: Given a PR diff, check for common patterns (missing tests, security issues, performance concerns)
- **Incident analysis**: Given an alert, pull relevant logs, metrics, and recent deploys

**For PMs:**
- **Spec reviewer**: Given a draft spec, check for completeness against a template (user stories, success metrics, edge cases)
- **Stakeholder update**: Pull recent progress from Jira/Notion, format for a specific audience

---

## Phase 5: The Feedback Loop That Makes It Compound

### The Secret Sauce

The thing that separates "AI-assisted" from "AI-native" is the feedback loop:

```
Use AI for a task
    → AI makes mistakes
        → You correct the mistakes
            → Correction is saved as context/memory/rule
                → AI doesn't make that mistake again
                    → Next task starts from a higher baseline
```

This is what happened in Shurain's logs. By week 4:
- One-word commands ("todo") triggered multi-step workflows
- Meeting prep that took 30 minutes now took 2 minutes
- The AI knew the org chart, the team dynamics, the data schema gotchas, and the political context

### How to Run the Feedback Loop

**After every session, ask yourself:**
1. Did the AI do anything I had to correct? → Add the correction to CLAUDE.md
2. Did I explain context that the AI should have known? → Add it to a context file
3. Did I do a multi-step task that I'll do again? → Consider making it a skill
4. Did the AI produce something useful? → Save it where it can be found later

**Shurain's specific moves:**
- Built `/learn-from-sessions` — a skill that reads its own session logs, extracts corrections and preferences, and proposes memory updates
- Maintained `lessons.md` — hard-won analytical lessons, each with a frequency count and last-triggered date
- Did weekly adversarial reviews of his own analysis using multiple models

---

## Phase 6: Working Patterns That Actually Work

### Pattern 1: Iterative Grounding

Don't write a detailed brief. Let the AI produce a first draft, then correct it with short domain-specific interventions.

**Example from the logs:**
```
Shurain: "Write up how MG AI works for Mike"
[AI produces draft from existing files]
Shurain: "Did you check the Notion meeting notes?"
[AI rewrites with real data]
Shurain: "What about our rituals? We do a lot of pre-mortems"
[AI adds section with evidence from Notion]
Shurain: "AI Velocity Hack — your description is the Tinder hackathon, not ours"
[AI corrects]
```

7 correction rounds, each one sentence. Final doc was grounded in primary sources. Total Shurain effort: ~15 minutes of corrections on a document that would have taken 3+ hours to write from scratch.

**Why this works:** You're the domain expert. The AI is the writer/researcher. You steer; it produces. This is faster than either writing from scratch or giving a detailed brief upfront (which you'd inevitably get wrong or incomplete).

### Pattern 2: Plan → Execute → Next Session

For complex tasks, split across sessions:

```
Session 1: "Let's plan how to do X" → AI writes a plan file
[You review the plan offline, maybe in 5 minutes between meetings]
Session 2: "Execute this plan: [paste plan]" → AI implements
```

This is how the "How MG AI Works" deck was built — plan in one session, document in another, slides in a third, corrections in a fourth.

### Pattern 3: Voice Ownership

For anything that represents YOUR thinking (memos, strategy docs, feedback), don't let the AI ghostwrite.

```
Shurain: "The domain expertise essay — it wasn't written by me, it's your creation."
AI: "You write, I engage with the substance."
```

Use the AI to:
- Research (pull meeting notes, find examples, check data)
- Structure (organize your rough thoughts into sections)
- Challenge (stress-test your arguments)
- Edit (tighten language after you've written it)

Don't use the AI to write things that need to sound like you.

### Pattern 4: Context Window Management

The AI has a finite attention span. Shurain learned this the hard way when a 21-newsletter translation crashed the context:

- **Sequential over parallel** when the total data is large
- **Carry plans across sessions** — end one session with a written plan, start the next by pasting it
- **Short sessions for unrelated topics** — don't keep a mega-session going across different projects

### Pattern 5: When Something Seems Off, Run With It

The most valuable discoveries came not from planned analysis but from noticing anomalies during replication work and immediately investigating:

- Replicating Lichao's cold-start analysis → noticed the scoring methodology had a flaw → built a diagnostic framework
- Replicating an A/B test → noticed offline/online metric gap → traced to negative sampling distribution differences
- Building an SAE interpretation → noticed features were mislabeled → discovered unit-of-analysis confusion

**The lesson:** Replication is not just reproduction — it's an opportunity to find things the original analyst missed. When something looks off, don't dismiss it. Investigate.

---

## Getting Started: Your First Week

### Day 1: Setup (30 minutes)

1. Install Claude Code (or your preferred agentic AI tool)
2. Create a working directory with a basic `CLAUDE.md`
3. Add your current project's key context (team, systems, goals)

### Day 2-3: Replicate (2-3 hours)

Pick ONE thing from this list:
- A past analysis report → give the AI the data source and final report, ask it to reproduce
- A meeting summary → give it the raw notes, ask it to produce the summary
- A spec or PRD → give it the inputs, ask it to draft
- A weekly update → give it access to your Jira/Notion, ask it to generate

**Correct every mistake.** Each correction you make is an investment that pays dividends forever.

### Day 4-5: Connect (1-2 hours)

Connect ONE external system (start with the one you use most):
- Notion, Jira, or Confluence for docs
- Databricks or BigQuery for data
- Calendar for scheduling awareness

Replicate something that requires this system.

### Week 2: Start Building Habits

- Start each morning by asking the AI "what's on my plate today?"
- Before each meeting, ask for a 2-minute prep brief
- After each complex task, spend 30 seconds updating CLAUDE.md with anything the AI should remember

---

## Common Mistakes

| Mistake | Why It Happens | What To Do Instead |
|---------|---------------|-------------------|
| Starting with something novel | "AI should help me think new thoughts" | Start with replication — novel comes after you trust the tool |
| Not correcting mistakes | "Good enough" | Every uncorrected mistake is a missed learning opportunity |
| Over-briefing upfront | Trying to specify everything before starting | Let the AI draft, then correct iteratively |
| Expecting perfection | "AI should get it right the first time" | First drafts are always wrong. The value is in the iteration speed. |
| Keeping context in your head | "I know this, why write it down?" | If you don't write it down, the AI can't know it next time |
| Giant sessions | "Let me do everything in one sitting" | Short, focused sessions. Carry state in files, not in context. |
| Treating it as a search engine | "What is X?" | Give it a task, not a question. "Analyze X in our codebase" > "What is X?" |

---

## FAQ

**Q: I'm a PM, not an engineer. Is this relevant to me?**

Yes. The replication approach works for any artifact — specs, analyses, stakeholder updates, competitive analyses. You don't need to write code. The AI reads your Notion pages, Jira boards, and docs — and produces drafts you correct.

**Q: This seems like a lot of setup. Is it worth it?**

Shurain's setup took ~3 days of active work. By week 2, tasks that took hours were taking minutes. By week 4, the AI was autonomously running parallel analyses while he was in meetings. The ROI is not linear — it compounds.

**Q: What if I make the AI worse by teaching it wrong things?**

Update your context files when you learn you were wrong. The AI uses the latest version of your files — it doesn't hold grudges about old instructions.

**Q: How is this different from just using ChatGPT?**

Three things ChatGPT doesn't have:
1. **Persistent memory** — it forgets everything between conversations
2. **File access** — it can't read your codebase, your docs, or your data
3. **Tool integration** — it can't query your Databricks, pull from your Notion, or check your Jira

These three things are the difference between a chatbot and a colleague.

**Q: What's the minimum viable version?**

A working directory with a `CLAUDE.md` file and one completed replication exercise. That's it. Everything else is optional optimization.

---

## Appendix: The Full Timeline (What Actually Happened)

| Date | What Shurain Did | Level |
|------|-----------------|-------|
| Feb 28 | Edited an essay with AI feedback. Ran `/learn` on 7 great thinkers. | 1 |
| Mar 1 | Tested `/gcal` (calendar scraping). Parsed Slack "Later" items. | 1→2 |
| Mar 2 | Built "How MG AI Works" doc — 7 correction rounds grounded in Notion data. | 2→3 |
| Mar 2 | Translated 21 newsletters sequentially (learned context limits the hard way). | 3 |
| Mar 3 | AI All-Hands prep from a 2-word prompt ("all-hands prep"). | 3 |
| Mar 4 | Built `/gdoc` skill. Domain expertise memo — voice ownership enforced, 27 IC-level examples from Notion. | 3→4 |
| Mar 5 | Built html2pptx converter. Deterministic calendar parser (replaced LLM parsing). | 4 |
| Mar 6 | **Marathon day**: `/prep`, `/decide`, `/feedback` skills. Backfilled entire year's notes. | 4→5 |
| Mar 6 | Built `/learn-from-sessions` — AI learns from its own mistakes. | 5 |
| Mar 7 | Built `/deep-dive` for code archaeology. Evaluated external frameworks (rejected most). | 5 |
| Mar 9 | Connected Notion MCP, Glean MCP, Jira MCP. First Tinder codebase deep-dive. | 5 |
| Mar 9 | **Replicated Lichao's cold-start diagnosis** — then extended it with scoring diagnostics. | 5 |
| Mar 9-14 | Built `/codex`, `/gemini-cli`, `/cursor`, `/consult` — multi-model adversarial review. | 5→6 |
| Mar 16 | Parallel decision analyses generated autonomously. Knowledge index consolidated. | 6 |
| Mar 19 | Cost audit. Built `enforce-sonnet` hook. Saved ~$2,000/month in API costs. | 6 |
| Mar 28 | 50+ skills, 7 MCP integrations, automated daily pipeline, multi-model review standard. | 6 |

---

## Appendix B: A Week at Level 6 (Mar 24–30, 2026)

What does a mature agentic workflow actually look like in practice? Here's a real week — not curated, just what actually happened.

### The Setup

10–15 concurrent workstreams running simultaneously: Pairs JP experimentation, LDM sequence model (Phases 3–7), Tinder experimentation framework, Chemistry steering, AURA causal analysis, 4WC two-tower model, VLM photo scoring, fast training optimization, and tooling infrastructure. Each workstream has its own state tracked in `workstreams.md` — phase, status, what it's waiting on, last result, next step. `/dispatch` resumes any thread with full context.

### Tuesday: 8 Meetings, Zero Prep Time

8 meetings that day. Morning prep for all 8 auto-generated by `/prep` — each brief pulled from 1-on-1 history and daily notes. Each included: last 3 discussions, open action items, and suggested talking points. Total human effort: review the briefs while drinking coffee.

Meanwhile, a literature survey on Smart Photos personalization ran in the background (Netflix, Uber, Instacart, Google Ads, academic bandit papers). SAE coaching prototype locked a configuration (145 features, R²=0.727). LDM Phase 4 results written up. A Floyd re-derivation (#10: Hard Filter Blindness) extracted from the SAE work and filed into `lessons.md`.

### Wednesday: Overnight Experiments + 8 More 1-on-1s

Pairs FTS analysis ran **overnight autonomously** while Shurain slept. By morning: direction-filtered AIPW analysis complete, key discovery that a single-day direction estimate was biased (22.2% not 44.9%), full write-up ready for review. LDM Phase 5 also completed overnight on HyperPod — COUNTING verdict confirmed, Floyd #15 extracted.

During the day: 8 back-to-back 1-on-1s (all with AI-generated prep). In parallel, the AI ran a post-mortem on FTS v1/v2 that traced the root cause to a spec inheriting the wrong lever without validation. Three system fixes were written into the experiment skill as anti-pattern checks.

### Thursday: Self-Improving Tooling

Built a skill harvester that reads its own session logs, extracts workflow patterns, and auto-generates playbooks. First attempt extracted procedural steps (low value). Pivoted to extracting **domain gotchas** — API behaviors, schema quirks, system invariants. Produced 8 infrastructure playbooks from 50 session episodes, including 3 Databricks traps not in any hand-crafted documentation.

Meanwhile: VLM LoRA experiment submitted to HyperPod, experimentation framework diagnosis completed (10 gap hypotheses stack-ranked), Pairs RSRR model passed overnight (ρ=0.3484).

### Friday: The Adversarial Review Catches Real Errors

This was the most concentrated day.

**What the adversarial review caught:**
- FTS analysis: Codex consultation flagged that a "60-min discount explains majority of conversions" claim was wrong — it explained only 26%. Report corrected before sharing.
- Resub/LTV EDA: initial conclusion "deeper engagement = worse renewal" was **reversed** by adversarial review. Codex + Gemini both identified collider bias + immortal-time bias. The actual finding: matched users are 2.5x more valuable per registered user.
- Post-sub survival analysis: fundamentally misspecified (invalid censoring). Day 28-29 cancellation cluster = App Store renewal notification, not churn.

Every one of these would have been shared as a finding if not caught. The adversarial review (`/premortem` + `/consult` with 2 external models) is not optional — it's the quality gate.

**Also that day:** 4 literature surveys completed (cross-side measurement, chemistry steering, RL vs SFT, chief-of-staff AI agents). Fast training playbook validated a 12× speedup. Tinder recs onboarding doc generated. Phoenix experimentation platform code archaeology completed. 4WC Two-Tower code reviewed — 27 pitfalls catalogued.

### Saturday: Literature-Driven Experimentation

A literature survey on fast tabular training (22 papers) → 7 experiments testing the top techniques → winner: Polars on consolidated files = **304× speedup** (0.7s vs 199s baseline). The survey-driven approach found the answer in hours, not weeks of trial-and-error.

Experimentation framework executive summary synthesized: pulled actual data from 39 real Tinder experiments, computed the effect size distribution (P50 = +0.2%), proved the current system is underpowered (MDE ~2.5% but most effects < 1%), and proposed a three-phase fix — while ruling out a seemingly promising approach (Airbnb's exposure index) with a precise structural argument.

Built the librarian CLI — a search tool for the entire wiki. Full index in 6 seconds, search in <0.01s, 7 subcommands. Built in a single `/forge` session.

"Ideas Worth Stealing" index compiled: ~70 techniques from 11 literature surveys (~180 papers), indexed by problem type with a quick lookup table.

### Sunday: Self-Correction Saves the Analysis

CUPED variance reduction analysis: pre-mortem caught that the headline number (84% variance reduction) was **2× overstated**. The correct number was 38%. Shared trends cancel in treatment-control subtraction; ρ=0.986 was a Simpson's paradox artifact. The original estimate of 43% was actually right — "we were wrong to 'correct' it."

Chemistry steering post-gate signal validated on 326M impressions. Pairs algorithmic confounding analysis revealed 81% of variance unexplained. VLM experiment spec written and submitted.

### The Numbers

| Metric | This Week |
|--------|-----------|
| New work files produced | 38+ |
| Literature surveys completed | 8 (covering ~180 papers) |
| Floyd re-derivations extracted | 8 |
| Experiments run on HyperPod | 33 (4WC alone) |
| Pre-mortems / adversarial reviews | 10+ |
| Errors caught before sharing | 5 (including 2 sign-reversals) |
| Concurrent workstreams | 10–15 |
| 1-on-1 meetings with AI-generated prep | 16 |

### What Makes This Different from ChatGPT

1. **Overnight autonomous execution.** Experiments run on HyperPod while you sleep. A cron job polls status and executes the next step. You wake up to results, not to "please run this command."

2. **Adversarial quality gates catch real errors.** Every analysis goes through `/premortem` + 2-model consultation before it's shared. This week alone, 5 errors were caught — including 2 that would have reversed the conclusion.

3. **Literature surveys feed directly into experiments.** A 22-paper survey on fast training → 7 experiments → a 304× speedup, all in one day. The survey isn't background reading — it's the hypothesis generator.

4. **Skills compound.** Every major error becomes a Floyd re-derivation → goes into `lessons.md` → gets checked before the next plan. The `/eda` skill was built from errors caught in THIS week's FTS workstream. The system gets better at catching its own mistakes over time.

5. **Parallel workstreams with no context switching cost.** `/checkpoint` saves state, `/dispatch` resumes with full context. Switching between Pairs causal inference and Tinder experimentation framework takes seconds, not the 30 minutes it takes a human to reload context.

6. **The AI catches things the human missed.** The CUPED 2× overstatement, the collider bias in resub/LTV, the exposure index category error — these were all corrections generated by the adversarial pass, not by Shurain noticing something was off.

---

## The Takeaway

The progression is:

**Replicate → Correct → Remember → Automate → Compound**

Every correction you make, every context file you write, every skill you build — it stays. The AI doesn't forget. The 30 seconds you spend today saves 30 minutes next month. Start by replicating something that already exists. The rest follows naturally.
