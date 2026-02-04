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

## Environment Detection

First, determine which environment you're running in:

1. **Check for local filesystem access**: Try `ls "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/"`
   - If successful → You're in **Claude Code** (local macOS) → Continue with full workflow below
   - If failed → You're in **Claude Desktop** (Linux container) → Skip to Claude Desktop Workflow section

Before doing anything else in Claude Code, verify this directory exists using `ls` with `$HOME`. If it does not exist, inform the user that Voice Memos iCloud sync does not appear to be enabled and stop.

## Tools

This skill includes two helper tools in its `scripts/` directory.

### `extract-apple-voice-memos-metadata`

Extracts recording metadata (title, date, duration, filename) from the CloudRecordings.db SQLite database.

- Outputs CSV to stdout with columns: `title`, `date`, `duration`, `path`
- Duration is formatted as `M:SS` or `H:MM:SS` for longer recordings
- `-d DAYS` controls how far back to look (default: 30 days)
- Dates are converted from Core Data epoch (seconds since 2001-01-01) to ISO 8601
- The database is opened in read-only mode (`?mode=ro`)

### `extract-apple-voice-memos-transcript`

Extracts the transcript embedded in a Voice Memo `.m4a` file. Apple stores transcripts in a proprietary `tsrp` atom inside the m4a container.

- **IMPORTANT**: Always use `--timestamps --clean` for the best LLM-readable output
- The `--timestamps` option shows when each segment was spoken in `[M:SS]` or `[H:MM:SS]` format
- The `--clean` option removes filler words (uh, um) for cleaner output
- Output includes paragraph breaks (blank lines) at topic shifts (6+ second pauses)
- Not all recordings have transcripts; the tool will exit with an error if no `tsrp` atom is found
- Only Python 3 standard library is required (no dependencies)

### Important notes

- `ZPATH` contains only the filename (e.g., `20260127 132931-409ABD3B.m4a`), not a full path. The full path is constructed by joining the Recordings directory with the filename.

## Claude Desktop Workflow (Container Environment)

If you're running in Claude Desktop (detected by lack of local filesystem access):

### Understanding the Limitations

Claude Desktop runs in a Linux container without access to your Mac's filesystem. However, you can still extract transcripts from individual Voice Memo files.

### How to Extract Transcripts in Claude Desktop

1. **Locate your Voice Memo files on your Mac**:
   ```
   ~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
   ```
   Your .m4a files are stored here with names like `20260127 132931-409ABD3B.m4a`

2. **Upload a Voice Memo file**:
   - Tell the user: "Please upload a Voice Memo .m4a file and I'll extract its transcript"
   - Files will appear in `/mnt/user-data/uploads/`

3. **Extract the transcript**:
   ```bash
   python3 /mnt/skills/apple-voice-memos/scripts/extract-apple-voice-memos-transcript "/mnt/user-data/uploads/<FILENAME>" --timestamps --clean
   ```

4. **Present the results**:
   - Display the transcript if found
   - If no transcript exists (tsrp atom not found), inform the user that this recording doesn't have an embedded transcript

**Note**: In Claude Desktop, database queries and bulk processing are not available. Focus on single-file transcript extraction.

## Claude Code Workflow (Local macOS)

If you're running in Claude Code with local filesystem access, use the full workflow:

### Step 1: Fetch recording metadata

Run the metadata extraction tool to list recent recordings:

```bash
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" -d <DAYS>
```

Where `<DAYS>` is the number of days from the `days:` argument (default 30).

This outputs CSV with columns: `title`, `date`, `duration`, `path`

### Step 2: Apply search filter (if provided)

If the user specified `search:<text>`, filter the results to only include rows whose title contains the search text (case-insensitive match). Apply this filter when presenting results.

### Step 3: Present the recordings

Display the recordings in a clear table or list format showing:
- Title
- Date (formatted readably)
- Duration
- Filename

### Step 4: Fetch transcripts (on request or automatically)

If the user asked for a transcript of a specific memo, or if there is only one result, fetch the transcript:

```bash
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-transcript "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/<FILENAME>" --timestamps --clean
```

**Note**: Always use `--timestamps --clean` for the best output. The `--timestamps` flag provides temporal context, and `--clean` removes filler words (uh, um) for cleaner LLM consumption. Output includes paragraph breaks (blank lines) at natural topic shifts.

Example output with timestamps:
```
[0:00] Okay so I've been thinking about the garage project.
[0:15] Mainly the electrical panel situation, whether we need to upgrade to 200 amp.

[0:32] Actually wait, first thing I need to call the permit office.

[1:05] Back to the panel, the quote from Mike seemed high.
[1:12] I should get at least two more quotes before deciding.
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

### Claude Desktop (Container Environment)
- **Recordings directory not found**: Explain that Claude Desktop runs in a container without local filesystem access. Guide the user to upload individual .m4a files from `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/` on their Mac.
- **No file uploaded**: Ask the user to upload a Voice Memo .m4a file for transcript extraction.
- **Transcript not available**: Tell the user the recording does not have an embedded transcript. Apple generates transcripts on-device and not all recordings will have one.

### Claude Code (Local macOS)
- **Recordings directory not found**: Tell the user Voice Memos iCloud sync is not enabled or the recordings have not synced to this Mac.
- **Database not found**: Tell the user the CloudRecordings.db file is missing - Voice Memos may not have synced yet.
- **No recordings found**: Tell the user no recordings were found in the specified date range. Suggest increasing the `days:` parameter.
- **Transcript not available**: Tell the user the recording does not have an embedded transcript. Apple generates transcripts on-device and not all recordings will have one.
