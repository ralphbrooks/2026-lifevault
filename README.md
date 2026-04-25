# LifeVault

A personal log. Append-only. Plain text. Mine.

## What this is

Raw thoughts, wins, lessons, decisions, observations. Logged as they happen, with as little structure as possible at write time. Intelligence (synthesis, pattern-matching, weekly reviews) happens later, on-demand, by AI tools that read this repo.

## Folders

- `inbox/` — where I write. One file per month. Append-only. Sacred.
- `archive/` — reserved for future structured output (JSONL, summaries, derived views). Empty today. Populated later by Cowork or local AI when I explicitly run them.

## Input format

Every line in an inbox file follows this format:
YYYY-MM-DD-HHMM [tag1][tag2]... - free text

- Timestamp is required. Date and 24-hour time, joined with hyphens, no spaces, no colons.
- Tags are optional. Zero, one, or many. Each tag in its own square brackets, no spaces inside the brackets. Tags butt up against each other: `[gen-8][lesson]`.
- Separator is hyphen-space-hyphen (` - `) between the timestamp/tags and the body.
- Body is free text. Newlines inside an entry are fine; new entries start on a new line that begins with a date.

Examples (all valid):
2026-04-25-1942 - got the home automation working after a year
2026-04-25-1955 [divorce] - call with Julie, she pushed back on the timeline
2026-04-26-0830 [gen-8][lesson] - replace spring clamp on first teardown, always
2026-04-26-2210 [win] - listed three units, two sold same day

## Tag rules (for AI tools reading this repo)

- Tags are hints, not categories. The body text is the substrate; tags are a 10% lift.
- Tag drift is expected. `[gen]`, `[gen-8]`, `[generator]`, `[firman]` may all appear over time and may all refer to the same domain. Resolve at read time; do not rewrite raw entries.
- Forgotten tags are fine. Many entries will be untagged. Infer domain and kind from body text when tags are absent.
- Tags are case-insensitive. `[Gen-8]`, `[gen-8]`, and `[GEN_8]` are equivalent.

## What goes in inbox

Anything. Wins, repair notes, mediation observations, ideas, mood, half-formed thoughts, lessons learned, decisions made. There is no wrong entry. The cost of typing a useless line is near zero; the cost of failing to capture a useful one is unbounded.

## What does NOT go in inbox

- Edits to past entries. Inbox is append-only. If something was wrong, write a new line correcting it.
- Long-form documents. Those go in their own files in a future `documents/` folder.
- Anything that should not be version-controlled (passwords, keys, etc.).

## Rules for AI tools that read this repo

1. **Inbox files are sacred.** Read them. Never modify them. Never reorder them. Never "clean them up."
2. **All synthesis is regenerable.** Anything written to `archive/` or any future derived folder must be reproducible from `inbox/` alone. Treat derived output as disposable.
3. **On-demand only.** Do not run scheduled tasks against this repo unless explicitly configured. The default is: do nothing until asked.
4. **Quote raw entries when synthesizing.** When summarizing, link back to the raw line(s) so I can verify.
5. **Tag aliases live in `archive/tag-aliases.json`** (when it exists). Treat that as the authoritative mapping from raw tags to canonical tags. If absent, infer.

## Owner

Ralph Brooks. Frisco, TX.
