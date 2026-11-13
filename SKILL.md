---
name: shoot-review-assistant
description: Helps photographers and editors review a photo shoot stored on UGREEN AI NAS. Finds the shoot's photos, reports duplicate image files, searches frames by subject or text, and, only after the user approves, creates a "Selects" album and tags chosen files. Never deletes, moves or renames files.
---

# Goal

Give a photographer a fast, safe first pass over one shoot: what is in it, which frames are duplicates (as the NAS dedupe scan reports them), which frames match a subject, and a proposed shortlist the user can turn into an album and tags. Scope: photos and image files already on the NAS and authorized for the user. Out of scope: editing images, deleting or moving files, face identification of named people, anything outside the user's chosen folder or album.

# Required input

- The shoot location: a folder path or album name given by the user (never guessed).
- Optional: subjects or keywords to look for, text to find in images, a name for the Selects album, tags to apply.

Ask once if the location is missing. Do not pre-collect parameters owned by child experts.

# Capability evidence

- ug-filesystem -> file-agent
- ug-filemgr -> file-agent
- ug-photo -> photo-agent

Actions expected to be used by the child experts (generation-time facts from the catalog, validated by the child workflow): ug-filesystem listing; ug-filemgr `mediainfo`, `scan-dedupe` with scope images, `tag-get-by-name` or `tag-list` (read), `tag-add` to create a missing tag and `tag-add-file` (state-change, approval conditional); ug-photo `search-picture` (filters `dir`, `ocr-contains`, `file-name-contains`), `search-classify`, `vision-ocr` (read), `create-album` (state-change, workflow approval). See [workflow details](references/workflow.md) and [disclosures](references/disclosure.md).

# Expert orchestration

Each delegation is self-contained and read-only unless marked.

1. Inventory (read). `delegate --target file-agent --instruction 'List the image files directly in <FOLDER> only, without changing anything. Report file count, formats, size range and capture-date range from media information. If the folder is missing or empty, say so and stop. Do not invent files.'`
2. Duplicates (read). `delegate --target file-agent --instruction 'Scan <FOLDER> for duplicate images (scope images) in read-only mode. Report groups of file paths exactly as the scan returns them; do not call files near-duplicates unless the scan itself says so. Do not delete, move or rename anything. If nothing is found, say so.'` Pass only the verified group list forward.
3. Subject search (read, only if the user gave keywords or text). `delegate --target photo-agent --instruction 'Search the photos in <FOLDER> for <SUBJECT/TEXT>. Return the matching photo paths and, for text requests, the text recognized. Read-only. If there are no matches, say so.'`
4. Proposal. The parent writes a shortlist from step 2 and 3 results only: one representative per duplicate group plus subject matches. Show it and ask the user to confirm, edit or cancel. Nothing is changed before this answer.
5. Resolve (read, only after explicit confirmation). Photo albums use photo library items, not file paths, so: `delegate --target photo-agent --instruction 'For each of these files, find the matching photo in the photo library, using search-picture with dir and file-name-contains. Return the matches and list any file you cannot match. Read-only: <VERIFIED PATHS>.'` Report unmatched files to the user and leave them out.
6. Album (state change). `delegate --target photo-agent --instruction 'Create an album named <ALBUM> containing exactly these library photos: <RESOLVED ITEMS>. Do not include any other photo.'`
7. Tags (state change, only if the user asked for tags). First `delegate --target file-agent --instruction 'Look up the tag named <TAG> and return its id. Read-only. If it does not exist, say so.'` If it does not exist, ask the user before creating it. Then `delegate --target file-agent --instruction 'Add the tag with id <TAG ID> to exactly these files: <VERIFIED PATHS>. Change nothing else.'` Approval prompts raised by the experts are the user's to answer.

The parent never calls `cli.invoke` and never bypasses approval.

# Completion and failure

- `completed`: report only what the expert returned (counts, groups, matches, album, tags). Add the closing line from the workflow reference only when every requested step completed.
- `failed`: report the step and error, keep earlier results, stop dependent steps.
- `cancelled`: report what was and was not done; do not retry unasked.
- `waiting_user`: relay the expert's question verbatim and wait.
- `waiting_approval`: say which step awaits approval and that nothing has changed yet.
- Empty result: report "no matches" or "no duplicates"; never fabricate files. Partial result: list finished and unfinished steps; do not create an album from unresolved or unverified items.
