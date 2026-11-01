# Installation

**Assigned:** David

Specdriven App is distributed via a zip archive in order to allow users
to run their security checks on the code before building the app.

Releases are published on the app's own repository:
https://github.com/SpecDriven/specdriven-app/releases

install.sh and install.ps1 are served straight from that repository's `main`
branch, in order to install via curl (or PowerShell) -- as is popular these days.

## What the script does

One command takes a machine from nothing to a built app:

- Follow `releases/latest` to the newest `v<version>`, or take
  `SPECDRIVEN_VERSION`.
- Download `specdriven-<version>-src.zip`, `SHASUMS256.txt` and
  `SHASUMS256.txt.sig`; check that the signature was made by the SpecDriven
  release key (an SSH signature, `ssh-keygen -Y verify`, against the public
  key the script embeds), then that the zip matches `SHASUMS256.txt`.
- Unpack to `~/.specdriven/app/specdriven-<version>`, with
  `~/.specdriven/app/current` pointing at it and the verified zip kept
  alongside for anyone who wants to check it themselves.
- Install the dependencies and build the desktop app. A Mac needs macOS 13
  or later: Electron 44 dropped macOS 12, so an older one fails inside the
  build rather than at install. On Linux the
  `.AppImage` is started; on macOS nothing is started, and the `.dmg` is opened
  last so the app can be dragged into Applications.
- Build with `bun run build:desktop --local`: only this machine's architecture
  and only the installer it runs (the `.AppImage` or the `.dmg`). The release's
  `.deb` is left out because its packager needs `libcrypt.so.1`, which Arch and
  other non-Debian distros lack.

It is POSIX sh, because it runs on whatever `/bin/sh` the machine has before
anything is known about it, and it works with curl or wget, with unzip or
bsdtar or python3, with sha256sum or shasum, and with ssh-keygen (OpenSSH
8.1 or later, which macOS, Linux and Windows 10 and 11 ship). A Windows shell (Git Bash,
MSYS, Cygwin) is pointed at `install.ps1` instead, with the releases page as
the way to do it by hand.

## On Windows

`install.ps1` is the same installer for PowerShell:

    irm https://raw.githubusercontent.com/SpecDriven/specdriven-app/main/install.ps1 | iex

It takes the same steps, overrides and checksum rules, under
`%USERPROFILE%\.specdriven\app`. `current` is a junction, which needs neither
admin rights nor Developer Mode. It needs nothing Windows 10 and 11 do not
ship (Windows PowerShell 5.1 or PowerShell 7), and it is ASCII only, because
5.1 reads a script file without a BOM in the legacy code page.

Bun is offered the same way, at the console, and comes from Bun's own
`install.ps1` — downloaded first, then run — never from an elevated
(Administrator) PowerShell. `bun run build:desktop --local` leaves the app
unpacked in `release\win-unpacked` (`win-arm64-unpacked` on ARM): the script
adds a Start menu shortcut to it through `current`, so the next install moves
the shortcut along, and starts it. The `irm | iex` line runs the script in the
caller's own session, so it keeps its variables to itself and a failure ends
the script without closing their window.

## Bun

Building needs Bun, which is the one thing the script installs outside
`~/.specdriven`, so it is the one thing it asks about. `curl | sh` has already
spent the script's own stdin on the pipe, so the question and its answer go to
the terminal directly; with no terminal there is nobody to ask, and the
question is skipped rather than answered on the user's behalf.

A yes runs Bun's own installer — downloaded first, then run — as the invoking
user, into `BUN_INSTALL` (default `~/.bun`). Never with sudo, never
system-wide, and never as root: an install as root would leave a tree the user
cannot write to. `SPECDRIVEN_INSTALL_BUN` settles the question in advance for
an unattended run.

## Failing closed

Nothing unverified is ever written: a signature that is missing or does not
verify, a checksum mismatch, a missing or incomplete `SHASUMS256.txt`, a 404,
or a machine with no sha256 tool or no ssh-keygen at all stops the install
with nothing left behind, scratch directories included. A
version already unpacked is not unpacked over — someone may have edited or
built in it — though the build still runs on it as it stands.

Once the source is on disk it stays there: a failed `bun install` or build is
an error that names the checkout, so the code can be looked at or built by
hand.

## Older versions

Each install unpacks a whole tree, a gigabyte or more once built, so updates
would otherwise pile up under `~/.specdriven/app`. When `current` moves from
one version to another, the version it pointed at until then stays, to go back
to, and every other `specdriven-<version>` goes, with its zip and checksum.
That happens before the build, so a build that fails still has the old version
beside it. A first install, a reinstall of the same version, or a `current`
the script did not make (a real directory) clears nothing, and nothing under
`app` that is not named for a version is touched.

## The source zip

The zip is the product, so it has to stand on its own: everything in it
resolves inside it. The checkout's `specs/` is a symlink to the sibling spec
repository, which no release ships — left in, it unpacks dangling and fails
the desktop build, since electron-builder stats every path in the project.

## S1: a build that fails leaves the source in place @v1 [proposed]

- Given: a verified release unpacked under `~/.specdriven/app`
- When: `bun install` or the desktop build fails
- Then: the run stops with the failure and the path to the checkout, which is
  still there to build by hand

[test: a build that fails stops the run and says where the checkout is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L943 ]
[test: a failed `bun install` is reported the same way : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L956 ]

## S2: bun is not installed @v1 [published]

If `bun` is not installed, offer to install it and if user accepts install bun 
under the current users authority.

[test: a yes at the terminal installs Bun, as the user and in their home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L706 ]
[test: a no at the terminal leaves Bun alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L725 ]
[test: with no terminal to ask at, Bun is left to the reader : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L738 ]

## S3: install on Windows from PowerShell @v1 [proposed]

- Given: a Windows machine
- When: the user runs `irm https://raw.githubusercontent.com/SpecDriven/specdriven-app/main/install.ps1 | iex`
- Then: the latest release is downloaded, verified against `SHASUMS256.txt`
  and unpacked under `%USERPROFILE%\.specdriven\app`, with `current` pointing
  at it, exactly as `install.sh` does it
- and a missing Bun is offered, and installed only on a yes
- and the app is built with `bun run build:desktop --local`, added to the
  Start menu and started from `release\win-unpacked\SpecDriven.exe`
- and a Windows shell running `install.sh` is pointed at this command

[test: the latest release is downloaded, verified and unpacked under ~/.specdriven/app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L162 ]
[test: install.ps1 is plain ASCII, which Windows PowerShell 5.1 reads as written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L152 ]
[test: SPECDRIVEN_VERSION picks a release, with or without the v : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L189 ]
[test: a version that is not version-shaped is refused before any download : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L198 ]
[test: a checksum mismatch aborts without unpacking anything : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L208 ]
[test: a release without SHASUMS256.txt, or one not listing the zip, is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L225 ]
[test: no releases at all is reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L317 ]
[test: a version already unpacked is left alone, and `current` moves between versions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L326 ]
[test: a `current` that is a real directory is never replaced : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L371 ]
[test: SPECDRIVEN_INSTALL_BUN=1 installs Bun as the user, into their home, and builds with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L413 ]
[test: a yes at the terminal installs Bun; a no leaves it alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L439 ]
[test: with no one to ask, Bun is left alone and never downloaded : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L463 ]
[test: a Bun installer that fails is reported, and the checkout still stands : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L478 ]
[test: the run ends with the app built and started from release/win-unpacked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L492 ]
[test: a build that fails stops the run and says where the checkout is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L513 ]
[test: Windows shells are sent to install.ps1, or the releases page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L580 ]
[test: with no override, the releases come from SpecDriven/specdriven-app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L528 ]

## S4: install via curl @v1 [proposed]

A user can install specdriven app via curl.

Example: curl -fsSL https://raw.githubusercontent.com/SpecDriven/specdriven-app/main/install.sh | sh

By default, the script will attempt to execute the entire install.

If the user want to scan the code before installing, they will download the zip file manually.

[test: the latest release is downloaded, verified and unpacked under ~/.specdriven/app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L152 ]
[test: with no override, the releases come from SpecDriven/specdriven-app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L1053 ]

## S5: the checksum has to match before anything is unpacked @v1 [proposed]

- Given: a release whose zip does not match its `SHASUMS256.txt`, or has none
- When: the user runs the installer
- Then: it stops saying so, and `~/.specdriven` holds nothing from that run

[test: a checksum mismatch aborts without unpacking anything : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L256 ]
[test: a release without SHASUMS256.txt is refused rather than installed unverified : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L277 ]
[test: a SHASUMS256.txt that does not list the zip is as good as none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L292 ]
[test: without ssh-keygen the download is not installed, however good it may be : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L404 ]
[test: an ssh-keygen that cannot verify stops the install, in its own words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L303 ]

## S6: the install ends with the app built and handed over @v1 [proposed]

- Given: a machine with Bun, and a release to install
- When: the user runs the installer
- Then: the dependencies are installed and the desktop app is built into
  `release/`. On macOS the `.app` is not started; the last thing the
  installer does is open `release/SpecDriven-<version>-arm64.dmg`. On Linux
  the AppImage is made executable and started, and on a machine with no
  display it is built and left with its path printed rather than started.
  The messages come through whatever locale the shell runs in, single-byte
  ones included.

[test: the run ends with the .dmg opened, and nothing started : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L832 ]
[test: the messages survive a single-byte locale : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L867 ]
[test: the run ends with the AppImage made runnable and started : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L895 ]
[test: with no display the app is built but left unstarted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L924 ]

## S7: the source zip is self-contained @v1 [proposed]

- Given: the release source zip
- When: it is unpacked on a machine that has never seen this project
- Then: nothing in it points outside itself, so the build has no dangling path
  to trip over

[test: symlinks that dangle in the release source zip are kept out of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L985 ]

## S8: without Bun the install stops at the source @v1 [proposed]

- Given: a machine with no Bun, whose user declines to install it
- When: the installer has unpacked the release
- Then: it stops there and prints the commands to install the dependencies,
  run the web app and build the desktop app

[test: the next steps say whether Bun is there to run them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L630 ]

## S9: an update keeps one version to go back to @v1 [proposed]

- Given: earlier versions unpacked under `~/.specdriven/app`, with `current`
  pointing at one of them
- When: the installer puts another version in and points `current` at it
- Then: the version `current` pointed at until then stays, and every other
  `specdriven-<version>` goes with its zip and checksum, before the build
- and a first install, a reinstall of the same version, or a `current` that is
  a real directory clears nothing out

[test: an update keeps the version it replaces and clears out the older ones : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L522 ]
[test: going back to an older version keeps the newer one it replaces : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L552 ]
[test: a first install, or the same version again, clears nothing out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L564 ]
[test: a `current` that is a real directory is never replaced : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L497 ]
[test: an update keeps the version it replaces and clears out the older ones : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L343 ]

## S10: a release that is not signed by the release key is refused @v1 [proposed]

- Given: a release whose `SHASUMS256.txt.sig` is missing, was made by some
  other key, was made in some other namespace, or no longer matches
  `SHASUMS256.txt` because the zip and the checksum file were swapped after
  signing
- When: the user runs the installer
- Then: it stops before comparing any checksum, saying the file is not signed
  by the SpecDriven release key and where the current installer is (the key
  may have been rotated), and `~/.specdriven` holds nothing from that run
- and the key the installers trust is the one line the release workflow
  signs with and checks its own signature against, in `install.sh`,
  `install.ps1` and the docs alike

[test: a release without SHASUMS256.txt.sig is refused rather than installed unsigned : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L316 ]
[test: a release swapped for another, checksum file and all, fails the signature and is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L333 ]
[test: a release signed with some other key is refused, and says a newer install.sh may know it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L362 ]
[test: a signature by the right key in some other namespace does not count : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L378 ]
[test: SPECDRIVEN_RELEASE_PUBKEY aside, the installers trust the one key the release workflow checks against : https://github.com/SpecDriven/specdriven-app/blob/main/tests/install.test.ts#L413 ]
[test: a release without SHASUMS256.txt.sig is refused rather than installed unsigned : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L251 ]
[test: a release swapped for another, checksum file and all, fails the signature and is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L265 ]
[test: a release signed with some other key is refused, and says a newer install.ps1 may know it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/installPs1.test.ts#L290 ]
