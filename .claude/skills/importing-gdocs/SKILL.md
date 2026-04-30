---
name: importing-gdocs
description: Download a Google Docs document as Markdown and save it as a source note under note/<scope>/ for use as reference or research material. Triggers when the user wants to collect, archive, or source content from a Google Doc — e.g. "note에 저장", "소스로 저장", "참고자료로 추가", "자료로 수집", "소스 노트로 추가", "save as source", "import to note", "add as reference".
---

# Usage
/importing-gdocs <google docs url or document name>

The key words MUST, MUST NOT, and SHOULD in this document are to be interpreted as described in RFC 2119.

# Workflow

## 1. Download to ~/Downloads/

Invoke /downloading-gdocs with the same argument. The result is `~/Downloads/YYYY-MM-DD-<title-slug>-gdocs.md` with any images in `~/Downloads/images/`.

## 2. Resolve scope and save to note/<scope>/

Use **resolving-docs** to determine the target `note/<scope>/` directory from the document topic or filename.

The output filename is identical to the one produced in step 1.

1. Resolve `note/<scope>/` via resolving-docs. Create the directory if it does not exist.
2. Copy `~/Downloads/YYYY-MM-DD-<title-slug>-gdocs.md` → `note/<scope>/YYYY-MM-DD-<title-slug>-gdocs.md`.
3. For each image referenced in the file, copy `~/Downloads/images/HASH.ext` → `note/images/HASH.ext`.
   - Create `note/images/` if it does not exist.
   - Skip if the same file already exists in `note/images/` (deduplication by filename).
4. Update image references in the saved file from `images/HASH.ext` → `../images/HASH.ext`.

## 3. Report

Report to the user:
- `~/Downloads/` path (intermediate copy)
- `note/<scope>/` path (archived copy)
- Number of images copied (if any)
- Brief preview (first ~10 lines) of the saved file using the Read tool
