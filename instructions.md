# Project: 2026-lifevault

## What this is

A personal log system.

The local folder is Ralph's source of truth.

Google Drive is the published/archive destination.

Git is Ralph's external backup/version-control system.

Ralph may use git whenever he wants.

Claude never touches git.

Claude never reads git logs.

Claude never runs git commands.

## Ownership model

Ralph owns and edits:

- `inbox/YYYY-MM.md`

Claude reads from:

- `inbox/YYYY-MM.md`

Claude writes only to:

- `archive/raw/YYYY/YYYY-MM.jsonl`
- `archive/state/YYYY-MM.state.json`
- `archive/derived/wins.md`
- `archive/derived/state-of-ralph.md`
- `archive/derived/weekly/YYYY-WXX.md`

Claude never modifies files in `inbox/`.

Claude never modifies git state.

## Git model

Ralph uses git as his external copy / backup after adding logs.

This is Ralph's workflow only.

Claude must not use git for:

- detecting changed lines
- finding diffs
- reading history
- checking status
- committing
- pushing
- restoring files
- any other operation

If Claude thinks it needs git, it does not.

Claude uses only local files and `archive/state/*.state.json` for processing state.

## Google Drive publishing model

Publishing is done with Ubuntu bash and `rclone`.

For the `publish` command, Ubuntu/rclone is the source of truth.

Claude must use the same command-line path Ralph can verify.

For `publish`, Claude must not use Claude's Google Drive tool.

Required local setup:

- Ubuntu bash
- `jq`
- current `rclone`
- an rclone Google Drive remote named `gdrive`

One-time setup check:

```bash
rclone version
rclone help lsjson
rclone lsd gdrive: >/dev/null
echo "rclone gdrive remote works"
```

If `rclone help lsjson` fails, upgrade rclone:

```bash
curl https://rclone.org/install.sh | sudo bash
```

## Hard rules

1. Never touch git.

   Forbidden:
   - `git status`
   - `git log`
   - `git add`
   - `git commit`
   - `git push`
   - `git diff`
   - `git checkout`
   - `git restore`
   - `git reset`
   - any git command

2. Never modify files in `inbox/`.

3. Never publish raw `inbox/` files unless Ralph explicitly asks for raw inbox backup.

4. Publishing means:
   - first incrementally process the current inbox into `archive/raw/YYYY/YYYY-MM.jsonl`
   - then publish the processed archive file to Google Drive

5. Claude writes only inside `archive/`.

6. For `publish`, do not use Claude's Google Drive tool.

7. For `publish`, use Ubuntu bash and rclone only.

8. Do not invent helper functions.

9. Do not ask questions during command execution unless a required file, command, or rclone remote is missing.

10. If a command fails, stop and report the exact failure.

---

# Processing state

Claude maintains local processing checkpoints in:

```text
archive/state/YYYY-MM.state.json
```

This file is owned by Claude.

Purpose:

Avoid reprocessing the full monthly inbox every time.

State file shape:

```json
{
  "source_file": "inbox/YYYY-MM.md",
  "source_size_bytes": 0,
  "source_sha256": "",
  "processed_through_byte": 0,
  "last_processed_at": "YYYY-MM-DDTHH:MM:SS",
  "raw_file": "archive/raw/YYYY/YYYY-MM.jsonl"
}
```

Rules:

1. `inbox/YYYY-MM.md` is append-only.

2. On `process inbox`, if the state file exists, read only bytes after `processed_through_byte`.

3. If the inbox file is smaller than `processed_through_byte`, stop. This means the inbox was rewritten or truncated.

4. If historical inbox content appears to have changed, stop instead of guessing.

5. Append only new JSONL rows to `archive/raw/YYYY/YYYY-MM.jsonl`.

6. Update `archive/state/YYYY-MM.state.json` only after successful parsing and successful JSONL validation.

7. Do not use git for change detection.

8. If no state file exists, process the full current-month inbox and create the state file.

9. If the unprocessed tail is empty, do not rewrite the raw JSONL file. Report that there are no new entries to process.

---

# Commands

- `"process inbox"` — incrementally read the current month's `inbox/YYYY-MM.md`, parse new log content, append new rows to `archive/raw/YYYY/YYYY-MM.jsonl`, and update `archive/state/YYYY-MM.state.json`. Do not modify `inbox/`. Do not run git.

- `"wins"` — regenerate `archive/derived/wins.md` from available archive data.

- `"weekly review"` — generate `archive/derived/weekly/YYYY-WXX.md` from the past 7 days.

- `"state of ralph"` — update `archive/derived/state-of-ralph.md`.

- `"publish"` — incrementally process the current inbox, then publish the processed archive output to Google Drive using Ubuntu bash and rclone.

---

# Command: process inbox

## Purpose

Convert new content from the current month's hand-written inbox log into structured JSONL.

This command is incremental.

It should not reprocess the full monthly log if `archive/state/YYYY-MM.state.json` already exists and the inbox is append-only.

## Input

Current month inbox file:

```bash
FILE_MONTH="$(date +%Y-%m)"
INBOX_FILE="inbox/$FILE_MONTH.md"
```

Example:

```text
inbox/2026-04.md
```

## Outputs

Processed archive file:

```bash
YEAR="$(date +%Y)"
RAW_DIR="archive/raw/$YEAR"
RAW_FILE="$RAW_DIR/$FILE_MONTH.jsonl"
```

Example:

```text
archive/raw/2026/2026-04.jsonl
```

Processing state file:

```bash
STATE_DIR="archive/state"
STATE_FILE="$STATE_DIR/$FILE_MONTH.state.json"
```

Example:

```text
archive/state/2026-04.state.json
```

## JSONL row shape

Each row should be one valid JSON object on one line.

Use this structure unless Ralph later changes the schema:

```json
{
  "date": "YYYY-MM-DD",
  "time": "HH:MM",
  "source_file": "inbox/YYYY-MM.md",
  "entry_type": "log",
  "text": "...",
  "tags": [],
  "signals": {
    "win": false,
    "stress": false,
    "legal": false,
    "generator": false,
    "money": false,
    "health": false,
    "relationship": false,
    "business": false
  }
}
```

If exact time is missing, use `null`.

If exact date is missing but can be inferred from surrounding context, infer it.

If date cannot be inferred, use `null` and preserve the text.

## Human / Ubuntu / Claude Specific execution pattern

For this command, Claude performs the AI parsing work.

Ubuntu commands are used for file verification and state mechanics.

### 1. Set current month.

Human:
Determine current month.

Ubuntu:
```bash
FILE_MONTH="$(date +%Y-%m)"
echo "$FILE_MONTH"
```

Claude Specific:
Use the current month in `YYYY-MM` format.

### 2. Set inbox, raw, and state paths.

Human:
Set all local paths.

Ubuntu:
```bash
YEAR="$(date +%Y)"
INBOX_FILE="inbox/$FILE_MONTH.md"
RAW_DIR="archive/raw/$YEAR"
RAW_FILE="$RAW_DIR/$FILE_MONTH.jsonl"
STATE_DIR="archive/state"
STATE_FILE="$STATE_DIR/$FILE_MONTH.state.json"

echo "$INBOX_FILE"
echo "$RAW_FILE"
echo "$STATE_FILE"
```

Claude Specific:
Use these exact paths.

### 3. Verify inbox exists.

Human:
Stop if the inbox file is missing.

Ubuntu:
```bash
test -f "$INBOX_FILE" || { echo "missing: $INBOX_FILE"; exit 1; }
echo "exists: $INBOX_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If missing, stop.

### 4. Create archive directories.

Human:
Create Claude-owned archive folders if missing.

Ubuntu:
```bash
mkdir -p "$RAW_DIR" "$STATE_DIR"
```

Claude Specific:
Run the Ubuntu command exactly.

### 5. Get current inbox size and hash.

Human:
Measure current source file.

Ubuntu:
```bash
SOURCE_SIZE_BYTES="$(wc -c < "$INBOX_FILE" | tr -d ' ')"
SOURCE_SHA256="$(sha256sum "$INBOX_FILE" | awk '{print $1}')"

echo "$SOURCE_SIZE_BYTES"
echo "$SOURCE_SHA256"
```

Claude Specific:
Run the Ubuntu command exactly.

### 6. Determine processed byte offset.

Human:
If state exists, use prior processed byte offset. Otherwise start at 0.

Ubuntu:
```bash
if test -f "$STATE_FILE"; then
  PROCESSED_THROUGH_BYTE="$(jq -r '.processed_through_byte // 0' "$STATE_FILE")"
else
  PROCESSED_THROUGH_BYTE="0"
fi

echo "$PROCESSED_THROUGH_BYTE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 7. Guard against truncation or rewrite.

Human:
If the current inbox is smaller than the processed offset, stop.

Ubuntu:
```bash
if [ "$SOURCE_SIZE_BYTES" -lt "$PROCESSED_THROUGH_BYTE" ]; then
  echo "inbox appears truncated or rewritten: $INBOX_FILE"
  exit 1
fi
```

Claude Specific:
Run the Ubuntu command exactly. If it fails, stop.

### 8. Extract unprocessed tail.

Human:
Read only new content after the processed offset.

Ubuntu:
```bash
TAIL_FILE="/tmp/lifevault-$FILE_MONTH-tail.md"

tail -c +"$((PROCESSED_THROUGH_BYTE + 1))" "$INBOX_FILE" > "$TAIL_FILE"

TAIL_BYTES="$(wc -c < "$TAIL_FILE" | tr -d ' ')"
echo "$TAIL_FILE"
echo "$TAIL_BYTES"
```

Claude Specific:
Run the Ubuntu command exactly.

### 9. Stop cleanly if there is no new content.

Human:
If no new bytes exist, do not rewrite archive files.

Ubuntu:
```bash
if [ "$TAIL_BYTES" = "0" ]; then
  echo "no new inbox content to process"
fi
```

Claude Specific:
If `TAIL_BYTES` is 0, do not rewrite `RAW_FILE`. Report no new content and stop unless this is part of `publish`; for `publish`, continue to publish the existing valid `RAW_FILE`.

### 10. Process new tail content.

Human:
Parse only the new tail content into JSONL rows.

Ubuntu:
```bash
# Verification only:
sed -n '1,80p' "$TAIL_FILE"
```

Claude Specific:
If `TAIL_BYTES` is greater than 0:
- read `TAIL_FILE`
- parse only this new content
- append new JSONL rows to `RAW_FILE`
- do not reprocess prior inbox content
- do not modify `INBOX_FILE`
- do not run git

### 11. Verify JSONL exists.

Human:
Confirm the processed file exists.

Ubuntu:
```bash
test -f "$RAW_FILE" || { echo "missing: $RAW_FILE"; exit 1; }
echo "exists: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 12. Validate JSONL.

Human:
Every line must be valid JSON.

Ubuntu:
```bash
jq -c . "$RAW_FILE" >/dev/null && echo "valid jsonl: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If validation fails, fix `RAW_FILE` before continuing.

### 13. Update processing state.

Human:
Update state only after successful parsing and JSONL validation.

Ubuntu:
```bash
LAST_PROCESSED_AT="$(date -Iseconds)"

jq -n \
  --arg source_file "$INBOX_FILE" \
  --arg source_sha256 "$SOURCE_SHA256" \
  --arg raw_file "$RAW_FILE" \
  --arg last_processed_at "$LAST_PROCESSED_AT" \
  --argjson source_size_bytes "$SOURCE_SIZE_BYTES" \
  --argjson processed_through_byte "$SOURCE_SIZE_BYTES" \
  '{
    source_file: $source_file,
    source_size_bytes: $source_size_bytes,
    source_sha256: $source_sha256,
    processed_through_byte: $processed_through_byte,
    last_processed_at: $last_processed_at,
    raw_file: $raw_file
  }' > "$STATE_FILE"

cat "$STATE_FILE"
```

Claude Specific:
Run the Ubuntu command exactly after `RAW_FILE` validates.

---

# Command: publish

## Purpose

First incrementally process the current inbox into `archive/raw/YYYY/YYYY-MM.jsonl`.

Then publish that processed archive file to Google Drive using Ubuntu bash and rclone.

Do not publish raw `inbox/`.

Do not use Claude's Google Drive tool.

Do not use git.

## Target Drive folder

```text
2026-lifevault-published
```

This folder lives at Google Drive root under the `gdrive:` rclone remote.

## Published file

The published file is the processed archive file:

```text
archive/raw/YYYY/YYYY-MM.jsonl
```

It is uploaded to Drive as:

```text
2026-lifevault-published/YYYY-MM.jsonl
```

## Required behavior

- Run incremental `process inbox` first.
- Publish only after the processed JSONL file exists and validates.
- Always overwrite the published file.
- Delete existing Drive copies with the same filename before upload.
- Create the Drive publish folder only if missing.
- Do not upload `inbox/YYYY-MM.md`.

## Claude Specific

For this command:

- Run the Ubuntu commands exactly.
- Perform the AI parsing step only where explicitly instructed.
- Do not use Claude's Google Drive tool.
- Do not invent helper functions.
- Do not run git.
- Do not modify `inbox/`.
- If a command fails, stop and report the failure.

## Procedure

### 1. Set current month.

Human:
Current month key is `YYYY-MM`.

Ubuntu:
```bash
FILE_MONTH="$(date +%Y-%m)"
echo "$FILE_MONTH"
```

Claude Specific:
Run the Ubuntu command exactly.

### 2. Set local paths.

Human:
Set inbox, raw archive, state, and output paths.

Ubuntu:
```bash
YEAR="$(date +%Y)"
INBOX_FILE="inbox/$FILE_MONTH.md"
RAW_DIR="archive/raw/$YEAR"
RAW_FILE="$RAW_DIR/$FILE_MONTH.jsonl"
STATE_DIR="archive/state"
STATE_FILE="$STATE_DIR/$FILE_MONTH.state.json"

echo "$INBOX_FILE"
echo "$RAW_FILE"
echo "$STATE_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 3. Confirm inbox file exists.

Human:
If the inbox file is missing, stop.

Ubuntu:
```bash
test -f "$INBOX_FILE" || { echo "missing: $INBOX_FILE"; exit 1; }
echo "exists: $INBOX_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If missing, stop.

### 4. Create archive directories.

Human:
Create Claude-owned archive directories if missing.

Ubuntu:
```bash
mkdir -p "$RAW_DIR" "$STATE_DIR"
```

Claude Specific:
Run the Ubuntu command exactly.

### 5. Get current inbox size and hash.

Human:
Measure source inbox file.

Ubuntu:
```bash
SOURCE_SIZE_BYTES="$(wc -c < "$INBOX_FILE" | tr -d ' ')"
SOURCE_SHA256="$(sha256sum "$INBOX_FILE" | awk '{print $1}')"

echo "$SOURCE_SIZE_BYTES"
echo "$SOURCE_SHA256"
```

Claude Specific:
Run the Ubuntu command exactly.

### 6. Determine processed byte offset.

Human:
If state exists, use prior processed byte offset. Otherwise start at 0.

Ubuntu:
```bash
if test -f "$STATE_FILE"; then
  PROCESSED_THROUGH_BYTE="$(jq -r '.processed_through_byte // 0' "$STATE_FILE")"
else
  PROCESSED_THROUGH_BYTE="0"
fi

echo "$PROCESSED_THROUGH_BYTE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 7. Guard against truncation or rewrite.

Human:
If the current inbox is smaller than the processed offset, stop.

Ubuntu:
```bash
if [ "$SOURCE_SIZE_BYTES" -lt "$PROCESSED_THROUGH_BYTE" ]; then
  echo "inbox appears truncated or rewritten: $INBOX_FILE"
  exit 1
fi
```

Claude Specific:
Run the Ubuntu command exactly. If it fails, stop.

### 8. Extract unprocessed tail.

Human:
Read only new content after the processed offset.

Ubuntu:
```bash
TAIL_FILE="/tmp/lifevault-$FILE_MONTH-tail.md"

tail -c +"$((PROCESSED_THROUGH_BYTE + 1))" "$INBOX_FILE" > "$TAIL_FILE"

TAIL_BYTES="$(wc -c < "$TAIL_FILE" | tr -d ' ')"
echo "$TAIL_FILE"
echo "$TAIL_BYTES"
```

Claude Specific:
Run the Ubuntu command exactly.

### 9. Process new content if present.

Human:
If there is new content, parse and append it to the processed JSONL file.

Ubuntu:
```bash
if [ "$TAIL_BYTES" = "0" ]; then
  echo "no new inbox content to process"
else
  echo "new inbox content found: $TAIL_BYTES bytes"
  sed -n '1,80p' "$TAIL_FILE"
fi
```

Claude Specific:
If `TAIL_BYTES` is greater than 0:
- read `TAIL_FILE`
- parse only this new content
- append new JSONL rows to `RAW_FILE`
- do not reprocess prior inbox content
- do not modify `INBOX_FILE`
- do not run git

If `TAIL_BYTES` is 0:
- do not rewrite `RAW_FILE`
- continue only if `RAW_FILE` already exists and validates

### 10. Verify processed archive file exists.

Human:
The processed JSONL file must exist before publishing.

Ubuntu:
```bash
test -f "$RAW_FILE" || { echo "missing processed file: $RAW_FILE"; exit 1; }
echo "exists: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 11. Validate JSONL.

Human:
Every line in the processed file must be valid JSON.

Ubuntu:
```bash
jq -c . "$RAW_FILE" >/dev/null && echo "valid jsonl: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If validation fails, stop and fix `RAW_FILE`.

### 12. Update processing state.

Human:
Update state after successful parsing and validation.

Ubuntu:
```bash
LAST_PROCESSED_AT="$(date -Iseconds)"

jq -n \
  --arg source_file "$INBOX_FILE" \
  --arg source_sha256 "$SOURCE_SHA256" \
  --arg raw_file "$RAW_FILE" \
  --arg last_processed_at "$LAST_PROCESSED_AT" \
  --argjson source_size_bytes "$SOURCE_SIZE_BYTES" \
  --argjson processed_through_byte "$SOURCE_SIZE_BYTES" \
  '{
    source_file: $source_file,
    source_size_bytes: $source_size_bytes,
    source_sha256: $source_sha256,
    processed_through_byte: $processed_through_byte,
    last_processed_at: $last_processed_at,
    raw_file: $raw_file
  }' > "$STATE_FILE"

cat "$STATE_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 13. Count processed file bytes.

Human:
Get exact byte count of the processed JSONL file.

Ubuntu:
```bash
BYTES="$(wc -c < "$RAW_FILE" | tr -d ' ')"
echo "$BYTES"
```

Claude Specific:
Run the Ubuntu command exactly.

### 14. Confirm rclone remote works.

Human:
Verify Google Drive remote named `gdrive`.

Ubuntu:
```bash
rclone lsd gdrive: >/dev/null
echo "rclone gdrive remote works"
```

Claude Specific:
Run the Ubuntu command exactly. If it fails, stop and report that `gdrive` remote is not configured.

### 15. Set publish folder and publish filename.

Human:
Set Drive folder and output filename.

Ubuntu:
```bash
PUBLISH_FOLDER="2026-lifevault-published"
PUBLISH_FILE="$FILE_MONTH.jsonl"

echo "$PUBLISH_FOLDER"
echo "$PUBLISH_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 16. Ensure publish folder exists.

Human:
Use the Drive root folder named `2026-lifevault-published`.
Create it only if missing.

Ubuntu:
```bash
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

### 17. Count existing published copies.

Human:
Count existing files in the publish folder with the same published filename.

Ubuntu:
```bash
DELETE_COUNT="$(
  rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
    | jq --arg name "$PUBLISH_FILE" '[.[] | select(.Name == $name)] | length'
)"

echo "$DELETE_COUNT"
```

Claude Specific:
Run the Ubuntu command exactly.

### 18. Delete existing published copies.

Human:
Delete every existing Drive file in the publish folder with the same filename.

Ubuntu:
```bash
rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
  | jq -r --arg name "$PUBLISH_FILE" '.[] | select(.Name == $name) | .Name' \
  | while IFS= read -r MATCHED_FILE; do
      rclone deletefile "gdrive:$PUBLISH_FOLDER/$MATCHED_FILE"
    done
```

Claude Specific:
Run the Ubuntu command exactly.

### 19. Verify old copies are gone.

Human:
Confirm zero matching published files remain before upload.

Ubuntu:
```bash
REMAINING_COUNT="$(
  rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
    | jq --arg name "$PUBLISH_FILE" '[.[] | select(.Name == $name)] | length'
)"

test "$REMAINING_COUNT" = "0" || { echo "delete failed: $REMAINING_COUNT copies remain"; exit 1; }
echo "deleted old copies"
```

Claude Specific:
Run the Ubuntu command exactly. If any matching copies remain, stop.

### 20. Upload processed archive file.

Human:
Upload `archive/raw/YYYY/YYYY-MM.jsonl` to Drive as `YYYY-MM.jsonl`.

Ubuntu:
```bash
rclone copyto "$RAW_FILE" "gdrive:$PUBLISH_FOLDER/$PUBLISH_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 21. Verify uploaded file exists.

Human:
Confirm exactly one published file now exists.

Ubuntu:
```bash
CREATED_COUNT="$(
  rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
    | jq --arg name "$PUBLISH_FILE" '[.[] | select(.Name == $name)] | length'
)"

test "$CREATED_COUNT" = "1" || { echo "upload verification failed: $CREATED_COUNT copies found"; exit 1; }
echo "uploaded: $PUBLISH_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If exactly one uploaded file is not found, stop.

### 22. Final reply.

Human:
Reply only with the publish result.

Ubuntu:
```bash
echo "Folder: $PUBLISH_FOLDER ($FOLDER_STATUS)"
echo "Deleted: $DELETE_COUNT existing copies"
echo "Created: $PUBLISH_FILE ($BYTES bytes)"
```

Claude Specific:
Reply with exactly the three echoed lines.

Expected final format:

```text
Folder: 2026-lifevault-published (used existing | created new)
Deleted: <N> existing copies
Created: <YYYY-MM.jsonl> (<bytes> bytes)
```

---

# Command: wins

## Purpose

Regenerate the wins summary from available processed archive data and inbox context.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `inbox/*.md` only if archive data is incomplete or missing

## Output

```text
archive/derived/wins.md
```

## Procedure

1. Read processed archive data.
2. Identify wins, progress, resilience, completed tasks, recovered problems, and business/legal/technical breakthroughs.
3. Write `archive/derived/wins.md`.
4. Do not modify `inbox/`.
5. Do not run git.

## Output style

Direct. Specific. No motivational fluff unless Ralph asks.

---

# Command: weekly review

## Purpose

Generate a weekly review from the past 7 days of processed logs.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `inbox/*.md` only if archive data is incomplete or missing

## Output

```text
archive/derived/weekly/YYYY-WXX.md
```

## Procedure

1. Determine the current ISO week.
2. Read entries from the past 7 days.
3. Summarize:
   - wins
   - problems
   - decisions
   - open loops
   - risks
   - next actions
4. Write the weekly review file.
5. Do not modify `inbox/`.
6. Do not run git.

---

# Command: state of ralph

## Purpose

Update Ralph's current operating state from processed logs.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `archive/derived/weekly/*.md`
3. `inbox/*.md` only if needed

## Output

```text
archive/derived/state-of-ralph.md
```

## Procedure

Update the file with:

- current priorities
- active constraints
- stressors
- wins
- risks
- projects
- open loops
- tactical next actions

Do not modify `inbox/`.

Do not run git.

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.
