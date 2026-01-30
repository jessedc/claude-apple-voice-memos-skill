# Apple Voice Memos Skill

This skill allows Claude to fetch metadata and transcripts from Apple Voice Memos that are synced via iCloud.

## Overview

Voice Memos recordings synced with iCloud are stored locally at:

```
~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
```

This directory contains:
- `CloudRecordings.db` - a Core Data-backed SQLite database with recording metadata
- `*.m4a` files - the audio recordings themselves, which may contain embedded transcripts

## Tools

### `tools/extract-apple-voice-memos-metadata`

Extracts recording metadata (title, date, filename) from the CloudRecordings.db SQLite database.

**Usage:**
```bash
python3 tools/extract-apple-voice-memos-metadata <path-to-CloudRecordings.db> [-d DAYS]
```

- Outputs CSV to stdout with columns: `title`, `date`, `path`
- `-d DAYS` controls how far back to look (default: 30 days)
- Dates are converted from Core Data epoch (seconds since 2001-01-01) to ISO 8601

### `tools/extract-apple-voice-memos-transcript`

Extracts the transcript embedded in a Voice Memo `.m4a` file. Apple stores transcripts in a proprietary `tsrp` atom inside the m4a container.

**Usage:**
```bash
python3 tools/extract-apple-voice-memos-transcript <path-to-m4a-file> [--text|--json|--raw]
```

- Default output is `--text` (plain text transcript)
- Not all recordings have transcripts; the tool will exit with an error if no `tsrp` atom is found
- Only Python 3 standard library is required (no dependencies)

## Important Notes

- The database is opened in read-only mode (`?mode=ro`) - this skill cannot modify recordings
- `ZENCRYPTEDTITLE` is plaintext despite the column name
- `ZPATH` contains only the filename (e.g., `20260127 132931-409ABD3B.m4a`), not a full path. The full path is constructed by joining the Recordings directory with the filename.
