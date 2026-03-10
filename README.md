# Apple Voice Memos Claude Skill

Extract and process transcripts from Apple Voice Memos synced via iCloud.

## Installing this skill

You can install this skill into Claude Code, Gemini, Cursor, and other agents that support the [Agent Skills](https://agentskills.io/home) format:

```bash
npx skills add https://github.com/jessedc/claude-apple-voice-memos-skill --skill apple-voice-memos
```

If you get the error `npx: command not found`, you need to install Node first:

```bash
brew install node
```

When using `npx`, you can select whether the skill should be installed just for one project or made available for all your projects.

Alternatively, you can clone this repository and install it however you want.

### Prerequisites

- macOS with Voice Memos iCloud sync enabled

## Using this skill

The skill is called Apple Voice Memos, and can be triggered in Claude Code:

> /apple-voice-memos

The skill walks you through three steps:

1. **Select** — Lists your 30 most recent voice memos by title, date, and duration
2. **Extract** — Pulls the embedded transcript from the selected memo
3. **Process** — Sends the transcript to a subagent that produces structured notes with narrative summary, detailed notes, asides, and action items

You can also trigger the skill using natural language:

> Use the Apple Voice Memos skill to summarize my most recent voice memo.

## How It Works

Two Python scripts (standard library only, no dependencies):

- **`extract-apple-voice-memos-metadata`** — Queries the `CloudRecordings.db` SQLite database (read-only) for recording titles, dates, durations, and filenames.
- **`extract-apple-voice-memos-transcript`** — Extracts transcript text from the `tsrp` atom embedded in `.m4a` files by Apple's on-device transcription. Adds timestamps, removes filler words, and inserts paragraph breaks at natural pauses.

Transcripts are extracted from the `.m4a` files present on your machine, no audio is processed.

## Further Reading

- [Unlocking Apple Voice Memo Transcripts](https://thomascountz.com/2025/06/08/unlocking-apple-voice-memo-transcripts) by Thomas Countz

## Acknowledgements

The original transcript extraction tool is by [Tomoki Aonuma](https://github.com/uasi), licensed under the [BSD Zero Clause License](https://opensource.org/license/0bsd). Source: [uasi/extract-apple-voice-memos-transcript](https://github.com/uasi/extract-apple-voice-memos-transcript).

## License

[0BSD](https://opensource.org/license/0bsd)
