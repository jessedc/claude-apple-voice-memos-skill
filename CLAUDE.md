# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude skill that provides access to Apple Voice Memos synced via iCloud on macOS. The skill includes tools to list recordings, search by date/title, and extract embedded transcripts from Voice Memo files.

## Architecture

The project consists of:
- **Skill definition**: `apple-voice-memos/SKILL.md` - Defines the skill's capabilities, arguments, and workflow
- **Python scripts**: Located in `apple-voice-memos/scripts/`
  - `extract-apple-voice-memos-metadata`: Queries the CloudRecordings.db SQLite database for recording metadata (title, date, duration, filename)
  - `extract-apple-voice-memos-transcript`: Extracts embedded transcripts from .m4a files using the proprietary `tsrp` atom format

## Key Technical Details

### Data Sources
- **CloudRecordings.db**: SQLite database at `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db`
  - Core Data backed (note `Z_` prefixed columns)
  - Dates stored as Core Data epoch (seconds since 2001-01-01, add 978307200 for Unix epoch)
  - Despite column names, `ZENCRYPTEDTITLE` and `ZENCRYPTEDNAME` contain plaintext
- **Audio files**: .m4a files in the same Recordings directory
  - Transcripts stored in proprietary `tsrp` atom within the m4a container
  - Not all recordings have transcripts (generated on-device by Apple)

### Environment Detection
The skill behaves differently in two environments:
- **Claude Code (local macOS)**: Full functionality with filesystem access
- **Claude Desktop (Linux container)**: Limited to single-file transcript extraction via uploads

## Recent Updates

### Improved Line-Breaking Algorithm (2026-02-04)
- Cleaned Apple pause markers (ellipsis artifacts like `. ..`) from output
- Sentence-ending punctuation is now the primary break signal (matching industry best practices)
- Added paragraph breaks (blank lines) when time gap between lines exceeds 6 seconds
- Improved false start detection ("One of my One of my" → "One of my")
- Filler words (uh, um) are now always removed for cleaner LLM consumption

### Duration in Metadata (2026-02-01)
- Added `ZDURATION` to the metadata extraction query
- Duration formatted as `M:SS` or `H:MM:SS` for recordings over an hour
- CSV output now includes `duration` column between `date` and `path`

### Timestamps as Default Output (2026-02-04)
- Changed transcript extraction tool to output timestamps by default
- Timestamps shown in `[M:SS]` or `[H:MM:SS]` format depending on recording length
- Added `--text` flag for plain text output without timestamps
- Intelligently groups word segments into readable lines based on time gaps and sentence structure

### Claude Desktop Compatibility (2026-01-30)
- Added environment detection to support both Claude Code and Claude Desktop
- Claude Code: Full functionality with database queries and bulk processing
- Claude Desktop: Single-file transcript extraction via manual .m4a file upload
- Environment-specific error handling and user guidance

## Commands

This is a Claude skill project, not a traditional development project. There are no build, test, or lint commands. The Python scripts are standalone executables that require only Python 3 standard library.

To test the skill locally:
```bash
# List recent recordings (metadata)
python3 apple-voice-memos/scripts/extract-apple-voice-memos-metadata "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db" -d 30

# Extract transcript from a specific file (with timestamps - default)
python3 apple-voice-memos/scripts/extract-apple-voice-memos-transcript "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/<FILENAME>.m4a"

# Extract transcript without timestamps (text only)
python3 apple-voice-memos/scripts/extract-apple-voice-memos-transcript "$HOME/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/<FILENAME>.m4a" --text
```

## Important Notes

- The skill uses read-only database access (`?mode=ro` in SQLite connection)
- All tools are licensed under 0BSD (BSD Zero Clause License)
- The transcript extraction tool is based on work by Tomoki Aonuma (https://github.com/uasi/extract-apple-voice-memos-transcript)