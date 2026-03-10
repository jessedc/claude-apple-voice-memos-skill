New Skill Voice Memo

Develop a plan based on instructions below to rework and simplifying the current project (the claude skill). 

Overview:
The skill’s purpose is to give Claude the capability to extract and summarise in a lossless manner the transcripts from Apple Voice Memos present on the user’s device. The skill has a current implementation that we want to rewrite and simplify.

Skill Definition Overview (three clear steps):

Step 1: select the voice memo to use from the available transcripts
- Run the script that queries the sqlite database and returns last 30 voice memos. It returns the name, date, length and filename.
- Error case: database file does not exist
- Future TODO: The script needs a way to select more than just this thirty. 
Step 2: extract the transcript from the selected memo using provided too.
- Input is the filename. There are no options, the default output it processed with timestamps
- Error case: transcript not present in provided file, or provided file doesn’t exist
- Future TODO: Allow raw output and processed output. 
Step 3: Process transcript with provided prompt.
- Append the transcript to the prompt, running as a subagent with a fresh context. The output should be a markdown file the user can save somewhere else. 
- Error case: no error cases here.

Implementation Notes:
- Mirror the simplicity, structure and brevity of this claude skill: https://github.com/twostraws/SwiftUI-Agent-Skill/blob/main/swiftui-pro/SKILL.md 
- Skill should be able to be installed with npx, the same method outlined here: https://github.com/twostraws/SwiftUI-Agent-Skill/blob/main/README.md
- Remove unnecessary scripts. There only needs to be two, one for step 1, one for step 2. 
- Remove unnecessary or unused format options or search options from scripts
- Maintain the transcript processing present in the processing script
- Save mentioned TODOs in their own file
- Use PROMPT.md as the prompt and place it in the right location, referencing it from the main SKILL
