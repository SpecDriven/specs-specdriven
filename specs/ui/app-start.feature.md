# App start

When the application starts, if the settings file does not
contain the name of the last data repo, the user is asked the name of the data repo.

Data repo format: https://github.com/SpecDriven/data-insurance (regular gitHub url)

The data repo is pulled — one Sync, the same tick the Sync button and the
reload cron run — when the app starts and whenever a new data folder is
picked, so the app never opens on data the last run left behind.

## Settings file

Local state lives in `.specdriven/settings.json`, looked up in this order:

- `./.specdriven/settings.json`, relative to the directory the app runs from
- `~/.specdriven/settings.json`, so one settings file can serve every checkout

`SPECDRIVEN_HOME` overrides that home directory — the same variable install.sh
uses for the install prefix. Settings are saved back to whichever of the two
the app read; when neither exists, the project one is created.

## a newly picked data folder is pulled @v1 [proposed]

- Given: the user confirms a data folder — in the first-run form or in the
  Data directory dialog
- When: the folder is accepted
- Then: it is pulled before the app reads it, so the folder's first feature
  list is the latest data
- and a folder that is not a git repo, or whose pull fails, is still selected

## changing the data repo @v1 [proposed]

- Given: a data repo is configured, and the sidebar header shows its folder
  name only — parent folders appear in a tooltip, and the full path in the
  edit dialog
- When: the user clicks the edit (✎) button next to the name
- Then: a dialog opens pre-filled with the current data repo
- and confirming a new value switches the app to that repo, resetting the
  view to home (the old selection belonged to the old repo)
- and a value the server rejects leaves the current repo in place, with the
  error shown as a status note

## forgetting a data repo @v1 [proposed]

- Given: a data repo is configured, and the sidebar header shows its folder
  name only — parent folders appear in a tooltip, and the full path in the
  edit dialog

- When: the user clicks the edit (✎) button next to the name
- Then: a dialog opens pre-filled with the current data repo
- A button in this dialog causes the app to stop showing this existing data repo in the dropdown

## migrating the pre-settings.json layout @v1 [proposed]

- Given: a plain `.specdriven` file (the old layout) is in the project root
- When: the app starts
- Then: it is read as the settings file
- and the next save moves it to `./.specdriven/settings.json`, preserving
  every setting

## no settings file anywhere @v1 [proposed]

- Given: neither settings file exists
- When: the app starts
- Then: the user is asked for the data repo

## only the home settings file exists @v1 [proposed]

- Given: `~/.specdriven/settings.json` names a data repo, and the project has
  no settings file
- When: the app starts
- Then: the user is not asked for a data repo
- and settings saved from the Settings page go back to the home settings file

## picking the data directory with the native folder chooser @v1 [proposed]

- Given: the runtime has a native folder chooser (the desktop app, or the
  local web server on macOS)
- When: the user clicks "Browse…" next to the data directory field — in the
  first-run form or in the Data directory dialog
- Then: the OS folder chooser opens (parented to the window in the desktop
  app; a Finder chooser owned by the local server in web mode)
- and picking a folder fills the field with its absolute path, leaving the
  user to confirm
- and canceling the chooser leaves the field unchanged
- and on runtimes with no native chooser the "Browse…" button is not shown

[test: macOS and Windows get a chooser out of the box, unknown platforms none : https://github.com/SpecDriven/specdriven_app/blob/main/tests/folderChooser.test.ts#L46 ]
[test: canChooseFolder reflects whether a chooser is installed : https://github.com/SpecDriven/specdriven_app/blob/main/tests/folderChooser.test.ts#L100 ]
[test: chooseFolder resolves the picked path, null when canceled : https://github.com/SpecDriven/specdriven_app/blob/main/tests/folderChooser.test.ts#L106 ]
[test: POST /api/choose-folder returns the picked path : https://github.com/SpecDriven/specdriven_app/blob/main/tests/folderChooser.test.ts#L135 ]
[test: POST /api/choose-folder is 501 with no native chooser : https://github.com/SpecDriven/specdriven_app/blob/main/tests/folderChooser.test.ts#L144 ]

## the data repo is pulled on start @v1 [proposed]

- Given: a data repo is configured, and its remote has commits this machine
  does not have yet
- When: the app starts
- Then: the data repo is pulled before the first screen is read, so the
  feature list shows the remote's latest data without the user pressing Sync
- and the pull does not hold up the rest of startup; a pull that hangs stops
  blocking reads after a short wait, leaving stale data on screen until the
  next reload tick rather than an app that never opens
- and a pull that fails is logged; the app still opens on the data on disk

## the project settings file wins @v1 [proposed]

- Given: both settings files exist
- When: the app starts
- Then: the project settings file is used
- and saving settings leaves the home settings file untouched
