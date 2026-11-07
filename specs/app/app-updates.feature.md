# App updates

**Assigned:** David

SpecDriven can check for a newer release and, as the desktop app, install it
and restart into it: **Check for Updates…** — clicking the version beside
the user name at the lower left, or the app menu on macOS (Help on Windows
and Linux) — opens the Updates page.

## How an update installs

A release is a source zip (specs/app/installation.feature.md), built on the
machine that runs it, so there is no binary to download and swap in. Instead
the app runs the very installer that shipped in its own source — the
`install.sh` (`install.ps1` on Windows) under `~/.specdriven/app/current` —
for the newer version: the same signature and checksum checks, the same
atomic unpack to `~/.specdriven/app/specdriven-<version>`, the same move of
`current` and clearing of older versions, the same `bun install` and
`bun run build:desktop --local`. The installer is told the version
(`SPECDRIVEN_VERSION`), that Bun is there (`SPECDRIVEN_INSTALL_BUN=0`, since
nobody is at a terminal to answer) and not to start what it built or open
the `.dmg` (`SPECDRIVEN_HANDOVER=0`): the restart is the app's own. Its
output shows on the page as it runs; a build fetches Electron and takes a
few minutes, and the app keeps working meanwhile.

The newest release is found the way the installer finds it: the releases
page's `latest` redirects to `tag/v<version>`, which needs no token.
`SPECDRIVEN_RELEASES_URL` points both at a release server of your own.

Nothing changes what runs until the user restarts. The version running stays
under `~/.specdriven/app` as the install's one copy to go back to.

## Restart into the new version

Where the app runs from differs by platform, so does the restart:

- On Windows and Linux the build under the new tree is the app
  (`release\win-unpacked\SpecDriven.exe`, the `.AppImage`), and the Start
  menu shortcut already reaches it through `current`: the app quits and
  starts that executable.
- On macOS the installer's `.dmg` is for dragging the app into
  Applications, so the copy there is what has to change: the running bundle
  is renamed aside as `SpecDriven.app.previous`, the new one is copied to the
  same path (`ditto`, which keeps its signature), and the app quits and starts
  from that path again — so the Dock and Launch Services keep pointing at it.
  The bundle renamed aside is removed once the new app has shown a window, so
  a build that cannot start leaves the previous app beside itself to rename
  back by hand. A Mac that runs the app straight from the install's
  `release/` directory simply starts the new one from there.
- Where the shell cannot do it — Applications is not writable, a development
  run with no bundle — Restart says so and where the new app is.

The in-app update runs the installer that was installed, not the one on
`main`: after a release-key rotation it refuses the new release, as it
should, and the installer one-liner from `main` is the way forward once.

## S1: Check for Updates finds the newest release @v1 [proposed]

- Given: the desktop app, running some version
- When: the user chooses Check for Updates… — by clicking the version at the
  lower left, or from the app menu (Help on Windows and Linux)
- Then: the Updates page opens and asks the releases page's `latest`
  redirect which release is newest, and says either that this version is up
  to date, or which newer version is available, with a link to its release
  notes
- and a releases page that cannot be reached, or does not name a release, is
  reported in so many words

[test: the releases page's `latest` redirect names it, as it does for the installer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L140 ]
[test: the version running, or a newer one, is up to date : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L157 ]
[test: a page that does not redirect to a release tag, or cannot be reached, is an error in so many words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L167 ]
[test: the releases page is SPECDRIVEN_RELEASES_URL, as install.sh reads it, else the app's own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L179 ]
[test: a newer version comes after the running one; a malformed one never does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/version.test.ts#L24 ]
[test: on macOS it sits in the app menu, under About; elsewhere under Help : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appMenu.test.ts#L12 ]
[test: the Debug menu is there only when given — a development run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appMenu.test.ts#L28 ]
[test: X.Y.Z parses to its three numbers; anything else is no version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/version.test.ts#L8 ]
[test: versions compare numerically, not as text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/version.test.ts#L16 ]
[test: each stage has its headline and its controls : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L31 ]
[test: a newer release offers Install & restart and says what installing does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L80 ]

## S2: An update installs through the installer, without hand-over @v1 [proposed]

- Given: a newer release found
- When: the user presses Install & restart
- Then: the installer in the running app's source runs for that version,
  told that Bun is there and not to start or open anything, with Bun's
  directory on its PATH; its output shows on the page as it runs
- and once it has finished, the new version's tree is beside the running
  one's with `current` on it and the app built in it, and the page offers
  Restart now
- and a second install while one runs, a version that is not newer, or one
  that is not a release version is refused

[test: the installer in the running app's source does it, told the version and not to start anything : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L222 ]
[test: on Windows it is install.ps1, run as a file by PowerShell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L246 ]
[test: a version that is not newer, not a version, or already installing is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L254 ]
[test: the installer's output is kept, newest last, to the last thousand lines : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L288 ]
[test: the real runner streams a command's output line by line and reports a command that is not there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L296 ]
[test: GET says where things stand; check, install and restart drive them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L342 ]
[test: while installing, the installer's output is on the page and nothing is offered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L101 ]
[test: the installer's last lines are what the page shows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L69 ]
[test: SPECDRIVEN_HANDOVER=0 builds the macOS app and opens nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L872 ]
[test: SPECDRIVEN_HANDOVER=0 makes the AppImage runnable but does not start it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L898 ]
[test: SPECDRIVEN_HANDOVER=0 builds the app and adds the shortcut, but does not start it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L513 ]

## S3: On macOS the new app takes the running one's place @v1 [proposed]

- Given: a newer version installed and built, and the app running from a
  bundle outside `~/.specdriven/app` — Applications, say
- When: the user presses Restart now
- Then: the running bundle is renamed aside as `SpecDriven.app.previous`, the
  new one copied to its path, and the app quits and starts from that path
- and a copy that fails puts the running bundle back and reports it, with
  nothing half done
- and the bundle renamed aside is removed once the new app has shown a
  window — not before, so a build that cannot start leaves it to go back to
- and a bundle running from the install's own `release/` directory is not
  copied anywhere: the new one is started from there
- and once a newer release is found, the Updates page on macOS says "You may
  need to request Admin privileges on Mac/OSX", since replacing the app in
  Applications can ask for an administrator

[test: on macOS, a newer release warns that Admin privileges may be needed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L90 ]
[test: a Mac app run from the install's own release directory is not copied anywhere, the new one just starts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L77 ]
[test: a Mac app in Applications is replaced where it is, and started from there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L57 ]
[test: the running bundle steps aside as .previous, a stale .previous first going, and the new one is copied to its path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L147 ]
[test: a copy that fails puts the running bundle back and reports the failure : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L161 ]
[test: the bundle of an executable is its .app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L135 ]

## S4: Windows and Linux restart into the new tree @v1 [proposed]

- Given: a newer version installed and built
- When: the user presses Restart now
- Then: the app quits and starts the new tree's `SpecDriven.exe` or
  `.AppImage`; the tree that was running stays as the install's copy to go
  back to

[test: Windows and Linux start the new build where the installer left it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L110 ]
[test: the built app is the .app, the AppImage or the unpacked exe the local build leaves : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L306 ]
[test: the runtime's activator gets the new source tree; without one the new app's path is handed back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L322 ]

## S5: Updating is refused where it cannot work @v1 [proposed]

- Given: the web server (`bun run start`), a machine without Bun or a shell,
  an app whose installed source is missing or is not the version running,
  or a desktop build with nothing registered to restart it
- When: the Updates page opens, or an install is asked for
- Then: the page says why an update cannot install here and offers the
  installer one-liner for the platform instead, and the install route
  refuses with the same reason
- and the demo and a hosted copy refuse every update route: neither changes
  the machine it serves from

[test: the desktop app with Bun, its source and a shell can; each missing piece is a reason : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L186 ]
[test: a check that finds nothing is a 502 with the reason; an install that cannot run is a 409 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L375 ]
[test: the demo and a hosted copy refuse every route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L397 ]
[test: where the app cannot update itself, the reasons and the installer one-liner take Install's place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L116 ]

## S6: A failed install changes nothing that runs @v1 [proposed]

- Given: an install under way
- When: the installer fails, or finishes without the new tree, without
  `current` on it, or without a built app in it
- Then: the page says so, with the installer's last lines, and offers to try
  again; the running app keeps running and Restart is refused
- and the tree running stays in place; `current` may already sit on the new,
  unbuilt tree, as the installer leaves it (specs/app/installation.feature.md
  "a build that fails leaves the source in place")

[test: an installer that fails, or finishes without what it promised, leaves a failed install and changes nothing that runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L265 ]
[test: a failed install shows the reason, the output and the way to try again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L108 ]

## S7: Where the app cannot replace itself it says where the new one is @v1 [proposed]

- Given: a newer version installed and built, but a bundle the shell cannot
  replace — Applications is not writable, or this is a development run with
  no bundle — or a new tree with no built app
- When: the user presses Restart now
- Then: nothing is renamed or started; the page says why and where the new
  app is, for the user to start or move by hand

[test: an Applications folder that is not writable, or no bundle at all, leaves the swap to the user and says where the new app is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L90 ]
[test: a new version with no built app cannot be started : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateActivation.test.ts#L125 ]
[test: a restart the shell could not do says so, with the new app's path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updatesPage.test.tsx#L129 ]

## S8: The desktop app checks once a day and marks the version chip @v1 [proposed]

- Given: the desktop app, running some version
- When: it starts, and once a day after that while it runs — not the web
  server, the demo or a hosted copy, which cannot update themselves
- Then: it asks the releases page which release is newest, without holding
  up the start, and a newer one puts a dot on the version at the lower left,
  saying on hover which version is available; clicking the version opens the
  Updates page, as it always does on the desktop app, and the user menu has
  no Check for Updates… of its own
- and a check that fails — no network, a page that names no release — is a
  debug line in the app's log and nothing on screen; the next look is a day
  on, and the manual check is there meanwhile

[test: the desktop app checks once a day, counted from the last check, and /api/state names the newer version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L424 ]
[test: the version running, or a newer one, leaves the chip unmarked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L455 ]
[test: the web server, the demo and a hosted copy never check by themselves : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L464 ]
[test: a check that fails is swallowed, counts as the day's check, and leaves the last answer alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appUpdate.test.ts#L480 ]
[test: a newer release puts a dot on the version, saying which version is out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateHint.test.tsx#L9 ]
[test: no newer release known, no dot — the release-notes dot is its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateHint.test.tsx#L17 ]
[test: on the desktop app the version is a button to the Updates page, carrying the same dot and hint : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateHint.test.tsx#L30 ]
[test: elsewhere the version is plain text, no way to the Updates page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/updateHint.test.tsx#L51 ]
