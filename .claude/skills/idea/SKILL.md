---
name: idea
description: Capture a quick idea into `note/idea/` as a timestamped entry.
---

# Usage
/idea <topic-slug>? <contents>*

Examples:
- `/idea learning-with-agents goal: run N agents concurrently` — append to existing topic
- `/idea collaboration-in-ai-first-teams` — create topic, no content
- `/idea 사람끼리 의사소통 어떻게 하지` — infer topic from sentence

# Workflow

1. **Resolve topic**:
   - If the first argument matches an existing file in `note/idea/` (e.g. `learning-with-agents`), use it as the topic. The rest is contents.
   - If no match, infer the topic from the full sentence. Use a short, lowercase slug (e.g. `onboarding`, `photo-review`).
   - If the topic is ambiguous, ask the user.
2. Check if `note/idea/<topic>.md` already exists.
3. If new file, create it with a `# <topic>` heading.
4. **Classify** the content into one of four sections by epistemic status:
   - `## Observations` — facts, patterns, things noticed
   - `## Questions` — unknowns, things to explore
   - `## Hypotheses` — beliefs to validate, guesses
   - `## Principles` — validated rules, best practices
5. Find or create the matching `##` section, then find or create a `### yyyy-MM-dd` subsection for today's date. Append the idea as a bullet.
6. If the date subsection already has bullets, append below existing ones.

# Rules
- **Write everything in English.** All output files must be in English regardless of the user's input language.
- One file per topic. Dates are sections within the file.
- Do not edit or reformat existing entries.
- Do not add commentary or expand the idea. Record it as-is.
- Section order: Observations → Questions → Hypotheses → Principles.
- If the user does not specify a category, infer it from the sentence. If ambiguous, default to Observations.
