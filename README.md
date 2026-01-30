# Apple Voice Memos Claude Skill

A Claude skill that lets you list, search, and read transcripts from Apple Voice Memos synced via iCloud.

## Prerequisites

- macOS with Voice Memos iCloud sync enabled
- Python 3
- Claude / Claude Code with the right subscription

## Installation

Copy the `apple-voice-memos` directory into your Claude skills directory. You can install it at either scope:

**Personal** (available in all projects):

```bash
cp -r apple-voice-memos ~/.claude/skills/apple-voice-memos
```

**Project** (available only in a specific project):

```bash
cp -r apple-voice-memos /path/to/your/project/.claude/skills/apple-voice-memos
```

## Usage

Once installed, the skill is available as a slash command in Claude Code:

```
/apple-voice-memos
```

### Arguments

| Argument | Description | Example |
|---|---|---|
| `days:<number>` | Number of days to look back (default: 30) | `days:90` |
| `search:<text>` | Filter memos by title (case-insensitive) | `search:meeting` |

Arguments can be combined:

```
/apple-voice-memos days:60 search:idea
```

### Examples

List all memos from the last 30 days:
```
/apple-voice-memos
```

Search for memos with "standup" in the title from the last 7 days:
```
/apple-voice-memos days:7 search:standup
```

After listing memos, you can ask Claude to fetch the transcript of any specific memo, summarize it, or search across transcripts.

## How It Works

The skill reads from the local iCloud-synced Voice Memos storage at:

```
~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
```

It uses two tools:

- **extract-apple-voice-memos-metadata** - Queries the `CloudRecordings.db` SQLite database (read-only) to list recording titles, dates, and filenames.
- **extract-apple-voice-memos-transcript** - Extracts transcript text from the `tsrp` atom embedded in `.m4a` files by Apple's on-device transcription.

No audio is processed or sent anywhere. The database is opened in read-only mode. Transcripts are extracted locally from the file metadata - not all recordings will have transcripts, as Apple generates them on-device.

## Further Reading

- [Unlocking Apple Voice Memo Transcripts](https://thomascountz.com/2025/06/08/unlocking-apple-voice-memo-transcripts) by Thomas Countz
- [CloudRecordings.db Details](CloudRecordings-db-details.md) - Schema and query reference for the Voice Memos database

## Acknowledgements

The transcript extraction tool (`extract-apple-voice-memos-transcript`) is by [Tomoki Aonuma](https://github.com/uasi), licensed under the [BSD Zero Clause License](https://opensource.org/license/0bsd). Source: [uasi/extract-apple-voice-memos-transcript](https://github.com/uasi/extract-apple-voice-memos-transcript).
