# Project: 2026-lifevault

## What this is

A personal log + working-memory system.

The local PC folder is Ralph's source of truth.

Git is Ralph's external backup/version-control system.

Google Drive is Claude's published working memory.

Claude never touches git.

Claude never reads git logs.

Claude never runs git commands.

## Goal

The workflow is:

1. Ralph edits logs on PC in `inbox/YYYY-MM.md`.
2. Ralph may commit/push those edits to git himself.
3. Ralph runs `"publish"`.
4. Claude processes only new log content from the current inbox.
5. Claude updates local processed archive files under `archive/`.
6. Claude regenerates working-memory summaries under `archive/derived/`.
7. Claude publishes the processed archive and working-memory files to Google Drive using Ubuntu bash + rclone.
8. Later, Ralph can ask Claude to reference Google Drive and answer questions such as:
   - "Tell me my wins."
   - "What has changed in my state?"
   - "What happened this week?"
   - "What are my open loops?"

## Core principle

AI performs semantic interpretation.

Script commands perform mechanical verification.

Claude should use AI judgment for converting human-language log text into structured JSONL and summaries.

Claude should use Ubuntu/rclone/jq commands to verify:
- files exist
- JSONL is valid
- every timestamped inbox entry was processed
- uploads completed
- expected Drive files exist

## Ownership model

Ralph owns and edits:

- `inbox/YYYY-MM.md`

Ralph may use git whenever he wants.

Claude reads from:

- `inbox/YYYY-MM.md`
- `archive/raw/**/*.jsonl`
- `archive/derived/**/*.md`
- `archive/state/*.state.json`

Claude writes only to:

- `archive/raw/YYYY/YYYY-MM.jsonl`
- `archive/state/YYYY-MM.state.json`
- `archive/derived/wins.md`
- `archive/derived/state-of-ralph.md`
- `archive/derived/weekly/YYYY-WXX.md`
- `archive/derived/manifest.md`

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
- resetting files
- any other operation

If Claude thinks it needs git, it does not.

Claude uses only local files and `archive/state/*.state.json` for processing state.

## Google Drive working-memory model

Google Drive is Claude's published working memory.

Publishing is done with Ubuntu bash and `rclone`.

For the `"publish"` command, Ubuntu/rclone is the source of truth.

Claude must use the same command-line path Ralph can verify.

For `"publish"`, Claude must not use Claude's Google Drive tool.

Claude's Google Drive connector may later be used in chat to read the published files after they have been uploaded by rclone.

If Claude cannot later find the files through the Google Drive connector, the issue is Drive connector access/indexing, not the local publish process.

## Required local setup

The local system must have:

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

## Google Drive publish folder

The Drive publish folder is:

```text
2026-lifevault-published
```

It lives at Google Drive root under the `gdrive:` rclone remote.

Published Drive layout:

```text
2026-lifevault-published/
  manifest.md
  raw/
    YYYY/
      YYYY-MM.jsonl
  derived/
    wins.md
    state-of-ralph.md
    weekly/
      YYYY-WXX.md
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
   - incrementally process the current inbox into `archive/raw/YYYY/YYYY-MM.jsonl`
   - regenerate working-memory files under `archive/derived/`
   - publish the processed archive and derived memory files to Google Drive

5. Claude writes only inside `archive/`.

6. For `"publish"`, do not use Claude's Google Drive tool.

7. For `"publish"`, use Ubuntu bash and rclone only.

8. Do not invent helper functions.

9. Do not ask questions during command execution unless a required file, command, or rclone remote is missing.

10. If a command fails, stop and report the exact failure.

---

# Inbox format

The inbox uses one log entry per physical line.

Ignore markdown heading lines such as:

```text
# Inbox — April 2026
```

Each nonblank line matching this pattern is one separate log entry:

```text
YYYY-MM-DD-HHMM [tag][tag] - body text
```

Example:

```text
2026-04-24-1200 [home-maintenance][win] - I successfully negotiated the plumber on the flange repair on the master toilet.
```

Parsing rules:

- `2026-04-24-1200` means:
  - `date = 2026-04-24`
  - `time = 12:00`
  - `ts = 2026-04-24T12:00`
- Each bracketed value becomes one tag.
- Text after ` - ` becomes the human-language body.
- One matching inbox line must produce exactly one JSONL row.
- Do not combine multiple matching inbox lines into one row.
- Do not skip matching inbox lines.
- AI may interpret the body semantically, but it must preserve the original meaning.
- AI may add normalized fields or signals when useful, but the raw body should remain available.

## Mechanical completeness rule

Every timestamped inbox entry must appear as exactly one JSONL row.

This is a mechanical validation rule, not a semantic rule.

Claude uses AI for meaning, but must use row-count validation to ensure it did not skip entries.

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

2. On `"process inbox"`, if the state file exists, read only bytes after `processed_through_byte`.

3. If the inbox file is smaller than `processed_through_byte`, stop. This means the inbox was rewritten or truncated.

4. If historical inbox content appears to have changed, stop instead of guessing.

5. Append only new JSONL rows to `archive/raw/YYYY/YYYY-MM.jsonl`.

6. Update `archive/state/YYYY-MM.state.json` only after successful parsing and successful JSONL validation.

7. Do not use git for change detection.

8. If no state file exists, process the full current-month inbox and create the state file.

9. If the unprocessed tail is empty, do not rewrite the raw JSONL file. Report that there are no new entries to process.

10. If the full raw JSONL row count does not match the full inbox timestamped-entry count, stop and fix the raw JSONL.

---

# Commands

- `"process inbox"` — incrementally read the current month's `inbox/YYYY-MM.md`, parse new log content, append new rows to `archive/raw/YYYY/YYYY-MM.jsonl`, validate completeness, and update `archive/state/YYYY-MM.state.json`. Do not modify `inbox/`. Do not run git.

- `"wins"` — regenerate `archive/derived/wins.md` from available archive data.

- `"weekly review"` — generate `archive/derived/weekly/YYYY-WXX.md` from the past 7 days.

- `"state of ralph"` — update `archive/derived/state-of-ralph.md`.

- `"manifest"` — update `archive/derived/manifest.md`, the entry point explaining how Claude should use the published working-memory files.

- `"publish"` — incrementally process the current inbox, validate completeness, regenerate working-memory files, and publish the processed archive and derived memory files to Google Drive using Ubuntu bash and rclone.

---

# Command: process inbox

## Purpose

Convert new content from the current month's hand-written inbox log into structured JSONL.

This command is incremental.

It should not reprocess the full monthly log if `archive/state/YYYY-MM.state.json` already exists and the inbox is append-only.

AI handles semantic interpretation.

Ubuntu/jq commands verify mechanical completeness.

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

Minimum required fields:

```json
{
  "ts": "YYYY-MM-DDTHH:MM",
  "date": "YYYY-MM-DD",
  "time": "HH:MM",
  "tags": [],
  "body": "..."
}
```

Recommended expanded fields when useful:

```json
{
  "ts": "YYYY-MM-DDTHH:MM",
  "date": "YYYY-MM-DD",
  "time": "HH:MM",
  "source_file": "inbox/YYYY-MM.md",
  "entry_type": "log",
  "tags": [],
  "body": "...",
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

Rules:

- `body` must preserve the meaning of the human-written entry.
- Tags from the inbox line must be preserved.
- Claude may add normalized fields or signals if useful.
- Exact timestamp parsing must follow the inbox format.
- One timestamped inbox line must produce exactly one JSONL row.

## Human / Ubuntu / Claude Specific execution pattern

For this command, Claude performs the AI parsing work.

Ubuntu commands are used for file verification, state mechanics, and completeness checks.

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
- one timestamped line equals one JSONL row
- do not reprocess prior inbox content
- do not modify `INBOX_FILE`
- do not run git

If `TAIL_BYTES` is 0:
- do not rewrite `RAW_FILE`
- continue only if `RAW_FILE` already exists and validates

### 10. Verify processed archive file exists.

Human:
The processed JSONL file must exist.

Ubuntu:
```bash
test -f "$RAW_FILE" || { echo "missing processed file: $RAW_FILE"; exit 1; }
echo "exists: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 11. Validate JSONL syntax.

Human:
Every line in the processed file must be valid JSON.

Ubuntu:
```bash
jq -c . "$RAW_FILE" >/dev/null && echo "valid jsonl: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If validation fails, stop and fix `RAW_FILE`.

### 12. Validate row-count completeness.

Human:
Every timestamped inbox entry must have one JSONL row.

Ubuntu:
```bash
EXPECTED_ROWS="$(grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}-[0-9]{4} ' "$INBOX_FILE" | wc -l | tr -d ' ')"
ACTUAL_ROWS="$(wc -l < "$RAW_FILE" | tr -d ' ')"

echo "expected rows: $EXPECTED_ROWS"
echo "actual rows: $ACTUAL_ROWS"

test "$EXPECTED_ROWS" = "$ACTUAL_ROWS" || {
  echo "row count mismatch: expected $EXPECTED_ROWS, got $ACTUAL_ROWS"
  exit 1
}
```

Claude Specific:
Run the Ubuntu command exactly. If row count mismatches, stop and fix `RAW_FILE`.
Do not update state until this passes.

### 13. Update processing state.

Human:
Update state only after successful parsing, JSONL validation, and row-count validation.

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
Run the Ubuntu command exactly after `RAW_FILE` validates and row counts match.

---

# Command: wins

## Purpose

Regenerate the wins summary from available processed archive data.

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

---

# Command: manifest

## Purpose

Update the entry-point file Claude should use later when reading Google Drive working memory.

## Output

```text
archive/derived/manifest.md
```

## Required contents

The manifest must explain:

```text
This folder is Ralph's published working memory.

Use these files:
- derived/wins.md for wins and progress.
- derived/state-of-ralph.md for current operating state.
- derived/weekly/*.md for weekly summaries.
- raw/YYYY/YYYY-MM.jsonl for detailed structured logs.

Do not treat the Google Drive folder as the source of truth.
The local PC folder is the source of truth.
The Google Drive folder is the published working-memory copy.
```

## Procedure

1. Create or update `archive/derived/manifest.md`.
2. Keep it concise.
3. Do not modify `inbox/`.
4. Do not run git.

---

# Command: publish

## Purpose

First incrementally process the current inbox into `archive/raw/YYYY/YYYY-MM.jsonl`.

Then validate that every timestamped inbox entry has one JSONL row.

Then regenerate working-memory files.

Then publish the processed archive and derived working-memory files to Google Drive using Ubuntu bash and rclone.

Do not publish raw `inbox/`.

Do not use Claude's Google Drive tool.

Do not use git.

## Required behavior

- Run incremental `"process inbox"` first.
- Validate JSONL syntax.
- Validate row-count completeness.
- Regenerate:
  - `archive/derived/wins.md`
  - `archive/derived/state-of-ralph.md`
  - `archive/derived/weekly/YYYY-WXX.md`
  - `archive/derived/manifest.md`
- Publish only after processed JSONL exists and validates.
- Always overwrite published files.
- Create the Drive publish folder/subfolders only if missing.
- Do not upload `inbox/YYYY-MM.md`.

## Claude Specific

For this command:

- Run the Ubuntu commands exactly.
- Perform AI parsing/synthesis only where explicitly instructed.
- Do not use Claude's Google Drive tool.
- Do not invent helper functions.
- Do not run git.
- Do not modify `inbox/`.
- If a command fails, stop and report the failure.

## Procedure

### 1. Set current month and week.

Human:
Set current month, year, and ISO week.

Ubuntu:
```bash
FILE_MONTH="$(date +%Y-%m)"
YEAR="$(date +%Y)"
ISO_WEEK="$(date +%G-W%V)"

echo "$FILE_MONTH"
echo "$YEAR"
echo "$ISO_WEEK"
```

Claude Specific:
Run the Ubuntu command exactly.

### 2. Set local paths.

Human:
Set inbox, raw archive, state, and derived output paths.

Ubuntu:
```bash
INBOX_FILE="inbox/$FILE_MONTH.md"
RAW_DIR="archive/raw/$YEAR"
RAW_FILE="$RAW_DIR/$FILE_MONTH.jsonl"
STATE_DIR="archive/state"
STATE_FILE="$STATE_DIR/$FILE_MONTH.state.json"
DERIVED_DIR="archive/derived"
WEEKLY_DIR="$DERIVED_DIR/weekly"
WINS_FILE="$DERIVED_DIR/wins.md"
STATE_OF_RALPH_FILE="$DERIVED_DIR/state-of-ralph.md"
WEEKLY_FILE="$WEEKLY_DIR/$ISO_WEEK.md"
MANIFEST_FILE="$DERIVED_DIR/manifest.md"

echo "$INBOX_FILE"
echo "$RAW_FILE"
echo "$STATE_FILE"
echo "$WINS_FILE"
echo "$STATE_OF_RALPH_FILE"
echo "$WEEKLY_FILE"
echo "$MANIFEST_FILE"
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

### 4. Create local archive directories.

Human:
Create Claude-owned archive directories if missing.

Ubuntu:
```bash
mkdir -p "$RAW_DIR" "$STATE_DIR" "$DERIVED_DIR" "$WEEKLY_DIR"
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
- one timestamped line equals one JSONL row
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

### 11. Validate JSONL syntax.

Human:
Every line in the processed file must be valid JSON.

Ubuntu:
```bash
jq -c . "$RAW_FILE" >/dev/null && echo "valid jsonl: $RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly. If validation fails, stop and fix `RAW_FILE`.

### 12. Validate row-count completeness.

Human:
Every timestamped inbox entry must have one JSONL row.

Ubuntu:
```bash
EXPECTED_ROWS="$(grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}-[0-9]{4} ' "$INBOX_FILE" | wc -l | tr -d ' ')"
ACTUAL_ROWS="$(wc -l < "$RAW_FILE" | tr -d ' ')"

echo "expected rows: $EXPECTED_ROWS"
echo "actual rows: $ACTUAL_ROWS"

test "$EXPECTED_ROWS" = "$ACTUAL_ROWS" || {
  echo "row count mismatch: expected $EXPECTED_ROWS, got $ACTUAL_ROWS"
  exit 1
}
```

Claude Specific:
Run the Ubuntu command exactly. If row count mismatches, stop and fix `RAW_FILE`.

### 13. Update processing state.

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

### 14. Regenerate wins.

Human:
Update wins working-memory file.

Ubuntu:
```bash
echo "$WINS_FILE"
```

Claude Specific:
Regenerate `archive/derived/wins.md` from `archive/raw/**/*.jsonl`.
Do not modify `inbox/`.
Do not run git.

### 15. Regenerate state of Ralph.

Human:
Update current-state working-memory file.

Ubuntu:
```bash
echo "$STATE_OF_RALPH_FILE"
```

Claude Specific:
Regenerate `archive/derived/state-of-ralph.md` from:
- `archive/raw/**/*.jsonl`
- `archive/derived/weekly/*.md` if useful
Do not modify `inbox/`.
Do not run git.

### 16. Regenerate weekly review.

Human:
Update current weekly review file.

Ubuntu:
```bash
echo "$WEEKLY_FILE"
```

Claude Specific:
Regenerate `archive/derived/weekly/YYYY-WXX.md` for the current ISO week from the past 7 days of processed logs.
Do not modify `inbox/`.
Do not run git.

### 17. Regenerate manifest.

Human:
Update working-memory manifest.

Ubuntu:
```bash
echo "$MANIFEST_FILE"
```

Claude Specific:
Create or update `archive/derived/manifest.md`.

It must say:

```text
This folder is Ralph's published working memory.

Use these files:
- derived/wins.md for wins and progress.
- derived/state-of-ralph.md for current operating state.
- derived/weekly/*.md for weekly summaries.
- raw/YYYY/YYYY-MM.jsonl for detailed structured logs.

Do not treat the Google Drive folder as the source of truth.
The local PC folder is the source of truth.
The Google Drive folder is the published working-memory copy.
```

### 18. Verify derived files exist.

Human:
All working-memory files must exist before upload.

Ubuntu:
```bash
test -f "$WINS_FILE" || { echo "missing: $WINS_FILE"; exit 1; }
test -f "$STATE_OF_RALPH_FILE" || { echo "missing: $STATE_OF_RALPH_FILE"; exit 1; }
test -f "$WEEKLY_FILE" || { echo "missing: $WEEKLY_FILE"; exit 1; }
test -f "$MANIFEST_FILE" || { echo "missing: $MANIFEST_FILE"; exit 1; }

echo "derived files exist"
```

Claude Specific:
Run the Ubuntu command exactly.

### 19. Count publish bytes.

Human:
Get total bytes for final report.

Ubuntu:
```bash
RAW_BYTES="$(wc -c < "$RAW_FILE" | tr -d ' ')"
WINS_BYTES="$(wc -c < "$WINS_FILE" | tr -d ' ')"
STATE_BYTES="$(wc -c < "$STATE_OF_RALPH_FILE" | tr -d ' ')"
WEEKLY_BYTES="$(wc -c < "$WEEKLY_FILE" | tr -d ' ')"
MANIFEST_BYTES="$(wc -c < "$MANIFEST_FILE" | tr -d ' ')"
TOTAL_BYTES="$((RAW_BYTES + WINS_BYTES + STATE_BYTES + WEEKLY_BYTES + MANIFEST_BYTES))"

echo "$TOTAL_BYTES"
```

Claude Specific:
Run the Ubuntu command exactly.

### 20. Confirm rclone remote works.

Human:
Verify Google Drive remote named `gdrive`.

Ubuntu:
```bash
rclone lsd gdrive: >/dev/null
echo "rclone gdrive remote works"
```

Claude Specific:
Run the Ubuntu command exactly. If it fails, stop and report that `gdrive` remote is not configured.

### 21. Set publish folder and Drive paths.

Human:
Set Drive publish folder paths.

Ubuntu:
```bash
PUBLISH_FOLDER="2026-lifevault-published"
DRIVE_RAW_DIR="$PUBLISH_FOLDER/raw/$YEAR"
DRIVE_DERIVED_DIR="$PUBLISH_FOLDER/derived"
DRIVE_WEEKLY_DIR="$PUBLISH_FOLDER/derived/weekly"
PUBLISH_RAW_FILE="$FILE_MONTH.jsonl"
PUBLISH_WEEKLY_FILE="$ISO_WEEK.md"

echo "$PUBLISH_FOLDER"
echo "$DRIVE_RAW_DIR"
echo "$DRIVE_DERIVED_DIR"
echo "$DRIVE_WEEKLY_DIR"
```

Claude Specific:
Run the Ubuntu command exactly.

### 22. Ensure Drive publish folders exist.

Human:
Create Drive folders if missing.

Ubuntu:
```bash
rclone mkdir "gdrive:$PUBLISH_FOLDER"
rclone mkdir "gdrive:$DRIVE_RAW_DIR"
rclone mkdir "gdrive:$DRIVE_DERIVED_DIR"
rclone mkdir "gdrive:$DRIVE_WEEKLY_DIR"

echo "drive folders ensured"
```

Claude Specific:
Run the Ubuntu command exactly.

### 23. Upload processed raw file.

Human:
Upload `archive/raw/YYYY/YYYY-MM.jsonl`.

Ubuntu:
```bash
rclone copyto "$RAW_FILE" "gdrive:$DRIVE_RAW_DIR/$PUBLISH_RAW_FILE"
```

Claude Specific:
Run the Ubuntu command exactly.

### 24. Upload derived working-memory files.

Human:
Upload derived memory files.

Ubuntu:
```bash
rclone copyto "$WINS_FILE" "gdrive:$DRIVE_DERIVED_DIR/wins.md"
rclone copyto "$STATE_OF_RALPH_FILE" "gdrive:$DRIVE_DERIVED_DIR/state-of-ralph.md"
rclone copyto "$WEEKLY_FILE" "gdrive:$DRIVE_WEEKLY_DIR/$PUBLISH_WEEKLY_FILE"
rclone copyto "$MANIFEST_FILE" "gdrive:$PUBLISH_FOLDER/manifest.md"
```

Claude Specific:
Run the Ubuntu command exactly.

### 25. Verify uploaded files.

Human:
Confirm expected files exist in Drive.

Ubuntu:
```bash
rclone lsjson "gdrive:$DRIVE_RAW_DIR" --files-only \
  | jq -e --arg name "$PUBLISH_RAW_FILE" '.[] | select(.Name == $name)' >/dev/null

rclone lsjson "gdrive:$DRIVE_DERIVED_DIR" --files-only \
  | jq -e '.[] | select(.Name == "wins.md")' >/dev/null

rclone lsjson "gdrive:$DRIVE_DERIVED_DIR" --files-only \
  | jq -e '.[] | select(.Name == "state-of-ralph.md")' >/dev/null

rclone lsjson "gdrive:$DRIVE_WEEKLY_DIR" --files-only \
  | jq -e --arg name "$PUBLISH_WEEKLY_FILE" '.[] | select(.Name == $name)' >/dev/null

rclone lsjson "gdrive:$PUBLISH_FOLDER" --files-only \
  | jq -e '.[] | select(.Name == "manifest.md")' >/dev/null

echo "uploaded files verified"
```

Claude Specific:
Run the Ubuntu command exactly. If any verification fails, stop.

### 26. Final reply.

Human:
Reply only with the publish result.

Ubuntu:
```bash
echo "Folder: $PUBLISH_FOLDER"
echo "Published raw: raw/$YEAR/$PUBLISH_RAW_FILE ($RAW_BYTES bytes)"
echo "Published derived: wins.md, state-of-ralph.md, weekly/$PUBLISH_WEEKLY_FILE, manifest.md"
echo "Total bytes: $TOTAL_BYTES"
```

Claude Specific:
Reply with exactly the four echoed lines.

Expected final format:

```text
Folder: 2026-lifevault-published
Published raw: raw/YYYY/YYYY-MM.jsonl (<bytes> bytes)
Published derived: wins.md, state-of-ralph.md, weekly/YYYY-WXX.md, manifest.md
Total bytes: <bytes>
```

---

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.
