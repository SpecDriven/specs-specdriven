# Code repository mode

**Created by:** vydra

The app normally opens a specifications folder: the Specs tab, the special
folders beside it, and a Sync with GitHub every minute. The coding agent pane
is useful without any of that, so the app can also open a **code repository
directly**, the way an IDE opens a project.

In that mode the app is a guest in the developer's repository:

- it runs no git command of its own there — no pull, commit or push, no
  `specs/` folder, no `.specdriven/` folder, no `project-settings.json`; every
  git operation is the developer's
- the Specs tab and the specification modules' tabs (Roadmap, Tasks,
  Notebook, Issues, Tests) are hidden, and so are Sync and the data-folder
  actions
- the navigation pane lists the repository's files like an IDE
- the editor pane is a plain text editor, with a button that opens the file
  in IntelliJ IDEA — most developers will edit in their own IDE
- the coding agent pane works in the repository

The mode is chosen, never guessed from the folder's contents: a code
repository linked to a spec repo carries a `specs` link, and a spec repo may
hold code. It is offered only where the app runs on the developer's own
machine — not in the demo, the hosted app or the read-only site.

## S1: Open a code repository @v1 [proposed]

**Created by:** vydra

- Given: the app on the developer's own machine
- When: the first-run screen asks where the specs are and the user picks
  "A code repository", then names a folder (typed, or with Browse…)
- Then: the app opens that folder in code repository mode
- and the code repositories opened before are on the same folder list as
  the specification folders (Manage folders and the recent folders menu),
  most recent first, each marked by a code-folder icon, openable and
  removable from the list (the folder itself stays on disk), plus
  "Open a code repository…"
- and the open folder, spec or code, is badged and cannot be removed
- and a path that names no folder is refused, saying so
- and the demo, the hosted app and the read-only site offer none of this,
  and refuse it if asked

[test: PUT /api/state openCodeRepo opens the folder and GET /api/state reports the mode : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L252 ]
[test: a code repository must be an existing folder named by its absolute path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L270 ]
[test: the demo and the hosted app open no code repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L281 ]
[test: an open code repository takes the place of the specifications folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L115 ]
[test: the code repositories list keeps the open one first, each once, and forgets on request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L139 ]
[test: another code repository can be opened from the open one, and the old one forgotten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L352 ]
[test: with a code repository open, the last specifications folder can be removed from the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L364 ]
[test: spec folders and code repositories share one list, in the given order, each marked by its kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L163 ]
[test: the open folder is badged whether it is a spec folder or a code repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L174 ]
[test: where code repositories cannot be opened, the dialog has none of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L202 ]

## S2: The app runs no git in a code repository @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the reload interval passes, or the app starts with it open
- Then: nothing is pulled, committed or pushed, and the app writes no file
  into the repository besides the ones the user saves
- and Sync and "Sync now" are not offered
- and plugins found in the repository's `.specdriven/plugins` folder are not
  loaded
- and the routes that read or write specifications answer that no
  specifications folder is open

[test: a sync runs no git in the code repository and writes nothing into it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L303 ]
[test: the specification routes answer that no specifications folder is open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L294 ]
[test: an open code repository mounts the Files tab and none of the specification modules : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L254 ]

## S3: The navigation pane lists the repository's files @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user looks at the navigation pane
- Then: it shows a Files tab instead of the Specs tab: the repository's
  folders and files as a tree, folders first, each folder opened and closed
  in place
- and `.git` is never listed, nor anything the repository's `.gitignore`
  files exclude (`node_modules`, build output); without git on the machine,
  everything but `.git` is listed
- and the folders left open are remembered per repository
- and Refresh lists them again, as does coming back to the window — an agent
  or the IDE may have added files

[test: the files routes are registered with their methods : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L76 ]
[test: outside code repository mode the files routes have no folder to serve : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L85 ]
[test: a path that leaves the repository or names git's folder is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L99 ]
[test: the routes refuse a path outside the repository with 400 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L108 ]
[test: a folder lists folders first, in natural order, without .git or what is ignored : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L117 ]
[test: a link is listed as what it points to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L126 ]
[test: a folder that is not there is 404, a file is not a folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L132 ]
[test: git check-ignore decides what a git repository ignores, tracked files excepted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L137 ]
[test: the store lists the top folder, and a folder when it is opened : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L89 ]
[test: the tree is treeitems, folders opening in place, files linking to their url, depth by level : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L198 ]
[test: a folder that cannot be read says why, and an empty repository says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L211 ]

## S4: A file opens in a plain text editor @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user picks a file in the tree
- Then: the editor pane shows its path and its text in a plain editor with
  line numbers, and its url (`/files/<path>`) opens it again
- and edits are saved on the autosave interval, at once with ⌘S, and when
  the user leaves the file, keeping the file's line endings
- and a file changed on disk since it was opened (by the IDE or an agent) is
  not overwritten: the editor says so and offers to reload it or save over it
- and a binary file, or one too large to edit here, is not opened as text;
  the editor says why
- and the home view opens the repository's README.md when it has one

[test: the tab's urls are home at / and /files, and /files/<path> for a file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L24 ]
[test: a file opens with its text, time and line endings : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L155 ]
[test: a missing, binary, non-UTF-8 or too large file is not opened as text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L166 ]
[test: a save writes the text in place, keeping the file's permissions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L179 ]
[test: a save keeps Windows line endings : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L195 ]
[test: a file changed on disk since it was opened is not overwritten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L202 ]
[test: a file deleted on disk since it was opened is not silently made again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L218 ]
[test: a save with no text is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L225 ]
[test: Markdown files get Markdown's highlighting, every other file is plain text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L36 ]
[test: a file opens, saves what changed with its time and line endings, and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L106 ]
[test: a file changed on disk is not overwritten until the user says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L128 ]
[test: Reload from disk drops the editor's text for the file as it is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L149 ]
[test: opening another file saves the one it replaces; leaving saves it too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L161 ]
[test: a file the server will not show as text says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L179 ]
[test: the status words follow the file's state : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L233 ]
[test: the tab's home opens the README when the repository has one; search finds loaded files : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L278 ]

## S5: Open the file in IntelliJ IDEA @v1 [proposed]

**Created by:** vydra

- Given: a file open in the editor pane
- When: the user presses "Open in IntelliJ"
- Then: IntelliJ IDEA opens the file at the line the cursor is on (the
  `idea` launcher, as for test links)
- and when `idea` is not on the PATH, the app says so with the one-line
  command that makes the symlink
- and the button works for a file the editor could not show as text

[test: idea gets --line only when a line is given : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L232 ]
[test: Open in IntelliJ runs idea on the file, at the cursor's line : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L237 ]
[test: Open in IntelliJ works for a file the editor cannot show : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L245 ]
[test: without idea on PATH the click is refused with the symlink to make : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L251 ]
[test: Open in IntelliJ refuses a bad request before anything runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L261 ]
[test: a launcher that fails to start is reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesApi.test.ts#L269 ]
[test: the pane's header names the file's folders and offers Open in IntelliJ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesTab.test.tsx#L220 ]

## S6: The coding agent works in the code repository @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user starts a coding agent session
- Then: the agent works in the repository's folder, and its prompt does not
  point it at a specifications folder
- and the Worktree option still runs the session in a worktree of its own
  (the one git command made in the repository, at the user's request)

[test: a coding agent session runs in the code repository, with no specifications folder behind it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L377 ]
[test: a folder the pane picks inside the code repository is where the session runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L389 ]
[test: a code gen session from the spec dialog needs a specifications folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L396 ]
[test: a code repository keeps a plugin list of its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L240 ]

## S7: Switching back to a specifications folder @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user opens a specifications folder from the folder list
- Then: the Specs tab, the modules' tabs and Sync come back, and the folder
  is synced as on any folder switch
- and the code repository stays on the list for next time

[test: a specifications folder takes the place of the code repository, which stays on the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L318 ]
