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
- "publish" — copy `inbox/<current-month>.md` to Google Drive using Ubuntu bash and rclone.

  Purpose:
  Publish the current month inbox file to Google Drive with no git interaction.
  Ubuntu/rclone is the source of truth. Claude must use the same command-line path Ralph can verify.

  Required local setup:
  `rclone` must have a Google Drive remote named `gdrive`.

  Claude Specific:
  For this command, do not use Claude's Google Drive tool.
  Use Ubuntu bash commands only.
  Run the same commands shown below.
  Do not invent helper functions.
  Do not run git.

  1. Set current month filename.

     Human:
     Current month filename is `YYYY-MM.md`.

     Ubuntu:
     ```bash
     FILE_NAME="$(date +%Y-%m).md"
     echo "$FILE_NAME"
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  2. Set local inbox file path.

     Human:
     Local file is `inbox/YYYY-MM.md`.

     Ubuntu:
     ```bash
     LOCAL_FILE="inbox/$FILE_NAME"
     echo "$LOCAL_FILE"
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  3. Confirm local file exists.

     Human:
     If the local inbox file is missing, stop.

     Ubuntu:
     ```bash
     test -f "$LOCAL_FILE" || { echo "missing: $LOCAL_FILE"; exit 1; }
     echo "exists: $LOCAL_FILE"
     ```

     Claude Specific:
     Run the Ubuntu command exactly. If missing, stop.

  4. Count local file bytes.

     Human:
     Get exact byte count.

     Ubuntu:
     ```bash
     BYTES="$(wc -c < "$LOCAL_FILE" | tr -d ' ')"
     echo "$BYTES"
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  5. Confirm rclone remote exists.

     Human:
     Verify the Google Drive remote named `gdrive` works.

     Ubuntu:
     ```bash
     rclone lsd gdrive: >/dev/null
     echo "rclone gdrive remote works"
     ```

     Claude Specific:
     Run the Ubuntu command exactly. If it fails, stop and report that `gdrive` remote is not configured.

  6. Ensure publish folder exists.

     Human:
     Use the Drive root folder named `2026-lifevault-published`.
     Create it only if missing.
     Do not create another folder when it already exists.

     Ubuntu:
     ```bash
     PUBLISH_FOLDER="2026-lifevault-published"

     if rclone lsjson --dirs-only gdrive: \
       | jq -e --arg name "$PUBLISH_FOLDER" '.[] | select(.Name == $name)' >/dev/null; then
       FOLDER_STATUS="used existing"
     else
       rclone mkdir "gdrive:$PUBLISH_FOLDER"
       FOLDER_STATUS="created new"
     fi

     echo "$FOLDER_STATUS"
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  7. Count existing published copies.

     Human:
     Count existing files in the publish folder with the current month filename.

     Ubuntu:
     ```bash
     DELETE_COUNT="$(
       rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
         | jq --arg name "$FILE_NAME" '[.[] | select(.Name == $name)] | length'
     )"

     echo "$DELETE_COUNT"
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  8. Delete existing published copies.

     Human:
     Delete every existing file in the publish folder with the current month filename.

     Ubuntu:
     ```bash
     rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
       | jq -r --arg name "$FILE_NAME" '.[] | select(.Name == $name) | .Name' \
       | while IFS= read -r MATCHED_FILE; do
           rclone deletefile "gdrive:$PUBLISH_FOLDER/$MATCHED_FILE"
         done
     ```

     Claude Specific:
     Run the Ubuntu command exactly.

  9. Verify old published copies are gone.

     Human:
     Confirm there are now zero existing files with the current month filename.

     Ubuntu:
     ```bash
     REMAINING_COUNT="$(
       rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
         | jq --arg name "$FILE_NAME" '[.[] | select(.Name == $name)] | length'
     )"

     test "$REMAINING_COUNT" = "0" || { echo "delete failed: $REMAINING_COUNT copies remain"; exit 1; }
     echo "deleted old copies"
     ```

     Claude Specific:
     Run the Ubuntu command exactly. If any matching copies remain, stop.

  10. Upload current inbox file.

      Human:
      Upload the local inbox file to the Drive publish folder.

      Ubuntu:
      ```bash
      rclone copyto "$LOCAL_FILE" "gdrive:$PUBLISH_FOLDER/$FILE_NAME"
      ```

      Claude Specific:
      Run the Ubuntu command exactly.

  11. Verify uploaded file exists.

      Human:
      Confirm exactly one current-month file exists in the publish folder.

      Ubuntu:
      ```bash
      CREATED_COUNT="$(
        rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
          | jq --arg name "$FILE_NAME" '[.[] | select(.Name == $name)] | length'
      )"

      test "$CREATED_COUNT" = "1" || { echo "upload verification failed: $CREATED_COUNT copies found"; exit 1; }
      echo "uploaded: $FILE_NAME"
      ```

      Claude Specific:
      Run the Ubuntu command exactly. If exactly one uploaded file is not found, stop.

  12. Final reply.

      Human:
      Reply with only the publish result.

      Ubuntu:
      ```bash
      echo "Folder: $PUBLISH_FOLDER ($FOLDER_STATUS)"
      echo "Deleted: $DELETE_COUNT existing copies"
      echo "Created: $FILE_NAME ($BYTES bytes)"
      ```

      Claude Specific:
      Reply with exactly the three echoed lines.

  Forbidden:
  - Do not use Claude's Google Drive tool for publish.
  - Do not run git.
  - Do not read git logs.
  - Do not run git status.
  - Do not run git diff.
  - Do not run git add.
  - Do not run git commit.
  - Do not run git push.
  - Do not modify anything in inbox/.
  - Do not ask questions.

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.