---
name: apple-voice-memos
description: Extract and process transcripts from Apple Voice Memos synced via iCloud. Use when the user wants to access, read, or summarize voice memos.
allowed-tools: Bash(python3:*)
compatibility: macOS with Voice Memos iCloud sync enabled, Python 3
license: 0BSD
---

# Apple Voice Memos

Extract and process transcripts from Apple Voice Memos synced via iCloud.

## Prerequisites

Voice Memos must be synced with iCloud on macOS.

## Tools

This skill includes two scripts in its `scripts/` directory:

- **`extract-apple-voice-memos-metadata`** — Queries the CloudRecordings.db SQLite database (read-only) and outputs CSV with columns: `title`, `date`, `duration`, `path`. Returns the 30 most recent recordings.
- **`extract-apple-voice-memos-transcript`** — Extracts the embedded transcript from a `.m4a` file's `tsrp` atom. Outputs timestamped text with filler words removed, intelligent line breaks, and paragraph breaks at natural pauses.

## Step 1: Select a voice memo

Run the metadata script to list recent recordings:

```bash
python3 scripts/extract-apple-voice-memos-metadata
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

The subagent will produce a structured markdown document with narrative summary, detailed notes, asides, and action items. Present this output to the user — they can save it or ask for adjustments.
