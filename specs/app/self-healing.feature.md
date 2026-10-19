# Self-healing: Fix this app

An app installed with `curl -fsSL …/install.sh | sh` is the source it was
built from, under `~/.specdriven/app/current`, plus the desktop app built
there and copied or started. When something in it breaks, the user can
describe the bug inside the app — words and screenshots — and ask a coding
agent to fix the app's own source. The agent's change is a diff the user
reviews; a rebuild turns it into bundles under the installed source, and a
restart boots them through a small shim that fronts the shipped app. A fix
that cannot start is stepped around; a fix the user does not want is put
back.

"Fix this app" is a built-in plugin, on by default: the Fix this app tab,
`g x`, the palette's "Fix this app…", and the `/api/self-fix` routes, all
gone when it is switched off. Nothing here runs in the demo or a hosted
worker. Like every plugin it is listed in a data folder's profile
(specs/app/plugins.feature.md), so an install that has never opened a
folder has no tab yet: a fresh install shows Specs alone.

## A crash after a rebuild falls back to the shipped build @v1 [proposed]

- Given: the desktop app booted a rebuilt fix
- When: that bundle fails to show a window on two launches in a row
- Then: the third launch boots the bundle shipped inside the app instead,
  and the Fix this app pane says so, with Revert and the agent on offer
- and a bundle that cannot even be loaded falls back in the same launch
- and once a rebuilt bundle has shown its window, its count starts over; a
  rebuild starts the count over too

[test: a crash after a rebuild falls back to the shipped build after two launches without a window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L45 ]
[test: a rebuilt fix boots, counted as an attempt until its window shows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L28 ]
[test: a rebuild starts the count over, and drops the last load error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L58 ]
[test: a corrupt state file counts as none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L81 ]
[test: the banner names the fix running, or why the shipped build runs instead : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L93 ]
[test: a fall-back banner is marked as a warning : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L104 ]

## A fix is reverted to the installed source @v1 [proposed]

- Given: a fix that was started, whether or not it was rebuilt
- When: the user presses Revert and confirms
- Then: the source goes back to the baseline — the agent's edits and new
  files are gone, what the install built (node_modules, dist, release)
  stays — the agent's session ends, the rebuilt bundle's manifest and the
  shim's memory of it are removed, and the app restarts into the shipped
  build where it can restart itself
- and a developer's checkout that had uncommitted work when the fix began
  is not reset: the revert is refused with the reason

[test: the tree goes back to the baseline, the agent's session ends, and the patched bundle is forgotten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L425 ]
[test: a developer's checkout with uncommitted work is not reset behind their back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L455 ]

## A fix is started from a bug description and screenshots @v1 [proposed]

- Given: the Fix this app tab, or "Fix this app…" in the command palette
- When: the user describes the bug, pastes or drops screenshots, and presses
  "Ask the agent to fix it"
- Then: a coding agent session opens in the agent pane, working in the
  installed source, with the description quoted as data, each screenshot's
  path to read, the newest lines of the app's log as evidence, and the
  rules: find the root cause, change as little as possible under src/, add
  a test and run it, leave the install scripts and dependencies alone, do
  not run git, end with a summary
- and the screenshots are stored beside the fix under the SpecDriven home,
  never in the data repo, whose images are committed and pushed
- and a fix starts with no data folder open, since the bug may be what
  keeps one from opening
- and an empty description, or a screenshot after the start, is refused

[test: the screenshot lands beside the fix, never in the data repo, and the agent is pointed at it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L252 ]
[test: the prompt carries the user's words as data, the screenshots as paths, the log as evidence, and the rules : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L212 ]
[test: a fix starts with no data repo open, the source standing in for it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L274 ]
[test: an empty description, a wrong mode, or an unknown fix are refused in words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L281 ]
[test: titles and labels come from the first line of the user's words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L139 ]

## A rebuilt fix restarts the desktop app with the fix enabled @v1 [proposed]

- Given: a fix whose agent has changed something
- When: the user presses Rebuild, then Restart, confirming each
- Then: the rebuild runs `build:desktop --bundle-only --fix <id>` in the
  installed source — the renderer, the main process and the boot shim, no
  installer — leaving dist/patch.json naming the fix and the app version;
  the change is committed in the source's own history and kept as a patch
  beside the fix
- and the desktop app quits and starts again: its entry is a shim that
  finds the rebuilt bundle through the install's `current` link and boots
  it in place of the shipped one, so the fix runs whether the app sits in
  /Applications, in an AppImage, or anywhere else
- and the pane then says which fix is running
- and a failed rebuild is recorded and changes nothing that runs; a restart
  before a rebuild is refused
- and the web server (`bun run start`) cannot restart itself: Restart
  reads Apply there, and says to run `bun run start` again

[test: the rebuild runs the bundle-only desktop build in the source, commits the change and keeps the patch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L368 ]
[test: a failed build is recorded and changes nothing that runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L387 ]
[test: the desktop relaunches itself once the answer is out; the web server says what to run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L399 ]
[test: --bundle-only skips the installers, and --fix names the fix the bundle is for : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L64 ]
[test: the shim tells the server which bundle runs, and the server reads it back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L94 ]
[test: rebuild needs a change, restart a build, revert a start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L114 ]
[test: the status line says where the fix stands, with the agent's state and the last error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L123 ]

## Fix this app is refused where it cannot work @v1 [proposed]

- Given: the Fix this app tab
- When: code generation is off in Settings, the `claude` command is not on
  PATH, git is not on PATH, the source is not found, or its version is not
  the one running
- Then: the pane says which, with the remedy, and a fix cannot start; the
  routes refuse it in the same words
- and without bun the fix can be made but not rebuilt; a desktop build from
  before self-fixing, or the web server, is told that a rebuilt fix needs a
  reinstall or a restart by hand — neither blocks the fix
- and the demo and a hosted worker refuse every route, and offer no tab

[test: code generation off, no claude, no source : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L294 ]
[test: the demo and a hosted worker refuse every route; a switched-off plugin has none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L315 ]
[test: every missing precondition is a hint; the ones that block come with the remedy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L57 ]
[test: the hints render as a list, with a Settings button where Settings holds the remedy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L83 ]

## The agent works in the installed source behind a baseline commit @v1 [proposed]

- Given: the installed source, unpacked by install.sh with no history
- When: the first fix starts
- Then: the source gets a git repository of its own — one commit of the
  tree as installed, tagged `baseline`, with node_modules, dist, release
  and the kept zip excluded — so every edit the agent makes is a diff and a
  revert is a reset; nothing is pushed, the repository has no remote
- and a developer's checkout keeps its history: the baseline is HEAD, and
  the fix remembers whether the tree was clean

[test: an installed tree gets a repository of its own, tagged, with the build's output kept out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L188 ]
[test: a developer's checkout keeps its history, the baseline is HEAD, and only a clean tree may be reverted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L199 ]

## The fix is reviewable as a diff before it is applied @v1 [proposed]

- Given: a fix whose agent is working, or done
- When: the user opens the fix's page
- Then: it lists the files changed so far with git's summary, and "Review
  changes" shows the unified diff against the baseline, new files included
- and "Open session" shows the agent's transcript in the agent pane, where
  the user answers its questions or stops it as with any session

[test: what the agent changed shows as files, a summary and a unified diff : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L331 ]
[test: the change list names the files, or says nothing changed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFixPane.test.tsx#L133 ]

## The installed source must match the running app @v1 [proposed]

- Given: the app looking for its own source
- When: it starts a fix
- Then: it takes SPECDRIVEN_APP_SOURCE when set, else the install's
  `app/current` link under the SpecDriven home, else — a developer's `bun
  run start` — the working directory when it is this package
- and the source's package.json version must be the version running: a fix
  made to other code would not be the fix that runs, so another version is
  reported and the fix refused until the installer is run again

[test: SPECDRIVEN_APP_SOURCE names it, else app/current under the SpecDriven home, else a developer's checkout : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L160 ]
[test: a source of another version is found but not matched, and a fix refuses to start on it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/selfFix.test.ts#L177 ]

## The patched bundle is ignored after an upgrade @v1 [proposed]

- Given: a rebuilt fix under one install, then the installer run again for
  a newer version
- When: the newer app starts
- Then: it boots its own shipped bundle: the new install's `dist` carries
  no patch manifest (an ordinary build removes a stale one), and a manifest
  naming another app version is ignored in any case
- and the fix's patch stays beside its record for whoever wants to carry it
  forward

[test: the patched bundle is ignored after an upgrade, since its version is not the one running : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L40 ]
[test: without a manifest the shipped bundle boots and nothing is remembered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L20 ]
[test: the manifest names the app version and the fix; a build without a fix removes a stale one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L74 ]
[test: a manifest needs all three fields; anything else is no manifest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bootPolicy.test.ts#L72 ]
