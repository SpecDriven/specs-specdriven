# Folders

Features can be organized into folders. A folder is a plain directory in the
data repo, so the structure renders naturally on GitHub too.

- The left sidebar displays the folder tree: folders first (collapsible, with
  a disclosure triangle), then features. Features sort alphabetically;
  folders too, unless their parent keeps an order ("order folders" below).
  Features at the repo top level appear at the root of the tree.
- Folders can nest arbitrarily.
- Clicking a folder selects it (shown with an outline) and toggles its
  subtree. Opening a feature or clicking the tree background deselects;
  deleting a selected folder (or an ancestor of it) deselects too. ESC
  deselects as well, putting the insertion point back at the tree root so
  the next "+ New folder" lands at the top level.
- "+ New folder" creates a folder; `/` in the name creates nested folders.
  Folder names are slugified like feature file stems. Git does not track empty
  directories, so a new folder exists locally until a feature lands in it.
- With a folder selected, "+ New folder" creates the new folder inside it —
  the prompt says which folder it will land in. The parent is expanded to
  reveal the new folder, and the new folder becomes the selected one, so
  repeated "+ New folder" clicks build a nested path.
- "+ New feature" accepts `folder/name` to create the feature inside a folder
  (folders are created as needed).
- A feature can be drag-moved from folder to folder (or to the tree root =
  the repo top level). The target folder highlights while dragging over it;
  dropping anywhere on a folder's subtree drops into that folder. The move is
  a `git mv` into the target directory, committed and pushed, keeping the
  feature's file name.
- The moved feature keeps its selection; its url follows the new folder path
  (navigation.feature.md).
- Each folder row carries the same `⋯` menu a feature row does (ui.feature.md
  "Feature entry menu"), visible on hover, on keyboard focus, or with Shift+F10
  / the menu key on the focused row: `New feature here`, `New folder here`,
  `Rename`, and `Delete`. A long folder name truncates with an ellipsis, like a
  feature's, so the `⋯` stays inside the sidebar within reach. The command
  palette offers Rename folder and Delete folder for the selected folder.
- Rename changes the folder's last segment, slugified like a file stem; the
  folder keeps its place in the tree. With features beneath it the rename is a
  `git mv` of the directory, their documents moving in the same commit,
  committed and pushed; every feature's url follows the new folder path. An
  empty folder, untracked by git, is renamed on disk alone.
- Deleting asks for confirmation. An empty folder goes at once, a local
  filesystem operation since git never tracked it, and the toast offers Undo.
  A folder with features beneath it asks again, naming how many go, and then
  removes them all — a `git rm -r`, committed and pushed, like deleting each
  feature from its own menu.
- The Edit link edits the feature name only (ui.feature.md), never the
  file name or its folder; moving a feature between folders is done by
  dragging, or by `Move to folder…` in the feature's `⋯` menu.
- A folder can carry a description: a `description.txt` file inside it,
  whose first non-blank line the sidebar shows in place of the folder name.
  The folder's name is still what sorts it, names its url and shows on
  hover. A folder with a description but no feature is one git tracks, so
  renaming or deleting it is committed and pushed like a folder with
  features; deleting it asks once, like an empty folder.

## S1: delete a folder @v1 [proposed]

- When: user asks to delete a folder with no files anywhere beneath it
- Then: the folder is removed (a local filesystem operation — git does not
  track empty directories), and so is its empty Automerge mirror directory
  under `.specdriven/automerge`, left behind when its last feature moved out
- Otherwise: the deletion is refused unless the user confirms deleting the
  features with it ("delete a folder with features in it")

[test: deleteFolder deletes empty folders only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L135 ]

## S2: delete a folder with features in it @v1 [proposed]

- Given: a folder with features anywhere beneath it
- When: the user picks Delete from the folder's `⋯` menu and confirms the
  prompt that names how many features go with it
- Then: the folder, its features and their documents are removed from the
  repository — a `git rm -r`, committed and pushed — and whatever git never
  tracked beneath it is swept off disk
- And: an editor pane open on one of those features closes without saving it
  back
- Otherwise: without the confirmation nothing changes

[test: deleteFolder with force removes a folder with features in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L165 ]
[test: deleteFolder with force git-rms the directory, commits and pushes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L544 ]
[test: folder routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L184 ]

## S3: describe a folder @v1 [proposed]

- Given: a folder with a `display-name.txt` file in it
- When: the sidebar tree is shown
- Then: the folder's row shows the file's first non-blank line, trimmed, in
  place of the folder name, and the folder name on hover
- And: a folder without the file, or with a blank one, shows its name
- And: a folder with no `display-name.txt` but a `description.txt` — the
  file's name before — shows that file's line the same way
- And: renaming a folder holding only a display name is a `git mv`,
  committed and pushed, and deleting it a `git rm`, committed and pushed,
  with no second confirmation since no feature goes with it

[test: readFolderDescription and listFolderDescriptions read display-name.txt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L301 ]
[test: buildTree carries folder descriptions; folderLabel falls back to the name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L377 ]
[test: renameFolder git-mvs a folder holding only a display-name.txt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L565 ]
[test: deleteFolder removes a folder holding only a display-name.txt, committed and pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L580 ]

## S4: deselect a folder with ESC @v1 [proposed]

- When: a folder is selected and the user presses ESC outside a dialog or text field
- Then: the folder is deselected and "+ New folder" creates at the tree root
- Otherwise: with a dialog open or a field focused, ESC closes/cancels that instead

## S5: edit a folder's display name @v1 [proposed]

- When: the user picks `Edit display name` — beside `Rename` in the folder's
  `⋯` menu, or from the command palette with the folder selected — and
  saves a name
- Then: the name becomes the first non-blank line of the folder's
  `display-name.txt`, any lines past it kept, committed and pushed, and the
  sidebar shows it in place of the folder name ("describe a folder")
- And: the folder itself keeps its name and path, so no link to a feature
  in it changes
- And: a legacy `description.txt` in the folder moves over to
  `display-name.txt` in the same commit
- And: saving a blank name clears the display name — the file goes, a
  `git rm`, committed and pushed — and the folder shows its name again

[test: setFolderDescription writes, keeps extra lines, and clears display-name.txt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L343 ]
[test: setFolderDescription commits and pushes display-name.txt, and git-rms it when cleared : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L598 ]
[test: setFolderDescription moves a legacy description.txt to display-name.txt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L646 ]
[test: folder routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L184 ]

## S6: move a feature from its menu @v1 [proposed]

- When: the user picks `Move to folder…` from a feature's `⋯` menu, or the
  command palette's Move feature to folder…
- Then: a list of every folder, the top level first and the feature's own
  folder left out, is offered, filtered by name as they type
- And: choosing one moves the feature the way a drag does — a `git mv` into
  that folder, committed and pushed, keeping its file name

[test: featuresBeneath, folderRenamed and folderOptions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L216 ]

## S7: move feature to another folder @v1 [proposed]

- When: user drags a feature onto another folder (or the tree root)
- Then: the file is moved with `git mv`, keeping its file name
- and the move is committed and pushed

## S8: rename a folder @v1 [proposed]

- When: the user picks Rename from a folder's `⋯` menu and types a new name
- Then: the folder takes that name, slugified, and keeps its place in the tree
- And: with features beneath it, the directory and its features' documents
  are moved with `git mv` in one commit, pushed; the open feature follows to
  its new url, and a name already taken by a sibling folder is refused
- And: an empty folder is renamed on disk alone, git never having tracked it

[test: renameFolder renames in place, slugified, refusing collisions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L184 ]
[test: renameFolder works in a plain directory with features (no git repo) : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L205 ]
[test: renameFolder git-mvs the directory and its documents, commits and pushes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L521 ]

## S9: order folders @v1 [proposed]

Folders sit where the reader puts them, the way scenarios do ("Number
scenarios and move them up or down" in ui.feature.md).

- When: the user picks `Move up` or `Move down` from a folder's `⋯` menu —
  or from the command palette with the folder selected — or drags a folder
  onto another row among its siblings
- Then: the folder swaps places with its neighbour, or lands where the drop
  line showed it would — just above the row while the pointer is on its
  upper half, just below on its lower half — and the parent folder's
  `folder-order.txt` (`specs/folder-order.txt` for the top level) is written
  with every sibling folder's name, one per line, in the new order,
  committed and pushed
- And: the sidebar sorts a folder's subfolders by that file, the ones it does
  not name after them alphabetically, so a repo without the file reads as
  before; a name with no folder behind it orders nothing
- And: a renamed folder keeps its place — the parent's order names the new
  folder in a commit of its own — and a folder with only an order file in it
  is one git tracks, deleted like a folder holding only a display name
- `Move up` is unavailable on the first folder and `Move down` on the last.
  A folder dragged onto a folder with another parent, onto a feature or onto
  a scenario drops nowhere: moving a feature between folders is still the
  feature's own drag, and a folder never moves into another folder this way.

[test: readFolderOrder and listFolderOrders read folder-order.txt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L398 ]
[test: setFolderOrder writes folder-order.txt, refusing names that are not subfolders : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L425 ]
[test: buildTree orders subfolders by folder order, the rest by name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L454 ]
[test: folderNamesMoved and folderDrop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L469 ]
[test: setFolderOrder commits and pushes folder-order.txt; renameFolder keeps the place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L621 ]
[test: folder routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L184 ]
