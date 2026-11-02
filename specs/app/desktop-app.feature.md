# Desktop app

SpecDriven also runs as a native desktop app (Electron), installed from a
macOS dmg or a Linux AppImage or deb, or on Windows built and started in place
by `install.ps1` (installation.feature.md). It is the same application as the web
version: one React frontend, one API, one git-backed data store. The desktop
shell serves the frontend and the API over an internal `app://` protocol — no
localhost port is opened, so nothing else on the machine can reach the API.

The web version keeps working unchanged (`bun run dev`, the compiled binary).
Server code is written against Node APIs only, so the same modules run under
Bun on the web and under Electron's runtime on the desktop.

## Packaging

One command builds the installable artifacts, for the platform the build runs
on unless told otherwise (`bun run build:desktop [--mac | --linux | --win | --all]`):

- macOS: a dmg and a zip.
- Linux: an AppImage and a deb, for x64 and arm64. Both cross-build from
  macOS, so a release can be cut from one machine.
- Windows: an NSIS installer, for x64 and arm64. `--local` packages no
  installer, only the app unpacked in `release\win-unpacked`, which is what
  `install.ps1` starts. `--host-arch` packages the installer for this
  machine's architecture only, `SpecDriven-<version>-<arch>-setup.exe`:
  on request, a release gets one for x64 and one for arm64, each built and
  smoke-tested by the Windows release workflow on a Windows runner of that
  architecture, since the native module the app ships is installed for the
  build machine alone, and published as a release of their own,
  `windows-v<version>`. They are not code-signed yet.

Every artifact carries the application icon: the sidebar's SD mark on its
accent tile, drawn once as `build/icon.svg`. `bun run build:icons` rasterizes
it with Electron's own Chromium into the `.icns`, `.ico` and Linux PNGs under
`build/` that electron-builder stamps, so no build machine renders it and no
image library joins the dependencies; the renderings are committed and
refreshed when the artwork changes. macOS and Linux get the tile on the
transparent margin their desktops expect, Windows the full-bleed one. A copy
ships beside the bundles for the Linux window's icon and for the dock of a
development run on macOS, and the web app's favicon is the same artwork.

The macOS artifacts need macOS tooling, so `--mac` off a Linux machine stops
with that as the reason rather than producing a broken build. The Windows ones
are built on Windows — off it, electron-builder needs Wine to stamp the
executable — so `--win` elsewhere stops the same way, and `--all` means the
release one Mac can cut: macOS and Linux.

The main process is bundled before packaging and no native addon is compiled
for it, so packaging never rebuilds node_modules (`npmRebuild: false`).
Without that, a build machine is asked to compile optional native accelerators
pulled in transitively — `cbor-extract`, say — against Electron's C++20
headers, and a host with an older toolchain fails a build it did not need.

The bundle is self-contained. Bun's bundler bakes a dependency's build-time
`__dirname` in as an absolute path, so the two dependencies that read files by
it — Automerge, whose engine is a wasm file, and cbor-extract, whose optional
native string extractor is a prebuilt addon — are bundled to look beside the
bundle instead, and the build copies those files there: `automerge_wasm_bg.wasm`
and `cbor-extract/prebuilds/<platform>-<arch>/` for every platform package
installed on the build machine (its own, normally; elsewhere cbor-x takes its
JavaScript path). A bundle that still carries a baked path fails the build.
Without this, a cross-built Linux artifact or a copied macOS app died at launch
looking for the wasm in the build machine's `node_modules`.

What the package carries is those bundles — `dist/electron`, `dist/renderer`,
the icon and a self-fix's `dist/patch.json`, named one by one so neither the
web build nor a crash test's output beside them rides along — and nothing of
the dependency tree they were built from: no `node_modules` (once 300 MB of
never-loaded code inside `app.asar`), no source maps, and only Electron's
English locale pack, since the app's own UI is English and the others merely
translate Chromium's context menus. The macOS dmg is LZFSE-compressed, which
downloads smaller than the zlib default and unpacks faster on a modern macOS.

A Linux artifact has to start on a machine that has never seen Electron:

- The deb names every shared library Electron loads. ALSA is the one the
  packaging defaults miss, and it is spelled `libasound2t64 | libasound2` —
  the t64 name first, because on releases that renamed it, plain
  `libasound2` resolves to an OSS compatibility stub that is missing symbols
  Electron calls.
- The AppImage carries the static runtime, so it needs neither FUSE nor a
  dynamically linked zlib on the host.

## Data locations

The desktop app has no meaningful working directory, so it anchors itself in
the OS application-data directory (`~/Library/Application Support/SpecDriven`
on macOS, `~/.config/SpecDriven` on Linux, `%APPDATA%\SpecDriven` on Windows)
via `SPECDRIVEN_ROOT`:

- Settings follow the usual lookup (app-start.feature.md) with the
  app-data directory as the "project root": an existing
  `~/.specdriven/settings.json` is picked up and written back — shared with
  the web version, so both see the same data repo. With no settings file
  anywhere, the desktop app keeps its own under the app-data directory.
- GitHub data repos are cloned under the app-data directory
  (`.../SpecDriven/repos/<name>`) via `SPECDRIVEN_REPOS_DIR`, instead of the
  web version's sibling-of-the-checkout location.

## Window

- One main window; launching the app a second time focuses the existing
  window instead of opening another.
- The window opens at 1280×860, centered, and can never be made smaller
  than 1280×700 — below that the layout starts hiding buttons and
  scrollbars.
- The window remembers its size, position, and maximized state across
  restarts (`window-state.json` in the app-data directory). Remembered
  geometry is clamped to the minimums and to the displays actually
  present; a missing or unreadable file simply means the centered default.
- On macOS the window uses the hidden-inset title bar: the traffic lights sit
  inside the sidebar header, and the drag regions are the sidebar header, the
  editor pane's header row — the trail, the page's title and its actions —
  and thin strips along the window's edges, so the window moves from the top
  of either pane alike. Elsewhere the window wears the system title bar and
  both headers are ordinary headers.
- A drag region swallows the clicks of everything inside it, so every control
  the headers hold — in the sidebar the folder name and its recent-folders
  menu, ✎, Sync, and the icons at the right that open the Pomodoro bar, the
  coding agent pane and the collapsed sidebar; in the editor pane the crumbs,
  the feature's name that renames on click and its input, the icon buttons
  and the More menu — stays clickable rather than dragging the window. The
  rule covers each header's controls as a class, so an icon added later is
  clickable without anyone remembering this.
- Deep links (navigation.feature.md) keep working inside the app: the
  internal protocol serves the app shell for every `/features/...` url.
- The app registers the `specdriven://` scheme with the OS, so a link shared
  by chat or email launches it, or focuses the running window, on that spec
  (navigation.feature.md "links from outside the app"). A second launch
  carrying a link still focuses the one window rather than opening another.
  The installed bundle declares the scheme to the OS — in `Info.plist` on
  macOS, in the desktop entry's `MimeType` on Linux.

## External links

- A link that leaves the app — a scenario's automated-test link on GitHub,
  any `http(s)` url — opens in the user's default browser, never inside the
  app window.

## Dialogs

The desktop runtime has no native `prompt()`, so every prompt and
confirmation is an in-app dialog (ui.feature.md "Dialogs") — the same
dialogs the web version shows.

The one native dialog is the data-directory folder chooser
(app-start.feature.md "picking the data directory with the native folder
chooser"): the desktop app parents the OS picker to its window, replacing
the osascript chooser the local web server falls back to on macOS.

## Git

Git operations shell out to the machine's binaries where it has them
(git-operations.feature.md): the GitHub CLI is used for GitHub
credentials and clones wherever it is installed, plain git where only that
is, and the built-in engine on a machine with neither — so the desktop app
works on a fresh machine with nothing installed. A packaged macOS app
launches with a minimal PATH, so the usual Homebrew locations are appended
— that is where both binaries usually live.

The GitHub sign-in token is stored OS-encrypted (Electron safeStorage,
keychain-backed on macOS); when the OS offers no key store it falls back
to the user-only token file next to the home settings file.

## S1: a build that crashes at launch fails the crash test @v1 [published]

[negative]

- Given: a fresh desktop build
- When: `bun run test:desktop` launches it three times, holding each window
  open before asking it to quit
- Then: the run passes only when every launch rendered the window, survived
  the hold and quit cleanly
- and a launch that dies early, never renders, hangs on quit, records an
  uncaught main-process exception or prints a crash signature fails the run,
  naming what went wrong

[test: a launch that rendered, held and quit with 0 passes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L43 ]
[test: dying before being asked to quit fails, naming how it died : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L47 ]
[test: a window that never renders fails even when the quit is clean : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L84 ]
[test: a quit that hangs, exits non-zero, or leaves a dead window fails : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L89 ]
[test: an exception the main process's listener recorded fails the run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L98 ]
[test: fatal signatures fail the run grouped by kind; notes do not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L117 ]
[test: the output scan finds crashes and ignores a headless box's usual noise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L129 ]

## S2: a Mac builds the Linux app too @v1 [proposed]

- Given: a Mac
- When: the user runs `bun run build:desktop --linux`, or `--all`
- Then: the Linux AppImage and deb are built for x64 and arm64 — with
  `--all`, alongside the macOS dmg and zip
- and with no flag the build packages only for the machine it runs on

[test: without flags the build targets the machine it runs on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L22 ]
[test: a Mac cross-builds the Linux app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L28 ]
[test: a release build packages every target and architecture in package.json : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L51 ]
[test: packaging never rebuilds native modules for the build machine : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L47 ]

## S3: a maximized session comes back maximized @v1 [published]

- Given: the user quit the app with the window maximized
- When: they launch the app again
- Then: the window opens maximized, and un-maximizing returns it to its
  previous size

[test: maximized survives fitting : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L111 ]
[test: save then load round-trips : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L120 ]

## S4: building the macOS app off a Mac stops with the reason @v1 [proposed]

- Given: a Linux machine
- When: the user runs `bun run build:desktop --mac` or `--all`
- Then: the build stops before packaging, saying the macOS app can only be
  built on macOS
- and a mistyped flag or an unsupported host stops the same way, naming the
  problem

[test: the macOS app is only built on macOS : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L33 ]
[test: an unsupported host or a mistyped flag stops before packaging : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L46 ]

## S5: desktop and web share settings @v1 [proposed]

- Given: the web version has recorded a data repo in `~/.specdriven/settings.json`
- When: the desktop app starts
- Then: it opens that data repo without asking

[test: the home settings file is used when the project has none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L58 ]
[test: settings are saved back to the home settings file when only it exists : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L73 ]

## S6: GitHub data repos are cloned under the app-data directory @v1 [proposed]

- Given: the desktop app
- When: the user opens a GitHub data repo that is not on the machine yet
- Then: it is cloned to `.../SpecDriven/repos/<name>` in the app-data
  directory, not next to the app

[test: SPECDRIVEN_ROOT re-bases the app root : https://github.com/SpecDriven/specdriven-app/blob/main/tests/paths.test.ts#L26 ]
[test: SPECDRIVEN_REPOS_DIR gathers clones under the app-data directory : https://github.com/SpecDriven/specdriven-app/blob/main/tests/paths.test.ts#L41 ]

## S7: reloading inside the app keeps the spec on screen @v1 [proposed]

- Given: the desktop app is showing a feature at `/features/<path>`
- When: the window reloads
- Then: the internal protocol serves the app shell for that url, and the
  same feature is back on screen

[test: the root path is the shell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L27 ]
[test: deep links resolve to paths that won't exist, so the caller serves the shell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L36 ]

## S8: test link opens in the browser @v1 [published]

- Given: a scenario with a link to an automated test on GitHub 
AND  open links in IntelliJ is Off
<img src="/api/images/image-1486276d.png" alt="image" width="643">
- When: the user clicks the test link in the desktop app
- Then: the link opens in the default browser and the app window stays where
  it was

[test: a test link on GitHub opens in the browser, not in a new window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L23 ]
[test: a click that would navigate the window away goes to the browser instead : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L28 ]
[test: the app's own pages stay in the window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L36 ]
[test: schemes that are not web pages go nowhere : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L53 ]

## S9: the GitHub sign-in survives a restart without an OS key store @v1 [proposed]

- Given: a machine whose OS offers no key store, and a user signed in to
  GitHub
- When: they quit and relaunch the app
- Then: they are still signed in — the token was kept in the user-only token
  file next to the home settings file

[test: token round-trips through the file backend across backend resets : https://github.com/SpecDriven/specdriven-app/blob/main/tests/auth.test.ts#L249 ]

## S10: the header's icons work inside the drag region @v1 [proposed]

- Given: the macOS desktop app, whose sidebar header is the window drag region
- When: the user clicks the timer, the Bot, or the sidebar collapse icon at
  the header's right
- Then: each opens what it names — the Pomodoro bar, the coding agent pane,
  the collapsed sidebar — instead of starting to drag the window

[test: every control in the header stays clickable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L156 ]
[test: the header's icon buttons stay clickable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L169 ]

## S11: the internal protocol serves only the app's bundled files @v1 [proposed]

- Given: the desktop app's internal `app://` origin
- When: a url asks for a path outside the bundled frontend — `..`, an
  encoded `..`, a smuggled separator
- Then: no file outside the bundle is read; the app shell is served instead

[test: bundle files resolve inside the renderer directory : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L31 ]
[test: escapes and smuggled separators are rejected : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L42 ]

## S12: the packaged Linux app opens on a clean machine @v1 [proposed]

- Given: a Linux system with no Electron and no developer tooling installed
- When: the user installs the deb, or marks the AppImage executable and runs it
- Then: the app window opens on the internal `app://` origin, the same as on
  macOS

[test: the deb names the shared libraries Electron loads : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L19 ]
[test: ALSA is required by its t64 name first, then the old one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L26 ]
[test: the AppImage carries the static runtime, needing neither FUSE nor a host zlib : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L33 ]
[test: both Linux artifacts are built for x64 and arm64 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L41 ]

## S13: the remembered position was on a monitor that is gone @v1 [proposed]

- Given: the app was last used on an external monitor that is now unplugged
- When: the app launches
- Then: the window opens centered at the default 1280×860 on a connected
  display

[test: a window on a disconnected monitor falls back to null : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L87 ]

## S14: the window cannot shrink below a usable size @v1 [proposed]

- Given: the desktop app is open
- When: the user drags a window edge as far as it will go
- Then: the window stops at 1280×700, with every button and scrollbar still
  visible

[test: a too-small saved size is raised to the minimums : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L76 ]

## S15: the window moves from the top of either pane @v1 [proposed]

- Given: the macOS desktop app, with the editor pane open on a spec or the
  README
- When: the user presses on the editor pane's header row — the trail, the
  empty space beside it, or the band above it — and drags
- Then: the window moves, as it does from the sidebar header; a click on a
  crumb, the feature's name or an icon in that row still does what it names

[test: the editor pane's header is a drag region too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L194 ]
[test: every control in the editor header stays clickable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L199 ]
[test: off macOS the editor header is not a drag region : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L211 ]
[test: the opt-out is scoped to the two headers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragRegion.test.ts#L225 ]

## S16: the window reopens where it was left @v1 [proposed]

- Given: the user resized and moved the window, then quit the app
- When: they launch the app again
- Then: the window opens at the same size and position

[test: save then load round-trips : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L120 ]
[test: bounds fully on-screen and large enough pass through unchanged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowState.test.ts#L72 ]

## S17: the Windows app is built on Windows @v1 [proposed]

- Given: a Windows machine
- When: the user runs `bun run build:desktop`, or `--win`
- Then: the NSIS installer is built; with `--local`, only the app unpacked for
  this machine's architecture; with `--host-arch`, the installer for this
  machine's architecture alone, `SpecDriven-<version>-<arch>-setup.exe`
- and `--win` on macOS or Linux stops before packaging, saying the Windows app
  can only be built on Windows, while `--all` there still means macOS and Linux

[test: the Windows app is only built on Windows, and --all leaves it out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L38 ]
[test: without flags the build targets the machine it runs on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L22 ]
[test: --local packages only the installer this machine starts, for its own architecture : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L55 ]
[test: --host-arch packages the release installers for this machine's architecture only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L71 ]

## S18: every artifact carries the application icon @v1 [proposed]

- Given: the artwork in `build/icon.svg` and the renderings `bun run build:icons`
  committed beside it
- When: the desktop app is packaged, or the web app is built
- Then: electron-builder stamps the `.icns`, `.ico` and Linux PNGs from
  `build/`, each the size its name says and laid out the way its platform reads it
- and macOS and Linux get the tile on Apple's transparent margin, Windows the
  full-bleed one
- and the web app's favicon is the same artwork

[test: electron-builder is pointed at committed icons for every platform : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L22 ]
[test: build/icons carries every Linux size, each file the size it names : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L31 ]
[test: the .icns and .ico are what their formats expect : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L39 ]
[test: encodeIco lays out PNG entries the way Windows reads them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L61 ]
[test: macOS and Linux renderings sit on Apple's margin; the Windows one is full-bleed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L83 ]
[test: the iconset names each file by its point size, with the @2x double beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L99 ]
[test: the web app's favicon is the same artwork : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appIcon.test.ts#L107 ]

## S19: the package carries the bundles and nothing else @v1 [proposed]

- Given: a desktop build on a machine whose `dist/` also holds the web
  binary and a crash test's output, and whose `node_modules` holds every
  development dependency
- When: electron-builder packages the app
- Then: `app.asar` holds the main-process and renderer bundles, the icon and
  a self-fix's patch manifest — no `node_modules`, no source maps, nothing
  else from `dist/`
- and the Electron framework ships only its English locale pack, so the
  macOS app is a fraction of the size the dependency tree would give it

[test: the package ships the bundles, not node_modules or the rest of dist/ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L64 ]
[test: source maps stay out of the package : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L74 ]
[test: only the English Chromium locale ships : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L78 ]

## S20: the bundle runs on a machine without the build machine's node_modules @v1 [proposed]

- Given: a desktop build cut on one machine — a Linux deb or AppImage
  cross-built on a Mac, or a macOS app copied from the Mac that built it
- When: the app starts on a machine that has no `node_modules` at the build
  machine's path
- Then: the main process reads Automerge's wasm from beside its own bundle
  and the window opens on `app://`
- and cbor-extract's native extractor loads from beside the bundle where a
  prebuild for that platform shipped, and cbor-x falls back to JavaScript
  where none did
- and a build whose bundle still names a path on the build machine fails
  instead of shipping

[test: Automerge's nodejs entry reads its wasm from beside the bundle, not the builder's node_modules : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L113 ]
[test: cbor-extract hands its loader the cbor-extract/ directory beside the bundle : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L126 ]
[test: a bundle that still bakes a build-machine path in is caught : https://github.com/SpecDriven/specdriven-app/blob/main/tests/buildDesktop.test.ts#L136 ]
