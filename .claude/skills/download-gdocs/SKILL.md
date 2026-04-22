---
name: download-gdocs
description: Download a Google Docs document as Markdown given its URL, with inline base64 images extracted to local files. Supports documents with tabs. Use when the user provides a Google Docs URL and wants to save it locally. Triggers on "gdocs download", "google docs download", "구글 문서 다운로드", "구글독스 저장", "save google doc", "download from google docs".
---

# Usage
/download-gdocs <google docs url or document name>

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Resolve the input

From the user's full request message, extract:
- **Document reference**: a URL (starts with `https://`) or a document name / descriptive phrase
- **Tab name**: if the user mentions a tab (e.g. "Serving Options 탭", "the Serving Options tab"), note it for use in step 3

Then resolve the document reference:
- **URL**: proceed directly to step 2.
- **Document name**: search Glean using the `tinder-glean` MCP tool:
  - Query: the extracted document name with `app: gdrive` filter
  - From the results, identify candidates that are Google Docs (URL contains `docs.google.com/document`)
  - If exactly one strong match is found, use its URL and inform the user which document was selected.
  - If multiple candidates are found, present them to the user (title + URL) and ask which to download.
  - If no match is found, tell the user and stop.

## 2. Parse the URL

Extract from the resolved URL:

- **DOCUMENT_ID** from the path: `https://docs.google.com/document/d/[DOCUMENT_ID]/...`
- **tab parameter** if already present in the URL (e.g., `?tab=t.abc123`)

## 3. Resolve tab ID (if tab name was mentioned and no tab ID yet)

If the user mentioned a tab name in step 1 and the URL does not already contain a tab parameter:

1. Open `https://docs.google.com/document/d/[DOCUMENT_ID]/edit` in the browser.

2. Tell the user: **"브라우저에서 '[tab name]' 탭을 클릭한 뒤 아무 말이나 해주세요."**
   MUST wait for the user to respond before continuing.

3. After the user responds, read the current browser URL via `osascript -e 'tell application "Google Chrome" to get URL of active tab of front window'`.

4. Extract the tab parameter from the captured URL (e.g., `tab=t.abc123`).
   - If no tab parameter is present, inform the user that the tab could not be detected and ask them to paste the tab URL directly.

## 4. Construct the export URL

- Base: `https://docs.google.com/document/d/[DOCUMENT_ID]/export?format=md`
- If a tab parameter exists, append: `&tab=[TAB_VALUE]`

## 5. Download via browser

The user must be logged in to Google in their default browser — auth is handled by the existing session.

1. Record the name of the current newest `.md` file in Downloads (e.g., `ls -1t ~/Downloads/*.md | head -1`).
2. Open the export URL: `open "<export_url>"`
3. Wait a few seconds, then re-query the newest `.md` file. If it differs from step 1, that is the downloaded file.

## 6. Determine output path

Always save to `note/source/` using the document title and today's date.

Extract the document title from the downloaded file:
- Read the first line (typically an `# H1` heading)
- Strip leading `#`, whitespace, and markdown bold markers (`**`)
- Lowercase and replace characters not suitable for filenames (spaces, `/`, `:`, etc.) with `-`
- Collapse consecutive hyphens; strip leading/trailing hyphens
- If no title can be extracted, fall back to `[DOCUMENT_ID]`

Output path: `note/source/YYYY-MM-DD-<title-slug>-gdocs.md` where the date is today's date.

## 7. Move and insert header

Move (and rename) the downloaded file from `~/Downloads/` to the target path.

Insert a metadata header table at the top of the file (before the H1 title):

| Key           | Value                          |
| ------------- | ------------------------------ |
| Source        | `<ORIGINAL_URL>`               |
| Downloaded at | `<YYYY-MM-DD HH:MM:SS>`        |

Followed by `---` and a blank line before the document content.

## 8. Extract inline base64 images

Google Docs export embeds images as base64 data URIs. Extract them into `note/source/images/`.

**Pattern:** Reference definitions at the end of the file:
```
[image1]: <data:image/png;base64,iVBOR...>
```
Used inline as: `![][image1]`

**Procedure:**
1. Create `note/source/images/` if it does not exist.
2. For each reference definition `[imageN]: <data:image/TYPE;base64,DATA>`:
   - Decode base64 to binary.
   - Name the file using the SHA-256 hash (first 12 hex chars) with appropriate extension. Use `jpg` instead of `jpeg`.
   - Skip if the same hash already exists (deduplication).
3. Rewrite Markdown:
   - Replace `![][imageN]` with `![imageN](images/HASH.ext)`
   - Remove all `[imageN]: <data:image/...>` definition lines.

## 9. Report

Report to the user:
- Output file path
- Number of images extracted (if any)
- Brief preview (first ~10 lines) using the Read tool
