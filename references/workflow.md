# Workflow details

Example: "Review Saturday's wedding shoot in /photos/2026-11-08-wedding, find frames with the cake, and make a Selects album."

1. file-agent inventories the folder (read only).
2. file-agent reports duplicate groups (read only).
3. photo-agent searches for "cake" (read only).
4. The parent proposes a shortlist. The user confirms.
5. photo-agent matches each confirmed file to its photo library item (read only); unmatched files are reported and left out.
6. photo-agent creates the album "Selects" from the matched items (approval required).
7. Optional: file-agent looks up the tag by name; if it is missing the user is asked before it is created; then tags the files (approval may be required).

Edge cases: files that are not in the photo library (they can be tagged but not added to the album); folder with mixed videos and photos (report counts, use photos only); very large folder (ask the user to narrow by date or subfolder); no matches (no album is created); user declines approval (stop, report that nothing changed).

Closing line, only after full success: "Review complete. No files were deleted, moved or renamed."
