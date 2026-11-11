# App start

**Created by:** vydra

**Updated by:** vydra

When the application starts, if the settings file does not
contain the name of the last data repo, the user is asked the name of the data repo.

The first-run screen offers two ways in: open a folder on this computer —
a specification folder or a code repository, in one field, told apart by
what is inside —
or clone a GitHub repository with a personal access token — so someone with
neither git nor the GitHub CLI installed, and without the GitHub App
sign-in, can still get a repository they can push to. The built-in git
engine (git-operations.md) does the cloning and pushing.

Data repo format: https://github.com/SpecDriven/data-insurance (regular gitHub url),
or the same url on a GitHub Enterprise host, e.g. https://github.acme.com/team/specs
or https://acme.ghe.com/team/specs

The data repo is pulled — one Sync, the same tick the Sync button and the
reload cron run — when the app starts and whenever a new data folder is
picked, so the app never opens on data the last run left behind.

## Settings file

Local state lives in `.specdriven/settings.yaml`, looked up in this order:

- `./.specdriven/settings.yaml`, relative to the directory the app runs from
- `~/.specdriven/settings.yaml`, so one settings file can serve every checkout

`SPECDRIVEN_HOME` overrides that home directory — the same variable install.sh
uses for the install prefix. Settings are saved back to whichever of the two
the app read; when neither exists, the project one is created.

Two runs use the home file only: the desktop app (desktop-app.feature.md
"Data locations"), and a run from the app's own source checkout (the
specdriven-app repository or a worktree of it), so every checkout, worktree
and the desktop app share one settings file.

The file is YAML. It used to be JSON, `settings.json` in the same two
places: one still there is read as it is, and converted to
`settings.yaml` when the app starts, so an existing installation keeps its
settings. The JSON file is kept beside it as `settings.json.bak`.

The file says which version of its format it is in (`settingsVersion`; a file
without one is version 0). A new version of the app that changes what the
file stores upgrades an older file when it starts, keeping the file as it
was beside it; a file a newer app wrote is read as it is.

## S1: a clone without a token is refused when signed out @v1 [proposed]

- Given: the app is not signed in to the repository's GitHub host
- When: the user chooses "Clone from GitHub" and leaves the token empty
- Then: the form says a token is needed, since a clone without one could
  never push the edits back
- and nothing is cloned

## S2: a GitHub Enterprise repository is cloned with a token @v1 [proposed]

- Given: a repository url on a GitHub Enterprise host, like
  https://github.acme.com/team/specs or https://acme.ghe.com/team/specs
- When: the user clones it from the first-run form with a token made on
  that host
- Then: the token is checked against that host's API — `/api/v3` on a
  GitHub Enterprise Server, `api.<name>.ghe.com` on GHE.com
- and the repository is cloned, pulled and pushed with it, also on the
  built-in engine with no git installed
- and the token is never offered to any other host, on the built-in engine
  or on system git
- and a GitHub sign-in saved before GitHub Enterprise support keeps working
  as a github.com one

[test: GitHub hosts map to their web, API and new-token urls : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L90 ]
[test: GitHub Enterprise repository urls parse with their host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L100 ]
[test: a pasted token signs in to a GitHub Enterprise host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L123 ]
[test: the built-in engine clones and pushes to GitHub Enterprise with the token, with no git installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L166 ]
[test: the token is never offered to another host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L186 ]
[test: system git hands the token to the sign-in's host through the scoped helper : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L201 ]
[test: a record saved before GitHub Enterprise reads as github.com : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L116 ]

## S3: a newly picked data folder is pulled @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the user confirms a data folder — in the first-run form or in the
  Data directory dialog
- When: the folder is accepted
- Then: it is pulled before the app reads it, so the folder's first feature
  list is the latest data
- and a folder that is not a git repo, or whose pull fails, is still selected

[test: picking a data folder pulls it before the client reads it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartSync.test.ts#L91 ]
[test: a data folder that is not a git repo is still selectable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartSync.test.ts#L110 ]

## S4: a profile folder keeps a dev run's settings apart @v1 [proposed]

- Given: a developer starts the app with `--profile <dir>` (e.g.
  `bun run dev --profile /tmp/try`)
- When: the app starts
- Then: settings live in `<dir>/.specdriven/`, the home folder — GitHub
  token, logs — in `<dir>/home`, and GitHub clones in `<dir>/repos`, all
  created if missing
- and the first-run screen appears for a new folder, without reading or
  changing the real settings, token or clones
- and `SPECDRIVEN_ROOT`, `SPECDRIVEN_HOME` or `SPECDRIVEN_REPOS_DIR` set
  explicitly still win

[test: profileArg reads --profile <dir> and --profile=<dir> : https://github.com/SpecDriven/specdriven-app/blob/main/tests/profile.test.ts#L18 ]
[test: --profile points settings, home and clones into the folder and makes them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/profile.test.ts#L25 ]
[test: a variable set explicitly wins over --profile; no flag changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/profile.test.ts#L38 ]

## S5: a rejected token is not kept @v1 [proposed]

- When: the user pastes a token GitHub does not accept — mistyped, expired
  or revoked — in the first-run form or in Settings
- Then: the form says "GitHub did not accept this token — check it and its
  expiry"
- and the sign-in the app had before, if any, stays
- and nothing is cloned

[test: POST /api/auth/github/token signs in, and refuses a bad token or host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L139 ]

## S6: changing the data repo @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a data repo is configured, and the sidebar header shows its folder
  name only — parent folders appear in a tooltip, and the full path in the
  Data directory dialog
- When: the user opens Manage folders… (the recent folders menu next to the
  name, or the command palette) and chooses "Open another folder…"
- Then: the Data directory dialog opens pre-filled with the current data repo
- and the header has no separate edit (✎) button: Manage folders is the one
  place to open, add or remove a folder
- and confirming a new value switches the app to that repo, resetting the
  view to home (the old selection belonged to the old repo)
- and a value the server rejects leaves the current repo in place, with the
  error shown as a status note

## S7: clone from GitHub with a personal access token @v1 [proposed]

- Given: the first-run form, on a machine that may have neither git nor the
  GitHub CLI
- When: the user chooses "Clone from GitHub", enters the repository url
  and pastes a personal access token
- Then: the token is checked against GitHub and kept as the app's GitHub
  sign-in (git-operations.md "Personal access token")
- and the repository is cloned into the app's repos folder and opens on
  its README, like a picked folder
- and edits are pushed back with that token, authored by its account
- and the form links to the page where the host makes fine-grained
  tokens, naming the access it needs: the repository, Contents read and
  write

[test: a pasted token for github.com is stored without a host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/personalAccessToken.test.ts#L132 ]

## S8: first run offers a folder or a clone @v1 [proposed]

- Given: no data repo is configured, in the local web app or the desktop
  app
- When: the app starts
- Then: the user picks between "Open a folder on this computer" — one
  path field for a specification folder or a code repository, with the
  recent folders on it and Browse… where there is a native chooser — and
  "Clone from GitHub"
- and the hosted app keeps its single GitHub url field, its token living
  on the account page

## S9: forgetting a data repo @v1 [proposed]

**Created by:** RomanWright

**Updated by:** RomanWright

- Given: a data repo is configured, and the sidebar header shows its folder
  name only — parent folders appear in a tooltip, and the full path in the
  edit dialog

- When: the user clicks the edit (✎) button next to the name
- Then: a dialog opens pre-filled with the current data repo
- A button in this dialog causes the app to stop showing this existing data repo in the dropdown

## S10: migrating the pre-settings.json layout @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a plain `.specdriven` file (the old layout) is in the project root
- When: the app starts
- Then: it is read as the settings file
- and the next save moves it to `./.specdriven/settings.yaml`, preserving
  every setting

## S11: no settings file anywhere @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: neither settings file exists
- When: the app starts
- Then: the user is asked for the data repo

## S12: only the home settings file exists @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: `~/.specdriven/settings.yaml` names a data repo, and the project has
  no settings file
- When: the app starts
- Then: the user is not asked for a data repo
- and settings saved from the Settings page go back to the home settings file

## S13: picking the data directory with the native folder chooser @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

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

[test: macOS and Windows get a chooser out of the box, unknown platforms none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L47 ]
[test: canChooseFolder reflects whether a chooser is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L101 ]
[test: chooseFolder resolves the picked path, null when canceled : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L107 ]
[test: POST /api/choose-folder returns the picked path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L136 ]
[test: POST /api/choose-folder is 501 with no native chooser : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L145 ]

## S14: the data repo is pulled on start @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a data repo is configured, and its remote has commits this machine
  does not have yet
- When: the app starts
- Then: the data repo is pulled before the first screen is read, so the
  feature list shows the remote's latest data without the user pressing Sync
- and the pull does not hold up the rest of startup; a pull that hangs stops
  blocking reads after a short wait, leaving stale data on screen until the
  next reload tick rather than an app that never opens
- and a pull that fails is logged; the app still opens on the data on disk

[test: the data repo is pulled on app start, before the first read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartSync.test.ts#L81 ]

## S15: the first-run clone lands in the repos folder on Windows too @v1 [proposed]

- Given: a Windows machine with neither git nor the GitHub CLI, so the
  built-in engine clones
- When: the user clones from the first-run form
- Then: the repository lands in the app's repos folder, a drive-letter
  path, and not under the app's own install folder
- and a clone that fails still names the url and the reason, with the
  sign-in hint when credentials are the problem

[test: the bundled engine never hands just-git the absolute clone target : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L326 ]
[test: a failed clone on the bundled engine reports the url and git's output : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L340 ]
[test: the bundled engine clones into an absolute target whose parent does not exist : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L354 ]

## S16: the project settings file wins @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: both settings files exist
- When: the app starts
- Then: the project settings file is used
- and saving settings leaves the home settings file untouched

## S17: the server cannot be reached at start @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the web app's server is not running, or takes the page's first
  request and never answers (the desktop app serves itself and cannot hit
  this)
- When: the page loads
- Then: it says the SpecDriven server at that address cannot be reached, with
  how to start it (`bun run dev`) and a Retry button
- and it keeps retrying on its own, opening normally once the server answers
- and it never sits on the loading skeleton indefinitely
- and a server that answers with an error shows that error instead

[test: retries a refused fetch and returns the state once the server answers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootStateRetry.test.ts#L46 ]
[test: reports the server unreachable after every try is refused, without throwing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootStateRetry.test.ts#L53 ]
[test: an error answer is reported with the server's message and not retried : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootStateRetry.test.ts#L60 ]
[test: a server that never answers is reported as stuck after the timeout, not retried : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootStateRetry.test.ts#L69 ]

## S18: the app's own checkout uses the home settings file @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the app runs from its own source checkout — the specdriven-app
  repository or one of its worktrees — with no `SPECDRIVEN_ROOT` pointing
  elsewhere
- When: it reads or saves settings
- Then: it uses `~/.specdriven/settings.yaml`, even when the checkout has a
  `.specdriven/settings.yaml` of its own
- and with no home file yet, the checkout's old settings file is read, and
  the first save moves those settings to the home file
- and with no settings file anywhere, the home one is created
- and agent session records and worktrees live beside the home file too

[test: the app's own checkout reads and writes the home settings file even with a project one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L205 ]
[test: the app's own checkout creates the home settings file when there is none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L217 ]
[test: the app's own checkout reads its old project settings until the first save moves them home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L225 ]
[test: another project's package.json does not count as the app's checkout : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L255 ]

## S19: a new app version upgrades the settings file @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a settings file written by an older version of the app, in an older
  format — the version it carries (`settingsVersion`), or none, which is
  version 0
- When: a new version of the app starts
- Then: it runs, in order, the upgrade steps the file has not had, and
  saves the file in its own format with its own version
- and the file as it was is kept beside it as `settings.yaml.v<old version>.bak`
  (a copy already there is kept: it holds the oldest file)
- and every setting the steps do not change is kept as it was
- and the next start finds nothing to do
- and a file a newer version of the app wrote is read as it is: nothing is
  rewritten and its version is not lowered
- and a file that does not parse is left alone

[test: there is one upgrade step per version, from 1 up to this app's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L67 ]
[test: a file without a version, or with a malformed one, is version 0 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L74 ]
[test: an older file has the steps it has not had and is stamped with this app's version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L80 ]
[test: a file a newer app wrote is left as it is, its version not lowered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L106 ]
[test: readState reads an older file in this app's format, without touching the file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L118 ]
[test: upgrading the file saves it in this app's format and keeps the old one beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L128 ]
[test: a backup already there is kept, as it holds the oldest file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L149 ]
[test: a newer app's file, an unreadable file and no file are left alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L157 ]
[test: starting the app upgrades an older settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsVersion.test.ts#L169 ]

## S20: one field opens a specification folder or a code repository @v1 [proposed]

**Created by:** vydra

- Given: no folder is open, in the local web app or the desktop app
- When: the user names a folder in the first-run screen's one path field —
  typed, or with Browse… — and clicks "Open this folder"
- Then: a folder that holds a `.specdriven` folder and a `specs` folder
  opens as a specification folder
- and a folder without those that holds a well-known build tool's file —
  Maven's pom.xml, npm's package.json, Gradle, Cargo, Go, Python, Make and
  the like — opens as a code repository (code-repo-mode.feature.md)
- and a `specs` symlink, the link a code repository carries to the specs
  it implements, does not count as a `specs` folder
- and a folder that is neither is asked about: "A specification folder",
  which adds `.specdriven` and `specs` to it, or "A code repository", which
  leaves it as it is
- and a folder opened before opens as the kind it was opened as
- and a GitHub url is cloned and opened as a specification folder; a path
  that names no folder is made, on a yes, as a new specification folder
  with both folders; a path that names a file is refused
- and the hosted app and the demo keep their own single field

[test: a folder with .specdriven and specs inside is a specification folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L65 ]
[test: a specification folder that also holds a build file is still a specification folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L70 ]
[test: one of the two folders alone does not make a specification folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L75 ]
[test: a folder with a well-known build tool's file is a code repository, named by the tool : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L80 ]
[test: a code repository's specs link does not make it a specification folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L103 ]
[test: an empty folder with another name, and a specs-… folder with files, cannot be told : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L120 ]
[test: a missing path and a file are reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L125 ]
[test: a repository url is a url, whatever is on disk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L133 ]
[test: a relative path is read against the app's folder, and ~ is the home folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L139 ]
[test: the build files name each tool, exact names before project-named ones : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L151 ]
[test: POST /api/folder-kind answers the kind of the folder at a path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L171 ]
[test: the demo and the hosted app tell no folder's kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L180 ]
[test: a folder made on request starts as a specification folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L199 ]
[test: there is one path field and one button, for a specification folder or a code repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L67 ]
[test: Browse… is there only where there is a native chooser : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L80 ]
[test: while a folder opens, the button waits and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L85 ]
[test: an error is shown under the field : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L90 ]
[test: the screen asks which kind it is, naming the folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L100 ]
[test: each answer hands back its kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L110 ]
[test: with no question, nothing is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L121 ]
[test: each kind of folder has its step : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L182 ]
[test: a malformed answer is no verdict : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L193 ]
[test: a folder opened before as one kind opens as that kind again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L213 ]
[test: the first-run field's chooser is titled for either kind of folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L197 ]

## S21: an empty specs- folder is set up on a yes @v1 [proposed]

**Created by:** vydra

- Given: an empty folder whose name starts with `specs-` — git's own
  `.git` folder does not make it non-empty
- When: the user names it in the first-run screen's folder field
- Then: the app asks whether it is a specification folder
- and on a yes a `.specdriven` folder and a `specs` folder are added
  inside it, and it opens as a specification folder
- and on a no nothing is added, and the first-run screen stays
- and an empty folder with any other name is asked about as a folder of
  neither kind

[test: an empty folder named specs-… is asked about; git's own folder does not make it non-empty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L115 ]
[test: initSpecFolder adds .specdriven and specs, leaving what is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L160 ]
[test: PUT /api/state initSpecFolder sets up the picked folder and opens it as specifications : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L189 ]
[test: initSpecFolder makes nothing at a path that names no folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderKind.test.ts#L206 ]

## S22: the folder field lists the recent folders @v1 [proposed]

**Created by:** vydra

- Given: the first-run screen, with folders opened before — after
  "Log out", say
- When: the user clicks the ▾ on the folder field
- Then: a list drops down with the ten most recent folders, specification
  folders and code repositories together, newest first, each with the icon
  of its kind, its name and its path
- and picking one fills the field with its path, leaving the user to
  confirm with "Open this folder"
- and it opens as the kind the list has it as
- and with no folder opened before there is no ▾

[test: the field carries a Recent folders button, closed at first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L127 ]
[test: with no folder opened before, there is no drop-down : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L133 ]
[test: opening it lists the folders newest first, each with its kind and path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L139 ]
[test: it lists at most ten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L156 ]
[test: picking a folder fills the field, hands back the entry and closes the list — the button still opens it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L164 ]

## S23: an existing settings.json is converted to settings.yaml @v1 [proposed]

- Given: a settings file an older version of the app left as
  `settings.json` — in the project, or in the home folder
- When: a new version of the app starts
- Then: the same settings are written as `settings.yaml` in the same
  folder, and the app reads and saves that file from then on
- and `settings.json` is kept beside it as `settings.json.bak`
- and nothing but the format changes in that step: an older file is then
  upgraded as any other (its copy is `settings.yaml.v<old version>.bak`)
- and every value comes back as it was saved — text that looks like a
  number, a switch or nothing, an empty text, a folder path or a url used
  as a key
- and the next start finds nothing to do
- and until it is converted a `settings.json` is read as it is, where the
  settings file is looked up; a save converts one the start did not
- and a `settings.json` beside a `settings.yaml` is not read, and is left
  as it is
- and a `settings.json` that does not parse is not converted: it reads as
  no settings, and the next save sets it aside as `settings.json.bak`
- and the hosted app reads a worker's `settings.json` until that worker
  converts it, and never writes a settings file over it

[test: settings are saved as YAML, in settings.yaml : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L75 ]
[test: every value comes back from settings.yaml as it was saved : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L85 ]
[test: a settings.json from before YAML is read as it is, until it is converted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L128 ]
[test: converting writes the same settings as settings.yaml and keeps settings.json as settings.json.bak : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L139 ]
[test: an older settings.json is converted, then upgraded to this app's format : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L155 ]
[test: starting the app converts a settings.json : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L175 ]
[test: settings.yaml wins over a settings.json beside it, which is left alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L183 ]
[test: a save converts a settings.json the start did not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L196 ]
[test: a settings.json that does not parse is not converted; the next save sets it aside : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L205 ]
[test: the home settings.json is converted where it is, and saves stay in the home folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L218 ]
[test: a project settings.json still wins over the home settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L231 ]
[test: the desktop app converts the home settings.json, not one under its app data : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L243 ]
[test: the settings file view shows a home settings.json that has not been converted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsYaml.test.ts#L256 ]
[test: a worker's settings.json from before YAML is read, and not seeded over : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L164 ]
