---
name: idea
description: Capture a quick idea into `note/idea/` as a timestamped entry.
---

# Usage
/idea <sentence>

# Workflow

1. Infer the topic from the sentence. Use a short, lowercase slug (e.g. `onboarding`, `photo-review`).
2. Check if `note/idea/<topic>.md` already exists.
3. If new file, create it with a `# <topic>` heading.
4. Find or create a `## yyyy-MM-dd` section for today's date. Append the idea as a bullet:
   ```
   - <sentence>
   ```
5. If today's date section already exists, append below existing bullets.

# Rules
- **Write everything in English.** All output files must be in English regardless of the user's input language.
- One file per topic. Dates are sections within the file.
- Do not edit or reformat existing entries.
- Do not add commentary or expand the idea. Record it as-is.
- If the topic is ambiguous, ask the user.
