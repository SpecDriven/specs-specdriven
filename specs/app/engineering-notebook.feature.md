# Engineering notebook

A shared, dated log of what we notice about the way we work and what we
decide to do about it. Observations ("the drift import bit us twice this
week") and decisions ("from now on a spec change lands before the code")
about our future process, kept in git beside the specs so every user of the
data repo reads and writes the same notebook.

It is not a chat and not a task list. An entry is written once, signed and
dated, and stays as the record of what we knew and chose at the time. The
roadmap says where we want to go; the tasks say who does what next; the
notebook says what we learned on the way and why the process looks the way
it does.

## Shape

- `notebook/` is a special folder at the root of the data repo, like
  `roadmap/` and `tasks/`, holding one regular markdown file per entry
- an entry's file is `notebook/<yyyy-mm-dd>-<slug>.md`, so the folder reads
  in order on GitHub and the list sorts without opening a file; nested
  subfolders are not offered — a notebook is flat and chronological
- the file is self-describing: a `# <title>` heading, then an
  `**Author:** <name>`, `**Date:** <yyyy-mm-dd>` and
  `**Kind:** Observation | Decision` line, then the body; a plain markdown
  file dropped into the folder by hand, with none of those lines, is still an
  entry — its date comes from the file name when it carries one, else from
  git, and it reads as an observation
- the author is the identity that authors the app's commits to the data repo
  (user-info-settings.feature.md): the GitHub login when signed in, else the
  git user name
- like a roadmap page, an entry has an Automerge document behind it
  (datastore/automerge-store.feature.md): concurrent edits merge, an entry
  edited on GitHub or in an older app version is absorbed on the next sync
  tick, and the sync tick commits and pushes every saved entry
- the "Engineering notebook" switch in Settings is **on by default** and
  device-local, like the other module switches; unlike the roadmap the
  folder is not created on sync but with the first entry, so a default-on
  module never touches a data repo by itself

## S1: A decision can supersede an earlier one @v1 [proposed]

- Given: a decision entry is open
- When: the user picks "Supersedes…" in the entry's header and chooses an
  earlier decision from the list
- Then: the entry gains a `**Supersedes:** [<title>](<entry link>)` line
  under its kind, and the earlier entry shows a "Superseded by <title>"
  badge in its header and a muted row in the list
- and the earlier entry's text is untouched — the notebook never rewrites
  history, it adds to it
- and removing the line from the newer entry, in the app or on GitHub,
  removes the badge on the next sync

[test: a decision supersedes an earlier one without touching it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L238 ]

## S2: Deep links @v1 [proposed]

- Given: the setting is on
- When: the user opens `/notebook/<entry>` — `<entry>` being the file name
  without `.md`, `2026-09-12-spec-before-code` — from the address bar, the
  command palette, a Copy link or a link in another entry
- Then: the entry opens in the editor and the sidebar turns to the Notebook
  tab, the way opening a roadmap page turns it to Roadmap
- and a reload of that url serves the app and reopens the entry
- and an entry that does not exist shows the not-found page, and with the
  setting off the url goes home

[test: an entry url round-trips to its file, and a foreign url does not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L269 ]
[test: notebook routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L78 ]

## S3: Entries are shared through git @v1 [proposed]

- Given: two people work from clones of the same data repo, both with the
  setting on
- When: one of them writes an entry, and the other's next sync tick runs
- Then: the entry appears in the other's Notebook tab with its author and
  date, and opens read-to-write like any page
- and when both edit the same entry at once, their edits merge at the
  document level and neither sees a conflict, as for any page
- and a person with the setting off still pulls the entries — the switch
  hides the tab, it does not stop the sync — so nothing is lost by turning
  it off and on again

[test: entries reach the other clone on a sync tick : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L283 ]
[test: the sync tick absorbs notebook drift whatever the other switches say : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L303 ]
[test: two clones editing one entry merge instead of conflicting : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L341 ]
[test: a merged entry holds each side's edit exactly once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L374 ]
[test: an entry edited outside the app is absorbed on the next tick : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L319 ]

## S4: Renaming an entry keeps its date and its links @v1 [proposed]

- Given: an entry exists, another entry supersedes it, and a third links to
  it in its body
- When: the user renames it from the ⋯ menu
- Then: the `# <title>` heading and the file both change — the file keeps the
  `<yyyy-mm-dd>-` it begins with and takes the new title's slug after it, so
  the folder still reads in order — committed and pushed as a `git mv`, with
  the entry's Automerge document moved alongside so its history survives
- and the entry's author, date and kind are untouched
- and the url follows the file name: an entry that is open stays open at its
  new url, rewritten in place so Back does not land on the retired one
- and every `**Supersedes:**` line and every body link that named the entry
  is pointed at its new url, each saved through its own entry's document so a
  teammate's concurrent edit merges rather than being overwritten; the link
  text is left as written, so the sentence still reads
- and unlike a feature file rename (ui/ui.feature.md), where links to the old
  name go stale, an entry's links are rewritten — an entry is cited by other
  entries, and a dead link there would quietly drop a badge
- and the slug the entry gave up is free again: a new entry with the old
  title, on the same day, is created rather than refused
- and a title whose slug another entry already holds is refused with
  "Notebook entry already exists", the way creating one over it is
- and a new title that slugifies to the name the file already has changes
  only the heading — the way an entry whose heading has drifted from its file
  name is repaired

[test: renaming an entry keeps its date and takes the new title's slug : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L452 ]
[test: renaming an entry carries its document along : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L479 ]
[test: renaming an entry validates, and an unchanged slug only repairs the heading : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L500 ]
[test: the slug a renamed entry gave up is free for a new entry : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L533 ]
[test: supersedes links and body links follow a renamed entry : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L550 ]
[test: renaming an entry is a git mv, committed and pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L612 ]

## S5: Search and the command palette @v1 [proposed]

- Given: the setting is on
- When: the user types in the search field or the command palette
- Then: entries are found by title and body and listed under a Notebook
  heading, each with its kind and date, the way roadmap pages are listed
  under Roadmap (search.feature.md)
- and "New notebook entry" is a palette command

## S6: The notebook follows the data folder @v1 [proposed]

- Given: the setting is on and an entry from the current data folder is open
- When: the user switches to another data folder
- Then: the Notebook tab lists the new folder's entries, the open entry
  closes and the app goes home, and whatever was typed is saved into the
  folder it was typed in — never the new one — exactly as the roadmap
  behaves (roadmap.feature.md)

## S7: The notebook is a sidebar tab @v1 [proposed]

- Given: the Engineering notebook setting is on, as it is by default
- When: the user looks at the sidebar
- Then: a Notebook tab sits to the right of the Tasks tab (right of whichever
  of Specs, Roadmap and Tasks are shown), before Models, Issues and Tests
- and the tab holds a "+ New entry" button and the entries newest first,
  grouped by month ("September 2026"); each row shows the title, a small
  kind chip (Observation or Decision), the author and the day
- and each row's ⋯ menu offers Rename, Copy link and Delete, the way a
  roadmap page's does; Rename moves the file too, so the url follows the
  title; Delete confirms, since the push cannot be undone
- and the chosen tab is remembered across reloads like the other tabs, and
  on a small screen the tabs sit in the drawer and switching keeps it open
- and with no entries yet the tab says the notebook is empty and what it is
  for, in a sentence, above the button

[test: groups by month, newest month first, order within kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.tsx#L59 ]
[test: undated entries come last rather than disappearing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.tsx#L70 ]
[test: a decision is a decision; anything else reads as an observation : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.tsx#L18 ]
[test: names the kind and carries its own class : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.tsx#L29 ]

## S8: The notebook is controlled by a switch in user settings @v1 [proposed]

- Given: a fresh install, or a data folder opened for the first time
- When: the user looks at the sidebar
- Then: there is no Notebook tab — like every plugin, it is off until the
  user turns it on (plugins.feature.md "Each data folder keeps its own
  plugins") — and the Plugins page shows "Engineering notebook" switched
  off with the hint "A shared, dated log of observations and decisions
  about your process, kept in the data repo's notebook folder"
- and an older settings file that names a folder but never mentions the
  notebook — it has the old module switches, or no plugin settings at all —
  still shows it in the folders it knows, since that switch was on by
  default
- When: the user turns it on
- Then: the Notebook tab appears
- and no `notebook/` folder is created and nothing is committed until the
  first entry is written
- When: the user turns the switch off
- Then: the tab disappears; an entry that is open leaves the editor and the
  app goes home; the server refuses the notebook's requests while the switch
  is off; the folder and its entries are left alone
- and the switch is stored in `.specdriven/settings.json` on this machine
  like the roadmap's, so it does not follow the user to another device

[test: the notebook setting is on unless explicitly turned off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L74 ]
[test: a sync tick never creates the notebook folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L84 ]
[test: a fresh install shows Specs alone without a settings write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L88 ]
[test: an older file with a folder but no plugin list keeps the Notebook on in that folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L153 ]

## S9: Write an entry @v1 [proposed]

- Given: the Notebook tab is open
- When: the user presses "+ New entry" (or "New notebook entry" in the
  command palette), picks Observation or Decision, types a title and
  confirms
- Then: `notebook/<today>-<slug>.md` is written with the heading, the
  author, today's date and the kind, and is committed and pushed at once
  with the message `Create notebook entry: <title>`, the way a roadmap page
  is
- and the entry opens in the editor with the cursor in the body; the same
  markdown editor and autosave as a roadmap page (editor/editor-autosave
  .feature.md), and the sync tick commits what is typed
- and a decision opens with a short skeleton in the body — `## Context`,
  `## Decision`, `## Consequences` — that the user is free to delete; an
  observation opens empty
- and a blank title cannot be confirmed, Enter confirms, and a title whose
  slug already exists for today is refused with "Entry already exists"
- and the kind is changed later from a chip in the entry's header, never by
  editing the metadata line; the author and date are set by the app and
  shown, not edited

[test: the notebook is flat — a slash in the title makes no subfolder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L206 ]
[test: an entry is created dated, signed and pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L160 ]
[test: a duplicate slug for the same day is refused, and so is a blank title : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L187 ]
[test: an entry's kind is changed through its document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L220 ]
[test: an entry parses into its metadata and body, and serializes back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L106 ]
[test: a hand-written file is still an entry, dated by its file name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/notebook.test.ts#L134 ]
