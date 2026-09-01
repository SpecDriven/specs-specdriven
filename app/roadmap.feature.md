# Roadmap

**Assigned:** David

'roadmap' is a special folder which contains regular markdown files. This folder will be on top of feature folders. User settings will control the visibility of this folder

## Create roadmap folder if necessary @v1 [published]

If user setting for Roadmap is On and the 'roadmap' folder does not exist -- create it.
On Sync, commit all files in this folder.

## The roadmap follows the data folder @v1 [proposed]

- Given: the Roadmap setting is on, and one of the current data folder's
  roadmap pages is open in the editor
- When: the user switches to another data folder — from the recent folder
  list, or the Data directory dialog
- Then: the sidebar's Roadmap section lists the new folder's pages, not the
  ones it was showing; a new folder with no pages shows the empty section
- and the open page closes and the app goes home to the new folder's README,
  the way the open feature is dropped
- and whatever was typed in that page is saved into the folder it was typed
  in, before the switch — never into the new folder
- and the address bar returns to `/`, so a reload does not ask the new folder
  for a page that belonged to the old one
- and a folder picked in the first-run form lists its pages the same way
