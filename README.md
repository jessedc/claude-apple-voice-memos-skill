# Apple Voice Memos Claude Skill

Fetch metadata and transcripts from local Apple Voice Memos synced via iCloud. List, search, and read transcripts

No audio is processed or sent anywhere. The database is opened in read-only mode. Transcripts are extracted locally from the file metadata.

## Prerequisites

- macOS with Voice Memos iCloud sync enabled
- Python 3
- Claude Code with the right subscription to use skills

## Example Usage

- "summarize my most recent voice memo"
- "list my voice memos from 2026"
- "Summarize the voice memo from today about "a new machine""

## Installation

### Claude Code

Copy the `apple-voice-memos` directory into your Claude skills directory. You can install it at either scope:

**Personal** (available in all projects):

```bash
cp -r apple-voice-memos ~/.claude/skills/apple-voice-memos
```

**Project** (available only in a specific project):

```bash
cp -r apple-voice-memos /path/to/your/project/.claude/skills/apple-voice-memos
```

### Claude Desktop

Upload a zip archive or release archive of the apple-voice-memos directory in Settings -> Capabilities -> Skills -> Add

## Slash Command

Once installed, the skill is available as a slash command in Claude Code. The [SKILL.md](./apple-voice-memos/SKILL.md) outlines that different opsions and how they're used internally.

```
/apple-voice-memos
```

## How It Works

The skill uses two tools with auto-detection of the Voice Memos directory:

- **extract-apple-voice-memos-metadata** - Queries the `CloudRecordings.db` SQLite database (read-only) to list recording titles, dates, durations, and filenames.
- **extract-apple-voice-memos-transcript** - Extracts transcript text from the `tsrp` atom embedded in `.m4a` files by Apple's on-device transcription and optionally processes it to produce line breaks, remove disfluencies, pause markers and provide timestamps.


## Further Reading

- [Unlocking Apple Voice Memo Transcripts](https://thomascountz.com/2025/06/08/unlocking-apple-voice-memo-transcripts) by Thomas Countz
- [CloudRecordings.db Details](CloudRecordings-db-details.md) - Schema and query reference for the Voice Memos database

## Acknowledgements

The original transcript extraction tool (`extract-apple-voice-memos-transcript`) is by [Tomoki Aonuma](https://github.com/uasi), licensed under the [BSD Zero Clause License](https://opensource.org/license/0bsd). Source: [uasi/extract-apple-voice-memos-transcript](https://github.com/uasi/extract-apple-voice-memos-transcript).
