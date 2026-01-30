---
name: apple-voice-memos
description: Fetch metadata and transcripts from Apple Voice Memos synced via iCloud. Use when the user wants to list, search, or read voice memos.
allowed-tools: Bash(python3:*), Bash(ls:*)
compatibility: macOS with Voice Memos iCloud sync enable, Python 3
license: 0BSD
metadata:
  version: "0.1.0"
  author: "Jesse Collis <jesse@jessedc.dev>"

---

# Apple Voice Memos

Fetch metadata and transcripts from Apple Voice Memos synced via iCloud.

User arguments: $ARGUMENTS

## Arguments

The user may provide optional arguments:

- **days:<number>** - Number of days to look back (default: 30). Example: `days:90`
- **search:<text>** - Filter memos by title (case-insensitive substring match). Example: `search:meeting`
- Both can be combined: `days:60 search:idea`
- If no arguments are provided, list all memos from the last 30 days.

## Prerequisites

Voice Memos must be synced with iCloud. The recordings directory is:

```
~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
```

Before doing anything else, verify this directory exists using `ls`. If it does not exist, inform the user that Voice Memos iCloud sync does not appear to be enabled and stop.

## Tools

This skill includes two helper tools in its `scripts/` directory.

### `extract-apple-voice-memos-metadata`

Extracts recording metadata (title, date, filename) from the CloudRecordings.db SQLite database.

- Outputs CSV to stdout with columns: `title`, `date`, `path`
- `-d DAYS` controls how far back to look (default: 30 days)
- Dates are converted from Core Data epoch (seconds since 2001-01-01) to ISO 8601
- The database is opened in read-only mode (`?mode=ro`)

### `extract-apple-voice-memos-transcript`

Extracts the transcript embedded in a Voice Memo `.m4a` file. Apple stores transcripts in a proprietary `tsrp` atom inside the m4a container.

- Default output is `--text` (plain text transcript)
- Not all recordings have transcripts; the tool will exit with an error if no `tsrp` atom is found
- Only Python 3 standard library is required (no dependencies)

### Important notes

- `ZPATH` contains only the filename (e.g., `20260127 132931-409ABD3B.m4a`), not a full path. The full path is constructed by joining the Recordings directory with the filename.

## Workflow

### Step 1: Fetch recording metadata

Run the metadata extraction tool to list recent recordings:

```bash
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" -d <DAYS>
```

Where `<DAYS>` is the number of days from the `days:` argument (default 30).

This outputs CSV with columns: `title`, `date`, `path`

### Step 2: Apply search filter (if provided)

If the user specified `search:<text>`, filter the results to only include rows whose title contains the search text (case-insensitive match). Apply this filter when presenting results.

### Step 3: Present the recordings

Display the recordings in a clear table or list format showing:
- Title
- Date (formatted readably)
- Filename

### Step 4: Fetch transcripts (on request or automatically)

If the user asked for a transcript of a specific memo, or if there is only one result, fetch the transcript:

```bash
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-transcript "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/<FILENAME>" --text
```

Where `<FILENAME>` is the `path` value from the metadata CSV.

If the transcript tool exits with an error (e.g., "tsrp atom not found"), inform the user that no transcript is available for that recording. This is normal - not all memos have transcripts (the device must have generated one).

### Step 5: Respond to follow-up requests

After presenting the initial results, the user may ask to:
- Get the transcript of a specific memo (by title or number)
- Search within transcripts
- Summarize a transcript
- Compare multiple memos

Handle these naturally using the tools above.

## Error Handling

- **Recordings directory not found**: Tell the user Voice Memos iCloud sync is not enabled or the recordings have not synced to this Mac.
- **Database not found**: Tell the user the CloudRecordings.db file is missing - Voice Memos may not have synced yet.
- **No recordings found**: Tell the user no recordings were found in the specified date range. Suggest increasing the `days:` parameter.
- **Transcript not available**: Tell the user the recording does not have an embedded transcript. Apple generates transcripts on-device and not all recordings will have one.
