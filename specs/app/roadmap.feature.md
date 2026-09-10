# Roadmap

**Assigned:** David

'roadmap' is a special folder which contains regular markdown files. It shows in the sidebar as a Roadmap tab beside the Specs tab. User settings will control the visibility of this folder

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

## The roadmap is a sidebar tab @v1 [proposed]

- Given: the Roadmap setting is on
- When: the user looks at the sidebar
- Then: under the search field sit two tabs, Specs and Roadmap; the Home,
  Roadmap and My specs links that used to sit there are gone — the data
  folder name in the header is the way home, and the Features filter's
  assignee list stands in for My specs
- and the Specs tab holds what the sidebar held before: the New folder and
  New feature buttons, Start session, the Features head with its filter, and
  the tree
- and the Roadmap tab holds a "+ New page" button and the page list grouped by
  subfolder, each page with its Rename / Copy link / Delete menu; the
  collapsible Roadmap section above the tree, its twist and its small + button
  are gone
- and the chosen tab is remembered across reloads, the way the sidebar width
  is
- and opening a roadmap page — from its row, the command palette or a deep
  link — turns the sidebar to the Roadmap tab, while opening a feature turns
  it to the Specs tab; a refresh of the open page from sync leaves the tab
  alone
- and with the Roadmap setting off there is no tab bar, just the Specs
  content
- and on a small screen the tabs sit in the drawer, and switching tabs keeps
  the drawer open
