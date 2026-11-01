# Common Editor

Common set of features for all editors: feature, scenario, task, etc.

## S1: Increase and decrease font size @v1 [published]

Support Increase and decrease font size. Next to 'H'

## S2: Spell check @v1 [proposed]

Every field that holds prose has the browser's spell checker on: the Markdown editor's Write and Vim tabs, the merge-conflict draft, the agent's message box, a bug report, an issue title, a test note, a proposal's steps. A misspelt word gets the browser's wavy underline and, on right-click, its suggestions; the desktop app offers the suggestions too, with "Add to Dictionary" for a word that is right. Fields that hold urls, names and code (test links, test names, attribute definitions, table cells) keep the checker off, where an underline would only be noise.

- **Given** Settings
- **When** the user turns the Spell check switch off
- **Then** no editor underlines a misspelt word, editors already open included
- **And** the choice is remembered on this device, like Vim mode; on is the default

[test: keep the checker off whatever the switch says: a url is never misspelt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L113 ]

[test: is on until switched off, and the choice is remembered on the device : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L52 ]
[test: tells the browser to check prose in the Vim tab, which CodeMirror turns off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L78 ]
[test: has the checker on by default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L89 ]
[test: follows the switch off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L94 ]
[test: checks the steps a visitor writes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L105 ]
[test: leads with the suggestions, which replace the word when picked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L34 ]
[test: says so when the checker has no guess, and still offers to learn the word : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L57 ]

## S3: web links are cickable @v1 [published]

a webink of the format 'https://abc.com' should be clickable from the editor.
