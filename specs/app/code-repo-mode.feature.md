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
- and the folder list ("Spec folders") has a "Code repositories" section:
  the code repositories opened before, most recent first, each one openable
  and removable from the list (the folder itself stays on disk), plus
  "Open a code repository…"
- and the open folder, spec or code, is badged and cannot be removed
- and a path that names no folder is refused, saying so
- and the demo, the hosted app and the read-only site offer none of this,
  and refuse it if asked

## S2: The app runs no git in a code repository @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the reload interval passes, or the app starts with it open
- Then: nothing is pulled, committed or pushed, and the app writes no file
  into the repository besides the ones the user saves
- and Sync, "Sync now" and "Change data folder" are not offered
- and plugins found in the repository's `.specdriven/plugins` folder are not
  loaded
- and the routes that read or write specifications answer that no
  specifications folder is open

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

## S5: Open the file in IntelliJ IDEA @v1 [proposed]

**Created by:** vydra

- Given: a file open in the editor pane
- When: the user presses "Open in IntelliJ"
- Then: IntelliJ IDEA opens the file at the line the cursor is on (the
  `idea` launcher, as for test links)
- and when `idea` is not on the PATH, the app says so with the one-line
  command that makes the symlink
- and the button works for a file the editor could not show as text

## S6: The coding agent works in the code repository @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user starts a coding agent session
- Then: the agent works in the repository's folder, and its prompt does not
  point it at a specifications folder
- and the Worktree option still runs the session in a worktree of its own
  (the one git command made in the repository, at the user's request)

## S7: Switching back to a specifications folder @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user opens a specifications folder from the folder list
- Then: the Specs tab, the modules' tabs and Sync come back, and the folder
  is synced as on any folder switch
- and the code repository stays on the list for next time
