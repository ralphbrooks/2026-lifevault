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
- "publish" — copy `inbox/<current-month>.md` to Google Drive.

  Purpose:
  Publish the current month inbox file to Google Drive with minimum tokens, no git interaction, and local Ubuntu-verifiable checks.

  Operating assumption:
  Local shell examples are Ubuntu bash only. They are for local verification of filename, path, existence, byte count, and content. Google Drive actions must be done through the available Drive tool, not through local shell, unless an actual Drive CLI is installed and configured.

  Variables:

  Human:
  Current month filename is `YYYY-MM.md`.

  Ubuntu:
  `FILE_NAME="$(date +%Y-%m).md"`

  Human:
  Current inbox file is `inbox/YYYY-MM.md`.

  Ubuntu:
  `LOCAL_FILE="inbox/$FILE_NAME"`

  Human:
  Target Drive folder is `2026-lifevault-published`.

  Ubuntu:
  `PUBLISH_FOLDER="2026-lifevault-published"`

  Procedure:

  1. Human:
     Determine the current month filename.

     Ubuntu:
     `FILE_NAME="$(date +%Y-%m).md"`

  2. Human:
     Determine the local inbox file path.

     Ubuntu:
     `LOCAL_FILE="inbox/$FILE_NAME"`

  3. Human:
     Confirm the local inbox file exists. If missing, stop and report the missing path.

     Ubuntu:
     `test -f "$LOCAL_FILE" && echo "exists: $LOCAL_FILE" || echo "missing: $LOCAL_FILE"`

  4. Human:
     Count the local file size in bytes.

     Ubuntu:
     `BYTES="$(wc -c < "$LOCAL_FILE" | tr -d ' ')" && echo "$BYTES"`

  5. Human:
     Optionally preview the first few lines without modifying the file.

     Ubuntu:
     `sed -n '1,20p' "$LOCAL_FILE"`

  6. Human:
     In Google Drive root, find folders matching all of these:
     - name is `2026-lifevault-published`
     - mimeType is Google Drive folder
     - trashed is false
     - parent is root

     Drive action:
     Use the Drive tool to list matching root folders.

  7. Human:
     Choose the Drive folder:
     - If zero folders exist, create `2026-lifevault-published` at Drive root.
     - If one folder exists, use it.
     - If multiple folders exist, use the oldest by createdTime.
     - Do not delete duplicate folders.
     - Do not create another folder when one already exists.

     Drive action:
     Set chosen folder ID internally.

  8. Human:
     In the chosen Drive folder, find all non-trashed files named `YYYY-MM.md`.

     Drive action:
     List files in chosen folder where name equals the current month filename.

  9. Human:
     Delete every matching Drive file. Verify each delete succeeded.

     Drive action:
     Delete all matching files and count deletions.

  10. Human:
      Create one new Drive file in the chosen folder:
      - file name: current month filename
      - content: exact contents of local file

      Ubuntu local verification:
      `cat "$LOCAL_FILE"`

      Drive action:
      Create the Drive file using the exact local file content.

  11. Human:
      Reply with exactly these lines:

      Folder: 2026-lifevault-published (used existing | created new)
      Deleted: <N> existing copies
      Created: <YYYY-MM.md> (<bytes> bytes)

  12. Human:
      If duplicate folders were detected, append exactly this line:

      Note: <N> duplicate folders detected, used oldest. Clean up manually.

  Forbidden:
  - Do not run git.
  - Do not read git logs.
  - Do not run git status.
  - Do not run git diff.
  - Do not run git add.
  - Do not run git commit.
  - Do not run git push.
  - Do not modify anything in inbox/.
  - Do not create another Drive folder if one already exists.
  - Do not ask questions.

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.