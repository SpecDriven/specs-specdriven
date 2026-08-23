# App Startup

## A newly picked data folder opens on its README @v1 [proposed]

- Given: the user confirms a data folder — in the first-run form or in the
  Data directory dialog
- When: the folder is accepted
- Then: the app opens on the folder's README.md in the editor, exactly as a
  start with that folder already selected does — not on the first spec in
  the list
- and a deep link on first run still wins over the README
- and a folder with no README.md opens on an empty editor; the first
  autosave of anything typed there creates the file

## App Starts with existing specs folder selected @v1 [published]

Given: specs filder is saved in .specdriven
Then: Open the README.md file in an editor so it can be edited if necessary.
We will automatically save the README.md file using the same frequesncy as we use for scenario editor.
