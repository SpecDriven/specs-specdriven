# Common Editor

Common set of features for all editors: feature, scenario, task, etc.

## S1: Increase and decrease font size @v1 [published]

Support Increase and decrease font size. Next to 'H'

## S2: Spell check @v1 [proposed]

Every field that holds prose has the browser's spell checker on: the Markdown editor's Write and Vim tabs, the merge-conflict draft, the agent's message box, a bug report, an issue title, a test note. A misspelt word gets the browser's wavy underline and, on right-click, its suggestions; the desktop app offers the suggestions too, with "Add to Dictionary" for a word that is right. Fields that hold urls, names and code (test links, test names, attribute definitions, table cells) keep the checker off, where an underline would only be noise.

- **Given** Settings
- **When** the user turns the Spell check switch off
- **Then** no editor underlines a misspelt word, editors already open included
- **And** the choice is remembered on this device, like Vim mode; on is the default

[test: keep the checker off whatever the switch says, since a url is never misspelt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L160 ]
[test: is on when storage is unavailable, and a flip does not throw : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L61 ]
[test: shows the checker's first few guesses only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L68 ]
[test: has the edit verbs in any editable field : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L77 ]
[test: offers Copy over a selection in the text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L82 ]
[test: is nothing over plain text, or a selection of only whitespace : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L86 ]
[test: is on until switched off, and the choice is remembered on the device : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L49 ]
[test: tells the browser to check prose in the Vim tab, which CodeMirror turns off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L75 ]
[test: has the checker on by default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L149 ]
[test: follows the switch off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/spellCheck.test.tsx#L153 ]
[test: leads with the suggestions, which replace the word when picked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L34 ]
[test: says so when the checker has no guess, and still offers to learn the word : https://github.com/SpecDriven/specdriven-app/blob/main/tests/contextMenu.test.ts#L57 ]

## S3: web links are cickable @v1 [published]

a webink of the format 'https://abc.com' should be clickable from the editor.

## S4: Font color @v1 [proposed]

A Font color button sits next to Bold in the editor's toolbar. It opens a menu of colors that read on the light theme and the dark one alike, plus Default color. Picking a color wraps the selected text in `<span style="color: …">…</span>`, the HTML most markdown tools render, since markdown has no color of its own; the Preview tab and pages published to Confluence show the text in that color. Picking a color for text that is already colored recolors it rather than nesting another span, and Default color takes the color off.

The Write tab shows colored text in its color too, not as the span that carries it: the span's tags are hidden, and show — receding, beside the text — only while the caret is at either end of the span or in a tag, never while it is in the text, so a click or a double-click in the word never shifts the word. The caret steps over a hidden tag to the span's edge, where the tags show and can be edited like any other markup.

- **Given** the Write tab with some text selected
- **When** the user clicks Font color and picks Red
- **Then** the selection is wrapped in a red span and stays selected
- **And** the Write tab shows the text in red, hiding the span's tags while the caret is in the text
- **And** the Preview tab shows it in red

[test: the button sits next to the Bold button : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L603 ]
[test: picking a color colors the selection; Default color takes it off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L610 ]
[test: ESC closes the menu and leaves the text alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L715 ]
[test: the Write tab shows the text in its color, with the tags hidden until the caret is on it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L654 ]
[test: the caret steps over a hidden tag, to the span's edge, where the tags then show : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L683 ]
[test: wraps the selection in a colored span, leaving the text selected : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L427 ]
[test: with nothing selected, leaves the caret inside an empty colored span : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L435 ]
[test: recolors a span the selection is in, rather than nesting another : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L443 ]
[test: recolors a span selected whole, tags and all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L451 ]
[test: Default color takes the span off, and leaves plain text alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L458 ]
[test: offers colors every one of which is a color the preview accepts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L467 ]
[test: shows a colored span in its color, with the markdown inside it read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L513 ]
[test: reads no other style, and no color that could break out of one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L531 ]
[test: colored text keeps its color, the way Confluence stores one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L52 ]

## S5: Highlight @v1 [proposed]

A Highlight button sits next to Font color. It opens a menu of highlights — translucent colors that read on the light theme and the dark one alike, over any text color — plus No highlight. Picking one puts the color behind the selected text, as `<span style="background-color: …">…</span>`; the Write tab, the Preview tab and pages published to Confluence all show it. Text that is colored and highlighted carries both in one span (`color: …; background-color: …`): a highlight joins a colored span rather than nesting another, and a color joins a highlighted one. No highlight takes the highlight off, and the span with it once nothing is left to carry.

- **Given** the Write tab with some text selected
- **When** the user clicks Highlight and picks Yellow
- **Then** the selection is highlighted in yellow and stays selected
- **And** the Write and Preview tabs show the text on yellow

[test: the Highlight button sits next to Font color, and its menu highlights the selection : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L631 ]
[test: wraps the selection in a highlighted span, leaving the text selected : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L477 ]
[test: joins a highlight to a colored span, and a color to a highlighted one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L485 ]
[test: No highlight takes the highlight off, and the span with it once nothing is left : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L496 ]
[test: offers highlights every one of which is a color the preview accepts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L506 ]
[test: shows a highlighted span with its color behind the text, and both together : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L522 ]
[test: highlighted text keeps its highlight, beside its color : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L59 ]

## S6: Undo and redo @v1 [proposed]

The Write and Vim tabs keep a history of edits: ⌘Z (Ctrl-Z off the Mac) takes the last one back and ⇧⌘Z (Ctrl-Y, or Ctrl-Shift-Z) brings it back. An edit the toolbar made — a color, a highlight, a link, a table — comes back whole, in one step, as does typing. Undo and Redo buttons lead the toolbar for anyone who does not know the keys. The history is the tab's own: it starts over when the editor is reopened or the tab changed.

- **Given** the Write tab, after Font color painted a word red
- **When** the user presses ⌘Z
- **Then** the word is plain again, in one step
- **And** ⇧⌘Z paints it red again

[test: ⌘Z takes a toolbar edit back whole, ⇧⌘Z brings it back, and the toolbar has both : https://github.com/SpecDriven/specdriven-app/blob/main/tests/commonEditor.test.tsx#L698 ]
