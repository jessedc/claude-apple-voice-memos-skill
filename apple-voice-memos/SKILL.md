---
name: apple-voice-memos
description: Extract and process transcripts from Apple Voice Memos synced via iCloud. Use when the user wants to access, read, or summarize voice memos.
allowed-tools: Bash(python3 *)
compatibility: macOS with Voice Memos iCloud sync enabled, Python 3
license: 0BSD
---

# Apple Voice Memos

Extract and process transcripts from Apple Voice Memos synced via iCloud.

## Prerequisites

Voice Memos must be synced with iCloud on macOS.

## Tools

This skill includes two scripts in its `scripts/` directory:

- **`extract-apple-voice-memos-metadata`** — Queries the CloudRecordings.db SQLite database (read-only) and outputs CSV with columns: `title`, `date`, `duration`, `path`. Supports optional flags: `--limit N` (default 10), `--offset N`, `--search TERM`, `--after YYYY-MM-DD`, `--before YYYY-MM-DD`.
- **`extract-apple-voice-memos-transcript`** — Extracts the embedded transcript from a `.m4a` file's `tsrp` atom. Outputs timestamped text with filler words removed, intelligent line breaks, and paragraph breaks at natural pauses.

## Step 1: Select a voice memo

Run the metadata script to find the right recording. Choose flags based on what the user asked for:

- **No specific request** → run with no flags (returns 10 most recent)
- **User mentions a topic or keyword** → use `--search TERM`
- **User mentions a time period** → use `--after YYYY-MM-DD` and/or `--before YYYY-MM-DD`
- **User wants to see more results** → use `--offset N` to paginate, or `--limit N` to increase the batch size

Flags can be combined, e.g. `--search work --after 2026-01-01 --limit 5`.

```bash
python3 scripts/extract-apple-voice-memos-metadata [flags]
```

Present the results as a numbered list showing title, date, and duration. Ask the user which memo they'd like to work with.

**Error handling:**
- "Database not found" → Voice Memos iCloud sync is not enabled on this Mac.

## Step 2: Extract the transcript

Run the transcript script with the `path` value from the selected recording:

```bash
python3 scripts/extract-apple-voice-memos-transcript "<FILENAME>.m4a"
```

Present the timestamped transcript to the user.

**Error handling:**
- "tsrp atom not found" → This recording does not have an embedded transcript. Apple generates transcripts on-device and not all recordings will have one.
- File not found → The recording file may not have synced to this Mac yet.

## Step 3: Process the transcript

Read `PROMPT.md` from this skill's directory. Append the transcript after the `## Transcript` heading. Send the complete prompt and transcript to a subagent with fresh context for processing.

The subagent will produce a structured markdown document with narrative summary, detailed notes, asides, and action items. Present this output to the user.

After presenting the output, ask the user if they'd like to save it as a markdown file. Suggest a filename in the format `YYYY-MM-DD-slugified-title.md` derived from the memo's title and date (e.g., `2026-02-04-the-soul-of-a-new-machine.md`). Save to the current working directory by default. The user may accept, provide a different name or path, request adjustments to the content first, or skip saving.
