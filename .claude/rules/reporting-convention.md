# Reporting convention for Slack messages

These rules apply when composing or posting any message to Slack — including status updates, thread replies, and summaries derived from `work/` documents.

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

Style rules in `writing-convention.md` (date format, arrow notation, K/M notation, dependency language) apply equally to Slack messages.

## Language

MUST ask the user which language to post in before sending. Do not infer from previous messages.

When posting in Korean:
- Domain terms (e.g., Eval, KB, rSRR, Prompt Factory, SOUP, Backend Contract) MUST remain in English.
- All other sentences MUST be written in Korean.

## Structure

- MUST use `#` for top-level section headers. Slack renders `#` as a bold heading.
- MUST separate sections with a blank line. Do not use `---` dividers; they do not render in Slack.
- Top-level items use `-` bullets. Sub-items use indented `-` bullets (4 spaces). Do not nest more than two levels.
- MUST NOT use checkboxes (`[x]`, `[ ]`). Use plain `-` bullets for all work items regardless of status.
- MUST NOT use emojis unless the user explicitly requests them.

## Emphasis

- Use `**bold**` for item titles, section sub-headers (e.g., `**Done**`, `**In Progress**`), and key terms that need to stand out.
- MUST NOT use `_italic_` for general emphasis. Reserve italic for foreign-language terms or genuine linguistic emphasis only.
- Do not bold every line — use bold sparingly so it remains meaningful.

## Links

- MUST use Slack link syntax: `<URL|display text>` — not Markdown `[text](url)`.
- MUST include source links on items that reference specific numbers (latency, cost, sample counts) or external documents. Do not link every item.

- `>` blockquotes SHOULD be used for section-level headlines and resolved decisions, matching the `work/` document convention.

## Long content

- If the body exceeds approximately 3,000 characters, split into multiple thread replies — one per mainstream stream, then one for substreams and exited members.
- Send the parent message first (title only), then reply in-thread. Do not post all content in the parent message.
