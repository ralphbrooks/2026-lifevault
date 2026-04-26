# Project: 2026-lifevault

## What this is

A personal log. The local folder is mine. Google Drive is working
memory you maintain. You never touch git. You never read git logs.
You never run git commands. Git is mine.

## Folder you maintain

Google Drive folder ID: <PASTE FOLDER ID HERE>
This is the only folder you write to in Drive. Never search for it
by name. Never create another folder. Always reference it by ID.

## Files I write

inbox/YYYY-MM.md — I edit these by hand on my PC. Append-only.

## Files you write

archive/raw/YYYY/YYYY-MM.jsonl — structured rows you parse from inbox.
archive/derived/wins.md, state-of-ralph.md, weekly/*.md — synthesis.

You only write to archive/. You never modify inbox.

## Hard rules

1. NEVER touch git. No git status, git log, git add, git commit, git
   push, git diff, git anything. The local folder is the source. Read
   from it directly. If you think you need git, you don't.

2. NEVER modify files in inbox/. Read only.

3. NEVER create folders in Drive. The folder ID above is the only
   Drive folder you ever write to.

4. NEVER search Drive for folders by name. Use the ID.

5. When publishing, always overwrite. Find existing file by name in
   the target folder, delete it, create the new one.

## Commands

- "process inbox" — read the current month's inbox/YYYY-MM.md, parse
  new lines into archive/raw/YYYY/YYYY-MM.jsonl. Do not touch the
  inbox. Do not run git.

- "wins" — regenerate archive/derived/wins.md from all inbox files.

- "weekly review" — generate archive/derived/weekly/YYYY-WXX.md from
  the past 7 days.

- "state of ralph" — update archive/derived/state-of-ralph.md.

- "publish" — sync inbox/<current-month>.md to a single Google Drive
  folder named "2026-lifevault-published" at Drive root.

  Procedure (no deviations, no questions):

  1. List items at Drive root where:
       - mimeType == 'application/vnd.google-apps.folder'
       - name == '2026-lifevault-published'
       - trashed == false
       - parents contains 'root'
     Sort by createdTime ascending.
     
     - If 0 results: create the folder at root. Use that ID.
     - If 1 result: use that ID.
     - If 2+ results: use the OLDEST one (first in sorted list).
       Do not delete the others. Do not stop. Just use the oldest.
       Report at the end: "Note: N duplicate folders detected,
       used oldest. Clean up manually."

  2. List files in the chosen folder where:
       - name == '<current-month>.md'
       - trashed == false
     Delete every match. Verify each delete succeeded.

  3. Read inbox/<current-month>.md from local repo.
     Create a new file in the chosen folder with that name and content.

  4. Reply with exactly:
       "Folder: <name> (used existing | created new)"
       "Deleted: <N> existing copies"
       "Created: <filename> (<bytes> bytes)"

  Forbidden: creating a second folder when one exists, asking
  questions, paginating beyond one call, running git, reading git
  history, modifying inbox files.

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.