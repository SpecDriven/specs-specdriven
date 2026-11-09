# App start

**Created by:** vydra

**Updated by:** vydra

When the application starts, if the settings file does not
contain the name of the last data repo, the user is asked the name of the data repo.

The first-run screen offers two ways in: open a folder on this computer,
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

Local state lives in `.specdriven/settings.json`, looked up in this order:

- `./.specdriven/settings.json`, relative to the directory the app runs from
- `~/.specdriven/settings.json`, so one settings file can serve every checkout

`SPECDRIVEN_HOME` overrides that home directory — the same variable install.sh
uses for the install prefix. Settings are saved back to whichever of the two
the app read; when neither exists, the project one is created.

Two runs use the home file only: the desktop app (desktop-app.feature.md
"Data locations"), and a run from the app's own source checkout (the
specdriven-app repository or a worktree of it), so every checkout, worktree
and the desktop app share one settings file.

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
  edit dialog
- When: the user clicks the edit (✎) button next to the name
- Then: a dialog opens pre-filled with the current data repo
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
- Then: the user picks between "Open a folder on this computer" — a path,
  with Browse… where there is a native chooser — and "Clone from GitHub"
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
- and the next save moves it to `./.specdriven/settings.json`, preserving
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

- Given: `~/.specdriven/settings.json` names a data repo, and the project has
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

[test: the bundled engine never hands just-git the absolute clone target : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L310 ]
[test: a failed clone on the bundled engine reports the url and git's output : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L324 ]
[test: the bundled engine clones into an absolute target whose parent does not exist : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L338 ]

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
- Then: it uses `~/.specdriven/settings.json`, even when the checkout has a
  `.specdriven/settings.json` of its own
- and with no home file yet, the checkout's old settings file is read, and
  the first save moves those settings to the home file
- and with no settings file anywhere, the home one is created
- and agent session records and worktrees live beside the home file too

[test: the app's own checkout reads and writes the home settings file even with a project one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L204 ]
[test: the app's own checkout creates the home settings file when there is none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L216 ]
[test: the app's own checkout reads its old project settings until the first save moves them home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L224 ]
[test: another project's package.json does not count as the app's checkout : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L254 ]
