# Project: 2026-lifevault

## What this is

A personal log + working-memory system.

The local PC folder is Ralph's source of truth.

GitHub/git is Ralph's external backup/version-control system.

Google Drive is Claude's published working memory.

Claude never touches git.

Claude never reads git logs.

Claude never runs git commands.

## Current Cowork rule

Claude Cowork may have a stale bash filesystem mount.

Therefore:

- Claude file tools are the source of truth for reading and writing local project files.
- Bash is not the source of truth for local project files.
- Bash must not be used to read or validate `inbox/`.
- If Claude's file Read tool and bash disagree, trust Claude's file Read tool for local project content.
- Claude should use its Google Drive connector/tool for publishing to Google Drive.
- Do not use `rclone` inside Cowork.

## Goal

The workflow is:

1. Ralph edits logs on PC in `inbox/YYYY-MM.md`.
2. Ralph may commit/push those edits to GitHub himself.
3. Ralph updates these project instructions in Cowork.
4. Ralph types `"publish"` in Cowork.
5. Claude reads the current inbox using Claude file tools, not bash.
6. Claude processes the current inbox into structured JSONL.
7. Claude regenerates working-memory summaries.
8. Claude writes/updates local archive and root alias files.
9. Claude publishes the processed archive and memory files to Google Drive using Claude's Google Drive connector/tool.
10. Ralph can verify from WSL2 using `rclone` if he wants.
11. Later, Ralph can ask Claude web: "Reference info on Google Drive. Tell me my wins for this week."

## Core principle

AI performs semantic interpretation.

Claude file tools perform local project reading/writing.

Claude Google Drive connector/tool performs Google Drive publishing.

Ralph may use WSL2/rclone separately to verify that Google Drive changed.

Do not make Cowork depend on local bash or `rclone` for publish.

## Required progress logging

When running `publish`, Claude must show concise progress logs so Ralph can verify that Cowork is not going rogue.

Use visible progress messages in this style:

```text
=== publish: read inbox ===
Read inbox/YYYY-MM.md with file tool.
Found <N> timestamped entries.

=== publish: write raw jsonl ===
Wrote archive/raw/YYYY/YYYY-MM.jsonl with <N> rows.
Row-count check: <N> entries -> <N> rows. PASS.

=== publish: write derived memory ===
Wrote archive/derived/wins.md.
Wrote archive/derived/state-of-ralph.md.
Wrote archive/derived/weekly/YYYY-WXX.md.
Wrote archive/derived/manifest.md.
Wrote root aliases: manifest.md, wins.md, state-of-ralph.md, weekly-current.md.

=== publish: google drive ===
Publishing to Google Drive folder: 2026-lifevault-published.
Published raw/YYYY/YYYY-MM.jsonl.
Published derived files.
Published root aliases.

=== publish: complete ===
Folder: 2026-lifevault-published
Published raw: raw/YYYY/YYYY-MM.jsonl
Published derived: wins.md, state-of-ralph.md, weekly/YYYY-WXX.md, manifest.md
Published root aliases: manifest.md, wins.md, state-of-ralph.md, weekly-current.md
```

If any step fails, stop and report:

```text
=== publish: failed ===
Step: <step name>
Failure: <exact failure>
Files modified before failure: <list>
Files not modified: <list if important>
```

Do not hide failures.

Do not continue after a publishing failure unless Ralph explicitly tells you to continue.

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

Claude may also create/update these root-level published-memory aliases for Drive retrieval:

- `manifest.md`
- `wins.md`
- `state-of-ralph.md`
- `weekly-current.md`

Claude never modifies files in `inbox/`.

Claude never modifies git state.

## Git model

Ralph uses git/GitHub as his external copy / backup after adding logs.

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

Claude uses local project files and `archive/state/*.state.json` for processing metadata.

## Google Drive working-memory model

Google Drive is Claude's published working memory.

Publishing to Google Drive is done with Claude's Google Drive connector/tool.

For the `"publish"` command:

- Use Claude file tools for local file reading/writing.
- Use Claude Google Drive connector/tool for Drive publishing.
- Do not use bash/rclone inside Cowork.
- Do not use bash to read `inbox/`.

Claude's Google Drive connector may later be used in chat to read the published files after they have been uploaded.

If Claude cannot later find the files through the Google Drive connector, the issue is Drive connector access/indexing, not local processing.

## Google Drive publish folder

The Drive publish folder is:

```text
2026-lifevault-published
```

It lives at Google Drive root.

Published Drive layout:

```text
2026-lifevault-published/
  README.md
  manifest.md
  wins.md
  state-of-ralph.md
  weekly-current.md
  raw/
    YYYY/
      YYYY-MM.jsonl
  derived/
    manifest.md
    wins.md
    state-of-ralph.md
    weekly/
      YYYY-WXX.md
```

Root-level files are aliases for easier conversational retrieval.

Canonical generated files live under `archive/` locally and under `derived/` / `raw/` in Drive.

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
   - read the current inbox with Claude file tools
   - process the current inbox into `archive/raw/YYYY/YYYY-MM.jsonl`
   - regenerate working-memory files under `archive/derived/`
   - create/update root-level retrieval aliases
   - publish the processed archive and memory files to Google Drive using Claude's Google Drive connector/tool

5. Claude writes only inside `archive/` plus root-level memory alias files.

6. For `"publish"`, do not use `rclone` inside Cowork.

7. For `"publish"`, do not use bash to read local project files.

8. Do not invent helper functions.

9. Do not ask questions during command execution unless a required file, file tool, or Google Drive connector/tool is unavailable.

10. If Google Drive publishing fails, stop and report the exact failure.

---

# Inbox format

The inbox uses one log entry per physical line.

Ignore markdown heading lines such as:

```text
# Inbox — April 2026
```

Standard entry format:

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
- AI may add normalized fields or signals when useful, but the raw body must remain available.

## Tolerant parsing rule

If a line is clearly intended as a timestamped log entry but has a minor format error, process it rather than dropping it.

Examples of minor format errors:

- `2020-4-21-1200 [win]: body`
- missing leading zero in month or day
- colon instead of ` - ` separator

When tolerant parsing is used:

- Preserve the original line in `raw_line`.
- Normalize the best-known date/time if obvious.
- Add a `parse_note` field explaining the issue.
- Do not silently discard the line.
- If date is ambiguous, keep the row but add `date_uncertain: true`.

## Mechanical completeness rule

Every timestamped or timestamp-intended inbox entry must appear as exactly one JSONL row.

This is a mechanical validation rule, not a semantic rule.

Claude uses AI for meaning, but must verify it did not skip entries.

---

# Processing state

Claude maintains local processing metadata in:

```text
archive/state/YYYY-MM.state.json
```

This file is owned by Claude.

State file shape:

```json
{
  "source_file": "inbox/YYYY-MM.md",
  "source_size_bytes": 0,
  "source_sha256": "",
  "processed_entry_count": 0,
  "last_processed_at": "YYYY-MM-DDTHH:MM:SS",
  "raw_file": "archive/raw/YYYY/YYYY-MM.jsonl"
}
```

Important:

- Because Cowork bash can be stale, do not use bash byte offsets as the authority for the inbox.
- State is metadata, not an authority that overrides the current file-tool read.
- If state says something inconsistent with the current file-tool read, rebuild the raw JSONL from the current inbox file-tool content and update state.
- Do not use git for change detection.

Preferred behavior:

1. Read the full current-month inbox with Claude file tools.
2. Parse every timestamped or clearly timestamp-intended line.
3. Rebuild `archive/raw/YYYY/YYYY-MM.jsonl` from the current inbox.
4. Validate row count.
5. Update `archive/state/YYYY-MM.state.json`.

This intentionally favors correctness over incremental byte-offset optimization.

---

# Commands

- `"process inbox"` — read the current month's `inbox/YYYY-MM.md` using Claude file tools, parse all timestamped log entries, rebuild `archive/raw/YYYY/YYYY-MM.jsonl`, validate completeness, and update `archive/state/YYYY-MM.state.json`. Do not modify `inbox/`. Do not run git.

- `"wins"` — regenerate `archive/derived/wins.md` and root `wins.md` from available archive data.

- `"weekly review"` — generate `archive/derived/weekly/YYYY-WXX.md` and root `weekly-current.md`.

- `"state of ralph"` — update `archive/derived/state-of-ralph.md` and root `state-of-ralph.md`.

- `"manifest"` — update `archive/derived/manifest.md` and root `manifest.md`.

- `"publish"` — process the current inbox with Claude file tools, validate completeness, regenerate working-memory files, create root aliases, and publish the processed archive and memory files to Google Drive using Claude's Google Drive connector/tool.

---

# Command: process inbox

## Purpose

Convert the current month's hand-written inbox log into structured JSONL.

For now, prioritize correctness over incremental processing.

Do not rely on bash reads of `inbox/`.

## Inputs

Current month inbox file:

```text
inbox/YYYY-MM.md
```

Example:

```text
inbox/2026-04.md
```

## Outputs

Processed archive file:

```text
archive/raw/YYYY/YYYY-MM.jsonl
```

Example:

```text
archive/raw/2026/2026-04.jsonl
```

Processing state file:

```text
archive/state/YYYY-MM.state.json
```

Example:

```text
archive/state/2026-04.state.json
```

## JSONL row shape

Each row must be one valid JSON object on one line.

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

Recommended expanded fields:

```json
{
  "ts": "YYYY-MM-DDTHH:MM",
  "date": "YYYY-MM-DD",
  "time": "HH:MM",
  "source_file": "inbox/YYYY-MM.md",
  "entry_type": "log",
  "tags": [],
  "body": "...",
  "raw_line": "...",
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
- Exact timestamp parsing must follow the inbox format when possible.
- One timestamped inbox line must produce exactly one JSONL row.

## Procedure

1. Determine current month as `YYYY-MM`.
2. Read `inbox/YYYY-MM.md` using Claude file tools.
3. Ignore markdown headings and blank lines.
4. For each timestamped entry line, create one JSONL row.
5. Rebuild `archive/raw/YYYY/YYYY-MM.jsonl` from the current inbox.
6. Ensure every timestamped or timestamp-intended inbox line has exactly one JSONL row.
7. Write `archive/state/YYYY-MM.state.json`.
8. Do not modify `inbox/`.
9. Do not run git.

## Required progress log

Show:

```text
=== process inbox: read ===
Read inbox/YYYY-MM.md.
Found <N> timestamped entries.

=== process inbox: write ===
Wrote archive/raw/YYYY/YYYY-MM.jsonl with <N> rows.

=== process inbox: validate ===
Row-count check: <N> entries -> <N> rows. PASS.

=== process inbox: state ===
Wrote archive/state/YYYY-MM.state.json.
```

## Required state update

After successful processing, write state similar to:

```json
{
  "source_file": "inbox/YYYY-MM.md",
  "source_size_bytes": 0,
  "source_sha256": "",
  "processed_entry_count": 0,
  "last_processed_at": "YYYY-MM-DDTHH:MM:SS",
  "raw_file": "archive/raw/YYYY/YYYY-MM.jsonl"
}
```

`source_size_bytes` and `source_sha256` may be computed from the file-tool content rather than bash.

---

# Command: wins

## Purpose

Regenerate the wins summary from available processed archive data.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `inbox/*.md` only if archive data is incomplete or missing

## Outputs

```text
archive/derived/wins.md
wins.md
```

## Procedure

1. Read processed archive data.
2. Identify wins, progress, resilience, completed tasks, recovered problems, and business/legal/technical breakthroughs.
3. Write `archive/derived/wins.md`.
4. Also write root alias `wins.md`.
5. Do not modify `inbox/`.
6. Do not run git.

## Required progress log

Show:

```text
=== wins: write ===
Wrote archive/derived/wins.md.
Wrote root alias wins.md.
```

## Output style

Direct. Specific. No motivational fluff unless Ralph asks.

---

# Command: weekly review

## Purpose

Generate a weekly review from the current week's processed logs.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `inbox/*.md` only if archive data is incomplete or missing

## Outputs

```text
archive/derived/weekly/YYYY-WXX.md
weekly-current.md
```

## Procedure

1. Determine the current ISO week.
2. Read entries for the current week or past 7 days.
3. Summarize:
   - wins
   - problems
   - decisions
   - open loops
   - risks
   - next actions
4. Write `archive/derived/weekly/YYYY-WXX.md`.
5. Write root alias `weekly-current.md`.
6. Do not modify `inbox/`.
7. Do not run git.

## Required progress log

Show:

```text
=== weekly review: write ===
Wrote archive/derived/weekly/YYYY-WXX.md.
Wrote root alias weekly-current.md.
```

---

# Command: state of ralph

## Purpose

Update Ralph's current operating state from processed logs.

## Input priority

1. `archive/raw/**/*.jsonl`
2. `archive/derived/weekly/*.md`
3. `inbox/*.md` only if needed

## Outputs

```text
archive/derived/state-of-ralph.md
state-of-ralph.md
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

Also write root alias `state-of-ralph.md`.

Do not modify `inbox/`.

Do not run git.

## Required progress log

Show:

```text
=== state of ralph: write ===
Wrote archive/derived/state-of-ralph.md.
Wrote root alias state-of-ralph.md.
```

---

# Command: manifest

## Purpose

Update the entry-point files Claude should use later when reading Google Drive working memory.

## Outputs

```text
archive/derived/manifest.md
manifest.md
```

## Required contents

The manifest/root orientation must explain:

```text
This folder contains Ralph's published working memory from real-world activity logs.

Use this folder when Ralph asks broad conversational questions about his recent or historical activity, including but not limited to:
- wins and progress
- generator repair and sales status
- divorce/legal status and strategy context
- home repairs and maintenance
- business experiments and cash runway
- technical work
- health, stress, and operating state
- open loops, blockers, and next actions
- what changed recently
- what Ralph has already tried
- what Ralph should focus on next

Start with:
- state-of-ralph.md for current operating context.
- weekly-current.md for the most recent weekly summary.
- wins.md for explicit wins and progress.
- manifest.md / README.md for folder orientation.
- raw/YYYY/YYYY-MM.jsonl for detailed structured source logs.

When answering Ralph's questions, first orient from the derived files. Then drill into raw JSONL only if needed. Prefer recent entries unless Ralph asks for historical context. Use tags and body text together. Do not assume the derived files are complete if raw logs contain more detail.

The local PC folder is the source of truth.
The Google Drive folder is the published working-memory copy.
```

## Procedure

1. Create or update `archive/derived/manifest.md`.
2. Create or update root `manifest.md`.
3. Do not overwrite `README.md` unless Ralph has explicitly allowed it. If README already exists, leave it alone.
4. Keep manifest concise.
5. Do not modify `inbox/`.
6. Do not run git.

## Required progress log

Show:

```text
=== manifest: write ===
Wrote archive/derived/manifest.md.
Wrote root alias manifest.md.
README.md left unchanged unless explicitly allowed.
```

---

# Command: publish

## Purpose

Use Claude file tools to process the current inbox and generate memory files.

Then use Claude's Google Drive connector/tool to publish the processed archive and memory files to Google Drive.

Do not publish raw `inbox/`.

Do not use rclone inside Cowork.

Do not use git.

## Required behavior

- Read `inbox/YYYY-MM.md` using Claude file tools.
- Process all current timestamped or timestamp-intended entries into JSONL.
- Validate that every entry has one JSONL row.
- Regenerate:
  - `archive/derived/wins.md`
  - `archive/derived/state-of-ralph.md`
  - `archive/derived/weekly/YYYY-WXX.md`
  - `archive/derived/manifest.md`
  - root `manifest.md`
  - root `wins.md`
  - root `state-of-ralph.md`
  - root `weekly-current.md`
- Publish only after processed JSONL exists and validates.
- Always overwrite published files.
- Create the Drive publish folder/subfolders if missing.
- Do not upload `inbox/YYYY-MM.md`.

## Claude Specific

For this command:

- Use Claude file tools for local file reading and writing.
- Do not use bash to read `inbox/`.
- Do not use `rclone`.
- Use Claude's Google Drive connector/tool for Drive publishing.
- Do not use bash `wc`, `cat`, or `tail` on local project files.
- Do not let stale bash output block processing.
- Do not invent helper functions.
- Do not run git.
- Do not modify `inbox/`.
- If Google Drive publishing fails, stop and report the exact failure.

## Procedure

### 1. Read current inbox with file tools.

Human:
Read the current month inbox from the local project.

Claude Specific:
Use Claude file Read tool to read `inbox/YYYY-MM.md`.

Do not use bash for this.

Progress log:

```text
=== publish: read inbox ===
Read inbox/YYYY-MM.md with file tool.
Found <N> timestamped entries.
```

### 2. Parse inbox.

Human:
Convert timestamped log entries to JSONL.

Claude Specific:
For every line matching or clearly intending:

```text
YYYY-MM-DD-HHMM [tag][tag] - body text
```

create exactly one JSONL row.

Preserve:
- timestamp
- date
- time
- tags
- body
- raw line when useful

Add semantic signals if useful.

Progress log:

```text
=== publish: parse inbox ===
Parsed <N> entries.
Tolerant parsed entries: <M>.
```

### 3. Write raw JSONL.

Human:
Write the processed structured log.

Claude Specific:
Write or overwrite:

```text
archive/raw/YYYY/YYYY-MM.jsonl
```

Use one valid JSON object per line.

Progress log:

```text
=== publish: write raw jsonl ===
Wrote archive/raw/YYYY/YYYY-MM.jsonl with <N> rows.
```

### 4. Validate row count.

Human:
Make sure no timestamped entries were skipped.

Claude Specific:
Count timestamped or timestamp-intended entries from the file-tool-read inbox content.

Count JSONL rows written.

If counts differ, fix the JSONL before continuing.

Progress log:

```text
=== publish: validate raw jsonl ===
Row-count check: <N> entries -> <N> rows. PASS.
```

### 5. Write processing state.

Human:
Update state metadata.

Claude Specific:
Write or overwrite:

```text
archive/state/YYYY-MM.state.json
```

Include:
- source file
- source size/hash if computable from file-tool content
- processed entry count
- last processed time
- raw file path

Progress log:

```text
=== publish: write state ===
Wrote archive/state/YYYY-MM.state.json.
```

### 6. Regenerate working-memory files.

Human:
Create summaries and aliases for broad conversational retrieval.

Claude Specific:
Generate/update:

```text
archive/derived/wins.md
archive/derived/state-of-ralph.md
archive/derived/weekly/YYYY-WXX.md
archive/derived/manifest.md
manifest.md
wins.md
state-of-ralph.md
weekly-current.md
```

Do not modify `inbox/`.

Progress log:

```text
=== publish: write derived memory ===
Wrote archive/derived/wins.md.
Wrote archive/derived/state-of-ralph.md.
Wrote archive/derived/weekly/YYYY-WXX.md.
Wrote archive/derived/manifest.md.
Wrote root aliases: manifest.md, wins.md, state-of-ralph.md, weekly-current.md.
```

### 7. Publish to Google Drive with connector/tool.

Human:
Publish generated files to Google Drive.

Claude Specific:
Use Claude's Google Drive connector/tool.

Drive folder:

```text
2026-lifevault-published
```

If the folder does not exist at Drive root, create it.

If the folder exists, use it.

If multiple folders exist with this exact name, use the first/oldest visible folder and do not delete duplicates.

Create subfolders if missing:

```text
raw/YYYY
derived
derived/weekly
```

Overwrite each published file by replacing the existing file with the same name in the target folder.

Publish these files:

Local to Drive:

```text
archive/raw/YYYY/YYYY-MM.jsonl
→ 2026-lifevault-published/raw/YYYY/YYYY-MM.jsonl

archive/derived/wins.md
→ 2026-lifevault-published/derived/wins.md

archive/derived/state-of-ralph.md
→ 2026-lifevault-published/derived/state-of-ralph.md

archive/derived/weekly/YYYY-WXX.md
→ 2026-lifevault-published/derived/weekly/YYYY-WXX.md

archive/derived/manifest.md
→ 2026-lifevault-published/derived/manifest.md

manifest.md
→ 2026-lifevault-published/manifest.md

wins.md
→ 2026-lifevault-published/wins.md

state-of-ralph.md
→ 2026-lifevault-published/state-of-ralph.md

weekly-current.md
→ 2026-lifevault-published/weekly-current.md
```

If local `README.md` exists, also publish:

```text
README.md
→ 2026-lifevault-published/README.md
```

Do not publish `inbox/YYYY-MM.md`.

Progress log:

```text
=== publish: google drive ===
Publishing to Google Drive folder: 2026-lifevault-published.
Ensured folder: 2026-lifevault-published.
Ensured folder: raw/YYYY.
Ensured folder: derived.
Ensured folder: derived/weekly.
Published raw/YYYY/YYYY-MM.jsonl.
Published derived/wins.md.
Published derived/state-of-ralph.md.
Published derived/weekly/YYYY-WXX.md.
Published derived/manifest.md.
Published manifest.md.
Published wins.md.
Published state-of-ralph.md.
Published weekly-current.md.
Published README.md if present.
```

### 8. Final reply.

Claude Specific:
Reply with the final publish result.

Expected format:

```text
=== publish: complete ===
Folder: 2026-lifevault-published
Published raw: raw/YYYY/YYYY-MM.jsonl
Published derived: wins.md, state-of-ralph.md, weekly/YYYY-WXX.md, manifest.md
Published root aliases: manifest.md, wins.md, state-of-ralph.md, weekly-current.md
```

---

# Ralph's WSL2 verification after Cowork publish

Ralph may verify from WSL2 using his own configured `rclone`.

This is for Ralph, not Cowork.

```bash
cd /mnt/c/Users/Ralph/Documents/2026-lifevault

rclone tree gdrive:2026-lifevault-published
rclone cat gdrive:2026-lifevault-published/wins.md
rclone cat gdrive:2026-lifevault-published/raw/2026/2026-04.jsonl
```

To check for a specific new win:

```bash
rclone cat gdrive:2026-lifevault-published/wins.md | grep -i "Red Baron"
```

If the Red Baron win appears in Google Drive, then the local log-to-Drive memory loop is working.

# Claude web retrieval test

After Cowork publish and WSL2 verification, Ralph should test Claude web with:

```text
Reference info on Google Drive. Tell me my wins for this week.
```

Expected behavior:

Claude should discover/use the LifeVault working-memory files and include the Red Baron win if it is in the published weekly/wins data.

---

## Context

- Name: Ralph
- Location: Frisco, TX
- Communication: direct, technical, no flowery output, push back.
- Token economics matter. Default to minimum.
