# Feature Editor

## Edit Description button opens description editor @v1 [proposed]

Given: A feature exists with a description available.
When: The user hits the "Edit Description" button in the top right.
Then: The description editting pane opens and defaults to "Write" mode.

## Edit feature descrition @v1 [proposed]

Editor for the feature description must be on after a new feature is created.

## The description lands in the folder it was typed in @v1 [proposed]

- Given: a feature is open with its description editor on, and text typed
  since the last autosave tick
- When: the user switches to another data folder — from the recent folder
  list, or the Data directory dialog
- Then: what was typed is saved into the folder it was typed in, before the
  switch — never into the new folder
- and the editor empties with the selection, so no later tick writes the old
  folder's description into the new one
- and a flush that meets an autosave already on its way waits for it, then
  writes what the editor still holds, rather than being skipped
