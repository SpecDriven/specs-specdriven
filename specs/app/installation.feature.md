# Installation

**Assigned:** David

Specdriven App is distributed via a zip archive in order to allow users
to run their security checks on the code before building the app.

Releases will be in this repo: https://github.com/SpecDriven/specdriven-app-release

install.sh script will be available in https://github.com/SpecDriven/specdriven-app-release in
order to install via curl -- as is popuilar these days.

## What the script does

One command takes a machine from nothing to a built app:

- Follow `releases/latest` to the newest `v<version>`, or take
  `SPECDRIVEN_VERSION`.
- Download `specdriven-<version>-src.zip` and `SHASUMS256.txt`, and verify one
  against the other.
- Unpack to `~/.specdriven/app/specdriven-<version>`, with
  `~/.specdriven/app/current` pointing at it and the verified zip kept
  alongside for anyone who wants to check it themselves.
- Install the dependencies and build the desktop app. On Linux the
  `.AppImage` is started; on macOS nothing is started, and the `.dmg` is opened
  last so the app can be dragged into Applications.
- Build with `bun run build:desktop --local`: only this machine's architecture
  and only the installer it runs (the `.AppImage` or the `.dmg`). The release's
  `.deb` is left out because its packager needs `libcrypt.so.1`, which Arch and
  other non-Debian distros lack.

It is POSIX sh, because it runs on whatever `/bin/sh` the machine has before
anything is known about it, and it works with curl or wget, with unzip or
bsdtar or python3, and with sha256sum or shasum. Windows shells are sent to
the releases page instead.

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

Nothing unverified is ever written: a checksum mismatch, a missing or
incomplete `SHASUMS256.txt`, a 404, or a machine with no sha256 tool at all
stops the install with nothing left behind, scratch directories included. A
version already unpacked is not unpacked over — someone may have edited or
built in it — though the build still runs on it as it stands.

Once the source is on disk it stays there: a failed `bun install` or build is
an error that names the checkout, so the code can be looked at or built by
hand.

## The source zip

The zip is the product, so it has to stand on its own: everything in it
resolves inside it. The checkout's `specs/` is a symlink to the sibling spec
repository, which no release ships — left in, it unpacks dangling and fails
the desktop build, since electron-builder stats every path in the project.

## a build that fails leaves the source in place @v1 [proposed]

- Given: a verified release unpacked under `~/.specdriven/app`
- When: `bun install` or the desktop build fails
- Then: the run stops with the failure and the path to the checkout, which is
  still there to build by hand

[test: a build that fails stops the run and says where the checkout is : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L803 ]
[test: a failed `bun install` is reported the same way : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L816 ]

## bun is not installed @v1 [published]

If `bun` is not installed, offer to install it and if user accepts install bun 
under the current users authority.

[test: a yes at the terminal installs Bun, as the user and in their home : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L583 ]
[test: a no at the terminal leaves Bun alone : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L602 ]
[test: with no terminal to ask at, Bun is left to the reader : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L615 ]

## install via curl @v1 [proposed]

A user can install specdriven app via curl.

Example: curl -fsSL https://github.com/SpecDriven/specdriven-app-release/install.sh | sh

By default, the script will attempt to execute the entire install.

If the user want to scan the code before installing, they will download the zip file manually.

[test: the latest release is downloaded, verified and unpacked under ~/.specdriven/app : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L226 ]

## the checksum has to match before anything is unpacked @v1 [proposed]

- Given: a release whose zip does not match its `SHASUMS256.txt`, or has none
- When: the user runs the installer
- Then: it stops saying so, and `~/.specdriven` holds nothing from that run

[test: a checksum mismatch aborts without unpacking anything : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L328 ]
[test: a release without SHASUMS256.txt is refused rather than installed unverified : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L351 ]
[test: a SHASUMS256.txt that does not list the zip is as good as none : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L366 ]

## the install ends with the app built and handed over @v1 [proposed]

- Given: a machine with Bun, and a release to install
- When: the user runs the installer
- Then: the dependencies are installed and the desktop app is built into
  `release/`. On macOS the `.app` is not started; the last thing the
  installer does is open `release/SpecDriven-<version>-arm64.dmg`. On Linux
  the AppImage is made executable and started, and on a machine with no
  display it is built and left with its path printed rather than started.
  The messages come through whatever locale the shell runs in, single-byte
  ones included.

[test: the run ends with the .dmg opened, and nothing started : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L699 ]
[test: the messages survive a single-byte locale : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L733 ]
[test: the run ends with the AppImage made runnable and started : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L757 ]
[test: with no display the app is built but left unstarted : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L785 ]

## the source zip is self-contained @v1 [proposed]

- Given: the release source zip
- When: it is unpacked on a machine that has never seen this project
- Then: nothing in it points outside itself, so the build has no dangling path
  to trip over

[test: symlinks pointing out of the repo are kept out of the release source zip : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L843 ]

## without Bun the install stops at the source @v1 [proposed]

- Given: a machine with no Bun, whose user declines to install it
- When: the installer has unpacked the release
- Then: it stops there and prints the commands to install the dependencies,
  run the web app and build the desktop app

[test: the next steps say whether Bun is there to run them : https://github.com/SpecDriven/specdriven_app/blob/main/tests/install.test.ts#L513 ]
