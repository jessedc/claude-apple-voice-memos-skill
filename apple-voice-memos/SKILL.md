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
- **since:<date>** - Include recordings since this date. Example: `since:2026-01-01`
- **until:<date>** - Include recordings until this date (use with since). Example: `since:2026-01-01 until:2026-01-31`
- **year:<year>** - Include recordings from specified year. Example: `year:2026`
- **month:<yyyy-mm>** - Include recordings from specified month. Example: `month:2026-01`
- **search:<text>** - Filter memos by title (case-insensitive substring match). Example: `search:meeting`
- Date filters are mutually exclusive (only use one of: days, since, year, month)
- Search can be combined with any date filter: `year:2026 search:career`
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

Queries the CloudRecordings.db SQLite database to retrieve recording metadata.

**Output Format:**
- CSV to stdout with columns: `title`, `date`, `duration`, `path`
- Duration formatted as `M:SS` or `H:MM:SS` for recordings over an hour
- Dates converted from Core Data epoch to ISO 8601 format
- Database opened in read-only mode (`?mode=ro`) for safety

**Date Filtering Options:**
- `-d, --days N`: Look back N days from today (default: 30)
- `--since YYYY-MM-DD`: Include recordings since this date
- `--until YYYY-MM-DD`: Include recordings until this date (use with --since)
- `--year YYYY`: All recordings from specified year
- `--month YYYY-MM`: All recordings from specified month

**Usage Examples:**
```bash
# Last 7 days
python3 scripts/extract-apple-voice-memos-metadata "path/to/CloudRecordings.db" -d 7

# Specific month
python3 scripts/extract-apple-voice-memos-metadata "path/to/CloudRecordings.db" --month 2026-01

# Date range
python3 scripts/extract-apple-voice-memos-metadata "path/to/CloudRecordings.db" --since 2026-01-01 --until 2026-01-31
```

### `extract-apple-voice-memos-transcript`

Extracts embedded transcripts from Voice Memo `.m4a` files using Apple's proprietary `tsrp` atom format.

**Output Modes:**
- **Default**: Timestamped transcript with `[M:SS]` or `[H:MM:SS]` format
- `--text`: Plain text without timestamps
- `--json`: Raw JSON data structure
- `--raw`: Binary tsrp atom data

**Features:**
- Automatic removal of filler words (uh, um) for cleaner LLM consumption
- Intelligent line breaking based on sentence boundaries and pauses
- Paragraph breaks (blank lines) at natural topic shifts (6+ second gaps)
- False start detection and cleanup ("I went to I went to" → "I went to")
- Removal of Apple's pause markers (ellipsis artifacts)

**Usage Examples:**
```bash
# Default (with timestamps)
python3 scripts/extract-apple-voice-memos-transcript "recording.m4a"

# Plain text only
python3 scripts/extract-apple-voice-memos-transcript "recording.m4a" --text

# Debug JSON structure
python3 scripts/extract-apple-voice-memos-transcript "recording.m4a" --json
```

**Error Handling:**
- Exit with error if no `tsrp` atom found (not all recordings have transcripts)
- Transcripts are generated on-device by Apple and may not be available for:
  - Recordings made before transcript feature was enabled
  - Recordings in unsupported languages
  - Very short recordings

### Important Notes

- `ZPATH` from the database contains only the filename (e.g., `20260127 132931-409ABD3B.m4a`)
- Full path must be constructed by joining the Recordings directory with the filename
- Both tools require only Python 3 standard library (no external dependencies)
- Tools are designed to be read-only and safe to run repeatedly

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
   python3 /mnt/skills/apple-voice-memos/scripts/extract-apple-voice-memos-transcript "/mnt/user-data/uploads/<FILENAME>"
   ```

4. **Present the results**:
   - Display the transcript if found
   - If no transcript exists (tsrp atom not found), inform the user that this recording doesn't have an embedded transcript

**Note**: In Claude Desktop, database queries and bulk processing are not available. Focus on single-file transcript extraction.

## Claude Code Workflow (Local macOS)

If you're running in Claude Code with local filesystem access, use the full workflow:

### Step 1: Fetch recording metadata

Run the metadata extraction tool with the appropriate date filtering option based on user arguments:

```bash
# Default (last 30 days)
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db"

# With days argument
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" -d <DAYS>

# With year argument
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" --year <YEAR>

# With month argument
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" --month <YYYY-MM>

# With date range
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" --since <DATE> [--until <DATE>]
```

The tool supports the following date filtering options:
- `-d, --days N`: Look back N days from today
- `--since DATE`: Include recordings since this date (YYYY-MM-DD format)
- `--until DATE`: Include recordings until this date (use with --since)
- `--year YEAR`: Include all recordings from the specified year
- `--month YYYY-MM`: Include all recordings from the specified month

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
python3 ~/.claude/skills/apple-voice-memos/scripts/extract-apple-voice-memos-transcript "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/<FILENAME>"
```

**Note**: The tool outputs timestamps by default, providing temporal context and automatically removing filler words (uh, um) for cleaner LLM consumption. Output includes paragraph breaks (blank lines) at natural topic shifts. Use `--text` if you need plain text without timestamps.

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
