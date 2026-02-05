# Apple Voice Memos Claude Skill

A Claude skill for listing, searching, and reading transcripts from Apple Voice Memos synced via iCloud.

No audio is processed or sent anywhere. The database is opened read-only and transcripts are extracted locally from file metadata.

## Prerequisites

- macOS with Voice Memos iCloud sync enabled
- Python 3
- Claude Code or Claude Desktop

## Example Usage

- "summarize my most recent voice memo"
- "list my voice memos from 2026"
- "Summarize the voice memo from today about "a new machine""

_There's actually a lot of things you can do once the transcript is available to the LLM. You should describe the specifics of your voice memos to Claude to seek guidance on the most appropriate prompt to apply to your transcript._

## Installation

### Claude Code

Copy the `apple-voice-memos` directory into your Claude skills directory. You can install it at either scope:

```bash
cp -r apple-voice-memos ~/.claude/skills/apple-voice-memos
```

Once installed, the skill is available as a slash command in Claude Code. See [SKILL.md](./apple-voice-memos/SKILL.md) for the full list of options. It will also be called automatically.

```
/apple-voice-memos
```

### Claude Desktop

Upload a zip archive or release archive of the apple-voice-memos directory in Settings -> Capabilities -> Skills -> Add

## How It Works

The skill uses two tools with auto-detection of the Voice Memos directory:

- **extract-apple-voice-memos-metadata** - Queries the `CloudRecordings.db` SQLite database (read-only) to list recording titles, dates, duration, and filenames.
- **extract-apple-voice-memos-transcript** - Extracts transcript text from the `tsrp` atom embedded in `.m4a` files by Apple's on-device transcription. Its default behavior appls logic to the extracted transcript to add timestamps, line breaks and remove disfluencies and pause markers.

## Limitations
Skills are the simplest way to extend Claude's functionality, they operate within the scope of the environment Claude/Claude Code is running in. The Claude Code CLI has full filesystem access so both scripts and their defaults can be leveraged to fetch metadata and transcripts from your recordings. Claude Desktop on the other hand runs its commands in a restricted containerized environment - so you need to provide the files to the container manually. 

## Extension Opportunities ([TODO.md](./TODO.md))
- Migrating the skill to a plugin would add more functionality to Claude Desktop
- Define an AGENT, add a new SKILL or extent the existing SKILL with some good summarisation prompts and guidelines


## Further Reading

- [Unlocking Apple Voice Memo Transcripts](https://thomascountz.com/2025/06/08/unlocking-apple-voice-memo-transcripts) by Thomas Countz
- [CloudRecordings.db Details](CloudRecordings-db-details.md) - Schema and query reference for the Voice Memos database

## Acknowledgements

The original transcript extraction tool (`extract-apple-voice-memos-transcript`) is by [Tomoki Aonuma](https://github.com/uasi), licensed under the [BSD Zero Clause License](https://opensource.org/license/0bsd). Source: [uasi/extract-apple-voice-memos-transcript](https://github.com/uasi/extract-apple-voice-memos-transcript).
