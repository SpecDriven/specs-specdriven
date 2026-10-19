# Folders

Features can be organized into folders. A folder is a plain directory in the
data repo, so the structure renders naturally on GitHub too.

- The left sidebar displays the folder tree: folders first (collapsible, with
  a disclosure triangle), then features, both sorted alphabetically. Features
  at the repo top level appear at the root of the tree.
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

## delete a folder @v1 [proposed]

- When: user asks to delete a folder with no files anywhere beneath it
- Then: the folder is removed (a local filesystem operation — git does not
  track empty directories), and so is its empty Automerge mirror directory
  under `.specdriven/automerge`, left behind when its last feature moved out
- Otherwise: the deletion is refused unless the user confirms deleting the
  features with it ("delete a folder with features in it")

[test: deleteFolder deletes empty folders only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L120 ]

## delete a folder with features in it @v1 [proposed]

- Given: a folder with features anywhere beneath it
- When: the user picks Delete from the folder's `⋯` menu and confirms the
  prompt that names how many features go with it
- Then: the folder, its features and their documents are removed from the
  repository — a `git rm -r`, committed and pushed — and whatever git never
  tracked beneath it is swept off disk
- And: an editor pane open on one of those features closes without saving it
  back
- Otherwise: without the confirmation nothing changes

[test: deleteFolder with force removes a folder with features in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L150 ]
[test: deleteFolder with force git-rms the directory, commits and pushes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L340 ]
[test: folder routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L183 ]

## deselect a folder with ESC @v1 [proposed]

- When: a folder is selected and the user presses ESC outside a dialog or text field
- Then: the folder is deselected and "+ New folder" creates at the tree root
- Otherwise: with a dialog open or a field focused, ESC closes/cancels that instead

## move a feature from its menu @v1 [proposed]

- When: the user picks `Move to folder…` from a feature's `⋯` menu, or the
  command palette's Move feature to folder…
- Then: a list of every folder, the top level first and the feature's own
  folder left out, is offered, filtered by name as they type
- And: choosing one moves the feature the way a drag does — a `git mv` into
  that folder, committed and pushed, keeping its file name

[test: featuresBeneath, folderRenamed and folderOptions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L201 ]

## move feature to another folder @v1 [proposed]

- When: user drags a feature onto another folder (or the tree root)
- Then: the file is moved with `git mv`, keeping its file name
- and the move is committed and pushed

## rename a folder @v1 [proposed]

- When: the user picks Rename from a folder's `⋯` menu and types a new name
- Then: the folder takes that name, slugified, and keeps its place in the tree
- And: with features beneath it, the directory and its features' documents
  are moved with `git mv` in one commit, pushed; the open feature follows to
  its new url, and a name already taken by a sibling folder is refused
- And: an empty folder is renamed on disk alone, git never having tracked it

[test: renameFolder renames in place, slugified, refusing collisions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L169 ]
[test: renameFolder works in a plain directory with features (no git repo) : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L190 ]
[test: renameFolder git-mvs the directory and its documents, commits and pushes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folders.test.ts#L317 ]
