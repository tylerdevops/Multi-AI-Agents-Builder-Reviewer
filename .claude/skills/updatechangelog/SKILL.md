---
name: updatechangelog
description: Append a session entry to CHANGELOG.md, grouping multiple same-day entries under one date heading instead of repeating the date. Use when the user invokes /updatechangelog, or asks to "update the changelog", "log this session", or "add a changelog entry" for this repo.
---

# Update changelog

Append one entry to this repo's `CHANGELOG.md` for the work just completed, following the file's own rules (append-only, never edit or delete a past entry, one entry per session/task).

## Steps

1. **Read `CHANGELOG.md`** (the whole file, or at least from the last `## [` heading to the end) to find the most recent entry.
2. **Determine today's date** (`YYYY-MM-DD`).
3. **Gather the entry content** from the session just finished: agent name, branch, commit hash(es), files touched, what/why, decisions made, ruled out, open questions. If any of these weren't explicitly discussed, infer them from the actual diff/commits rather than leaving placeholders — but do not fabricate reasoning that didn't happen; write "none" or "n/a" where genuinely empty.
4. **Check whether the last entry's date already matches today:**
   - **If today already has an entry** (the most recent `## [YYYY-MM-DD] — ...` heading is today's date): do **not** add a second `## [YYYY-MM-DD]` heading. Instead, append a nested sub-entry directly under that existing day's heading (after its last field, before the next `## ` heading or end of file), using:
     ```
     ### [Agent] — [Task/feature name]
     **Branch:** `feature/[name]` | **Commit(s):** `[short hash(es)]`
     **Files touched:** `path/one`, `path/two`

     **What/why:** ...
     **Decisions made:** ...
     **Ruled out:** ...
     **Open questions / left for next agent:** ...
     ```
     This keeps one `##` heading per calendar day, with multiple sessions nested as `###` beneath it, rather than repeating the date.
   - **If today does not yet have an entry** (last entry is an older date, or the file has no entries yet): append a new top-level entry using the full format from the file's own **Entry format** section:
     ```
     ## [YYYY-MM-DD] — [Agent] — [Task/feature name]
     **Branch:** `feature/[name]` | **Commit(s):** `[short hash(es)]`
     **Files touched:** `path/one`, `path/two`

     **What/why:** ...
     **Decisions made:** ...
     **Ruled out:** ...
     **Open questions / left for next agent:** ...
     ```
5. **Never rewrite, reorder, or delete any existing entry or heading.** Only ever add new content at the appropriate point (end of file, or end of today's section).
6. **Check the ~30-entry archive threshold** (count top-level `## [YYYY-MM-DD]` headings under `## Entries`). If it's now over ~30, follow rule 5 in `CHANGELOG.md`: move everything older than the most recent 15 day-headings into `changelog/archive/[YYYY-MM].md` and update the archive pointer line near the top of the `## Entries` section. Do this as a separate concern from adding today's entry — don't skip appending the new entry just because archiving is also due.
7. Report back which case applied (new day heading vs. nested same-day sub-entry) and the file path changed. Do not commit or push automatically — leave that to the normal session workflow (README **Builder instructions** step 7 / **Operating rules** rule 12), unless the user explicitly asks this invocation to also commit.
