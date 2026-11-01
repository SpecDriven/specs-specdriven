# Roadmap

**Assigned:** David

'roadmap' is a special folder which contains regular markdown files. It shows in the sidebar as a Roadmap tab beside the Specs tab. User settings will control the visibility of this folder

## S1: Create roadmap folder if necessary @v1 [published]

If user setting for Roadmap is On and the 'roadmap' folder does not exist -- create it.
On Sync, commit all files in this folder.

[test: the Roadmap setting is off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmap.test.ts#L61 ]
[test: ensurePageFolder creates the roadmap folder once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmap.test.ts#L69 ]
[test: a sync tick creates the folder only when the setting is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmap.test.ts#L79 ]
[test: a sync tick commits and pushes every file in the roadmap folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmap.test.ts#L95 ]

## S2: Reorder roadmap pages @v1 [proposed]

- Given: the Roadmap tab lists its pages, top level first, then each subfolder
- When: the user picks Move up or Move down from a page's ⋯ menu, or drags
  the page onto another page's row
- Then: the page moves one place up or down within its folder — or lands
  before or after the row it was dropped on — just as tasks do
  (tasks.feature.md "Reorder tasks by dragging")
- and Move down on the next-to-last page puts it at the bottom; Move up is
  greyed out on the first page and Move down on the last
- and Move to folder… and a drop on a subfolder's header move the page into
  that folder, a `git mv` committed and pushed with its document
- and the order is kept in an `order.txt` beside the pages, one file name per
  line, which the sync tick commits; a page the order does not name is listed
  after the named ones, in name order
- and each subfolder gets the task folders' ⋯ menu — New page here, New
  folder here, Rename, Delete — with a "+ New folder" button beside
  "+ New page"

[test: the roadmap tab and folder keep an order of their own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapOrder.test.ts#L48 ]
[test: roadmap pages move up and down one place, the next-to-last to the bottom : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapOrder.test.ts#L53 ]
[test: the roadmap routes list folders and move a page to the bottom : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapOrder.test.ts#L98 ]

## S3: The roadmap follows the data folder @v1 [proposed]

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

## S4: The roadmap is a sidebar tab @v1 [proposed]

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
  subfolder, each page with its Rename / Copy link / Delete menu — Rename
  changes the page's heading and its file name together, a `git mv` committed
  and pushed with the page's document alongside, so the url follows the title
  and the retired name is free again; the collapsible Roadmap section above
  the tree, its twist and its small + button are gone
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

## S5: Archive a roadmap page @v1 [proposed]

- Given: a roadmap page that is finished with, but worth keeping
- When: the user picks Archive from its ⋯ menu
- Then: the page leaves the Roadmap list and its markdown file is removed
  from the data repo, while its Automerge document stays — marked archived,
  with when and by whom — as the only copy; the removal and the document
  are committed and pushed together, so a teammate's sync never drops the
  document for a page whose file is gone
- and there is no confirmation, just an Undo on the toast, which restores
  the page; an open archived page closes and the app goes home
- and a collapsed "Archived" head with a count sits at the foot of the
  Roadmap list; opened, it lists the archived pages, most recently archived
  first, each with its day and a ⋯ menu of Restore and Delete
- and Restore takes the mark off the document and writes the markdown back
  under the file name it had — its folder made again if it went meanwhile —
  at the bottom of its folder, committed and pushed
- and Delete, confirmed first, removes the archived document for good
- and no new, renamed or moved page can take an archived page's file name,
  and an editor still open on it saves nothing back
- and deleting a roadmap folder leaves the archived pages that were in it
- and a markdown file put back beside an archived document by hand brings
  the page back on the next sync
- and only the roadmap's pages are archived; the read-only site lists none

[test: archiving removes the markdown and keeps the document, in one pushed commit : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L54 ]
[test: a restored page comes back as it was, last in its folder, pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L87 ]
[test: an archived page is deleted for good with its document, pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L115 ]
[test: archiving takes in an edit the sync tick has not absorbed yet : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L129 ]
[test: an archived page's name is not taken by another page, nor saved over : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L144 ]
[test: markdown put back beside an archived document brings the page back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L161 ]
[test: deleting a folder keeps the archived pages that were in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L172 ]
[test: only the roadmap's pages are archived : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L185 ]
[test: the roadmap routes archive, list, restore and delete a page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/roadmapArchive.test.ts#L191 ]
