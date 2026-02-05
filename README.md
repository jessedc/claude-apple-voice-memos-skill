# Apple Voice Memos Claude Skill

A Claude skill that lets you list, search, and read transcripts from Apple Voice Memos synced via iCloud locally on your machine. 

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

Copy the `apple-voice-memos` directory into your Claude skills directory. You can install it at either scope:

**Personal** (available in all projects):

```bash
cp -r apple-voice-memos ~/.claude/skills/apple-voice-memos
```

**Project** (available only in a specific project):

```bash
cp -r apple-voice-memos /path/to/your/project/.claude/skills/apple-voice-memos
```

## Slash Command

Once installed, the skill is available as a slash command in Claude Code with some hints. The [SKILL.md](./apple-voice-memos/SKILL.md) outlines how the options are used internally.

```
/apple-voice-memos   [days | date | search-text] [text-only | latest | all]
```

## How It Works

The skill reads from the local iCloud-synced Voice Memos storage at:

```
~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
```

It uses three tools:

- **detect-voice-memos-directly** -  Detects if iCloud sync is enabled for Voice Memos and returns the full path to the Recordings directory.
- **extract-apple-voice-memos-metadata** - Queries the `CloudRecordings.db` SQLite database (read-only) to list recording titles, dates, durations, and filenames.
- **extract-apple-voice-memos-transcript** - Extracts transcript text from the `tsrp` atom embedded in `.m4a` files by Apple's on-device transcription. Contains multiple output options the LLM can use, including the option to include timestamps or just plaintext.

## Further Reading

- [Unlocking Apple Voice Memo Transcripts](https://thomascountz.com/2025/06/08/unlocking-apple-voice-memo-transcripts) by Thomas Countz
- [CloudRecordings.db Details](CloudRecordings-db-details.md) - Schema and query reference for the Voice Memos database

## Acknowledgements

The original transcript extraction tool (`extract-apple-voice-memos-transcript`) is by [Tomoki Aonuma](https://github.com/uasi), licensed under the [BSD Zero Clause License](https://opensource.org/license/0bsd). Source: [uasi/extract-apple-voice-memos-transcript](https://github.com/uasi/extract-apple-voice-memos-transcript).
