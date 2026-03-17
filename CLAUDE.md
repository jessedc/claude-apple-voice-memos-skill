# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude skill that provides access to Apple Voice Memos synced via iCloud on macOS. The skill walks the user through selecting a memo, extracting its transcript, and processing it into structured notes.

## Architecture

The project consists of:
- **`apple-voice-memos/SKILL.md`** — Skill definition with 3-step workflow (select, extract, process)
- **`apple-voice-memos/PROMPT.md`** — Processing prompt sent to a subagent with the transcript for structured note generation
- **`apple-voice-memos/scripts/extract-apple-voice-memos-metadata`** — Queries the CloudRecordings.db SQLite database with optional filtering (title search, date range, pagination, count-only mode)
- **`apple-voice-memos/scripts/extract-apple-voice-memos-transcript`** — Extracts embedded transcripts from .m4a files using the proprietary `tsrp` atom format

## Key Technical Details

### Data Sources
- **CloudRecordings.db**: SQLite database at `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/CloudRecordings.db`
  - Core Data backed (note `Z_` prefixed columns)
  - Dates stored as Core Data epoch (seconds since 2001-01-01, add 978307200 for Unix epoch)
  - Despite column names, `ZENCRYPTEDTITLE` and `ZENCRYPTEDNAME` contain plaintext
- **Audio files**: .m4a files in the same Recordings directory
  - Transcripts stored in proprietary `tsrp` atom within the m4a container
  - Not all recordings have transcripts (generated on-device by Apple)

## Commands

This is a Claude skill project, not a traditional development project. There are no build, test, or lint commands. The Python scripts are standalone executables that require only Python 3 standard library.

To test the scripts locally:
```bash
# List recent recordings (metadata)
python3 apple-voice-memos/scripts/extract-apple-voice-memos-metadata

# Extract transcript from a specific file
python3 apple-voice-memos/scripts/extract-apple-voice-memos-transcript "<FILENAME>.m4a"
```

## Important Notes

- The skill uses read-only database access (`?mode=ro` in SQLite connection)
- Metadata script supports flags: `--limit N`, `--offset N`, `--search TERM`, `--after YYYY-MM-DD`, `--before YYYY-MM-DD`
- Transcript script takes no flags — always outputs timestamped text
