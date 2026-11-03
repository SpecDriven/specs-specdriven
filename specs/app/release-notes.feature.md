# Release notes

Every release says what it changed. The notes are made, not written: each
change lands as one squash-merged pull request whose title says what the user
can now do or see, so a release's notes are the titles of the pull requests
merged since the release before it, each linked to its pull request.

- `CHANGELOG.md` at the root of the app's repository lists every release,
  newest first: a `## v<version> — <date>` heading and a line per change.
  Nobody edits it to cut a release: `bun run bump` writes the new version's
  section into the release commit, beside `package.json`, from the
  first-parent commits since the last `v*` tag (`windows-v*` tags do not
  count). Release commits, `chore` commits and dependency bumps are left
  out; a release with nothing else says "Maintenance release."
- The GitHub release's body starts with that section under *What's
  changed*, above the install and verify notes. The release workflow takes
  it from the tagged `CHANGELOG.md` and refuses to publish a tag whose file
  has no section for its version.
- The app shows the same notes on a *What's new* page, bundled from
  `CHANGELOG.md` into the build, so every copy — desktop, local, demo,
  hosted and the read-only specs site — shows the notes it was built with.
- The releases before notes existed were backfilled once from the tags.

## S1: A release lists the pull requests merged since the one before @v1 [proposed]

- Given: pull requests merged on `main` since the last release tag, and a
  dependency bump
- When: the maintainer runs `bun run bump patch`
- Then: `CHANGELOG.md` gains a section for the new version on top, dated
  today, with a line per pull request — its title and a link to it — and
  none for the dependency bump
- and the release commit holds `package.json` and `CHANGELOG.md`, and the
  new tag points at it
- and `--dry-run` prints the section without writing anything
- and once the tag is pushed, the GitHub release opens with the same lines
  under *What's changed*

[test: each merged pull request is one note, titled as it was merged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L23 ]
[test: releases, dependency bumps and upkeep are left out of the notes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L37 ]
[test: a version's section is its heading and a linked line per pull request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L69 ]
[test: the newest release goes on top, and a version is written once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L80 ]
[test: the release workflow's section is the lines under the version's heading : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L87 ]
[test: bump commits the version and its notes together, and tags that commit : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L131 ]
[test: a dry run shows the notes it would write and touches nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L154 ]
[test: git log records split into subject and body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L51 ]
[test: the shipped CHANGELOG.md lists the version package.json names, newest first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L99 ]

## S2: An update marks the version until its notes are opened @v1 [proposed]

- Given: the user has run SpecDriven before in this browser or window, and
  has just updated to a newer version
- When: the app opens
- Then: a dot marks the version beside the user name and the user menu's
  *What's new* entry
- and opening *What's new* — from the menu or its url — clears both
- and a first run marks nothing: there is nothing new to point at yet

[test: an update is marked until its notes are opened; a first run is not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/whatsNewPage.test.tsx#L98 ]
[test: the user menu's What's new opens the page, with a dot while there are unread notes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/whatsNewPage.test.tsx#L116 ]

## S3: What's new lists every release, newest first @v1 [proposed]

- Given: any copy of the app, the read-only specs site included
- When: the user picks *What's new* in the user menu, or opens `/whats-new`
- Then: the editor pane shows every release's notes, newest first, with its
  date and a line per change linking to its pull request
- and the running version's release is marked *Installed*
- and the section list on the left jumps to the most recent releases
- and ✕ or ESC goes back to what was open before

[test: What's new lists every release newest first, each change linked to its pull request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/whatsNewPage.test.tsx#L67 ]
[test: the installed version's release is marked as installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/whatsNewPage.test.tsx#L84 ]
[test: the build carries the notes for the version it is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/whatsNewPage.test.tsx#L94 ]
[test: What's new reads back every release the file lists, newest first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/releaseNotes.test.ts#L94 ]
[test: What's new has its own url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L350 ]
