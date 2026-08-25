# Integration with Claude agent

## Create a prompt from selected specs @v1 [published]

Given: a code session has been created with at least one spec in it
When: we start a code gen session with Agent (Claude)
Then: we generate a prompt and pass it to the Claude Cli to execute and exit

If error code, display the error to the user in an error box.

## Create and name a code gen session @v1 [published]

- Select one or more specs
- 'Start session' (click button)
- Name the session

## Create and name a code gen session @v2 [published]

- Select a whole spec with the checkbox on its sidebar entry
- Select single scenarios with the checkbox on their scenario cards
- A spec whose scenarios are partly selected shows a dash in the sidebar;
  selecting every scenario of a spec is the same as selecting the spec
- The selection holds while moving between specs, and what it adds up to is
  shown next to 'Start session'
- 'Start session' (click button)
- Name the session
- A spec cut down to selected scenarios is marked as such in the prompt

## S4 @v1 [published]

fsfdsfdsfs

## Switch code generation on or off @v1 [published]

Given: the Settings dialog
When: the 'Code generation' switch is turned off and the settings are saved
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.
