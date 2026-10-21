# Editor Autosave

## S1: A Body's Indentation Survives the Save @v1 [proposed]

- Given: a scenario body whose first line is indented — an indented code block, a nested list item, a tab
- When: the scenario is saved
- Then: the body is kept as written, indentation included, and the feature file reads back as the same feature
- And: a first line of four spaces and three backticks is an indented code block, not a code fence — the save lands, and so does the next one

The parser used to trim the body, which took the indentation off its first line. For an indented fence line that turned the line into a fence opener: the file was written with a fence never closed, every later save of the feature was refused, and the scenario under it vanished from the editor while still being in the document.

[test: a body's first-line indentation survives the round trip : https://github.com/SpecDriven/specdriven-app/blob/main/tests/parser.test.ts#L370 ]
[test: a description's first-line indentation survives too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/parser.test.ts#L381 ]
[test: trimBlankLines drops blank edge lines and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/parser.test.ts#L388 ]
[test: a body opening with an indented fence line saves, and saves again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L976 ]

## S2: A Failed Autosave Is Retried @v1 [proposed]

Only a save that lands counts as saved: a failed autosave leaves the editor's
contents unsaved and reports on the status banner, and the next tick tries
again.

## S3: A README Autosave Keeps a Disk Edit @v1 [proposed]

- Given: the README is open in the editor and the file changes on disk (a git
  pull, another editor) before the next autosave
- When: the autosave tick sends the buffer, with the last saved text as its
  base
- Then: the server merges the disk edit and the buffer three ways instead of
  overwriting the disk, and the editor takes the merged text
- and a buffer that already holds the disk edit saves it once, not twice
- and if the buffer moved on while the save was in flight, the base stays at
  what was sent, so the next tick merges the disk edit in again
- and a README deleted on disk is recreated as typed
- and a save with no base replaces the file outright

[test: keeps the disk edit alongside the save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L122 ]
[test: an editor that already holds the disk edit saves it once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L132 ]
[test: a save with no disk edit writes the buffer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L140 ]
[test: a buffer that moved on during the fold keeps the disk edit : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L149 ]
[test: a README gone from disk is recreated as the editor holds it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L161 ]
[test: a save without from replaces the file outright : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appStartup.test.ts#L169 ]

## S4: A Scenario With No Name Is Not Autosaved @v1 [proposed]

A scenario with no name has nothing to be saved under, so no tick saves it.
The dialog's Save button is disabled for the same reason.

## S5: An Invalid Autosave Interval Is Refused @v1 [proposed]

Settings refuses an interval below the 3 second minimum with a message, and
changes nothing — the interval already saved keeps running. The interval is
stored in whole seconds.

## S6: Autosave Cannot Be Switched Off @v1 [proposed]

The setting is how often autosave runs, not whether it runs. Autosave is
independent of autosync (ui.feature.md "Settings"): turning autosync off,
which stores a frequency of 0, leaves autosave running at its own interval.

## S7: Autosave Never Adds the Same Scenario Twice @v1 [proposed]

The first tick on a new scenario places it in the feature; every later tick
rewrites that same entry.

The saved feature comes back in canonical order (by name, then version), so
the open dialog follows its scenario to wherever it moved, finding it by name
and version. A new scenario keeps deriving its version while the dialog is
open, with its own entry left out of the derivation — otherwise it would bump
past itself on every tick.

If the entry has gone from underneath the dialog, the next save re-adds the
work rather than overwriting a neighbour.

## S8: Autosave Only Saves What Changed @v1 [proposed]

A tick compares what the dialog holds — the name, the version it will save
under, the status, the body and the test links — against what it last had
saved. Nothing new, no save.

A scenario that is opened and read but not edited is never re-saved, so
reading a scenario costs no commit.

## S9: Autosave Runs Only While the Editor Is Open @v1 [proposed]

Ticks start when the scenario edit dialog opens and stop when it closes.

A save still in flight when the dialog closes never stamps its result onto
whatever the user opened next.

Changing the interval in Settings takes effect on the editor that is already
open.

## S10: Autosave Saves Locally @v1 [proposed]

An autosave writes the feature locally, exactly as the dialog's Save does; the
reload cron commits and pushes it (git-operations.feature.md).

Autosave is not a publish — a tick never advances a scenario's status.

## S11: Autosave Stands Down While a Delete Is Confirmed @v1 [proposed]

From the moment Delete is pressed in the scenario edit dialog until the
removal is away, no tick fires: a save landing while the confirmation is on
screen would put the scenario back, or re-sort the feature under the entry
being deleted.

## S12: Autosave Ticks Never Overlap @v1 [proposed]

A tick that comes round while the previous save is still in flight does
nothing.

## S13: Autosave Triggered at Defined Interval @v1 [published]

SpecDriven saves the contents of the editor at the defined interval.

## S14: Autosave Waits While a Code Fence Is Open @v1 [proposed]

- Given: the scenario body in the editor ends inside a code fence — a gherkin block whose closing fence is not typed yet
- When: the autosave tick comes round
- Then: the tick saves nothing, and the tick after the fence closes saves the scenario
- and a save sent anyway (Save, Propose, a publish) is refused, naming the line of the open fence, and the feature keeps every scenario
- and the sync tick leaves a feature file on disk that ends inside a fence alone rather than absorb it

Text ending inside a fence reads every later scenario heading as fenced content: the scenarios under them are swallowed into the body above and would be removed from the feature as if deleted. The buffer is in that state for a few keystrokes whenever a fence is typed, which is how a proposed scenario was once erased by an autosave.

[test: unclosedFence reports the line of a fence left open at the end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/parser.test.ts#L61 ]
[test: featureTextError refuses a feature whose last code fence is never closed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/parser.test.ts#L69 ]
[test: is refused, and keeps the scenario the open fence would swallow : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L291 ]
[test: lands once the fence is closed again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L301 ]
[test: a publish is refused the same way : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L309 ]
[test: a file left open on disk is not absorbed by the drift pass : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L314 ]

## S15: Autosave Waits While the Name Is Being Typed @v1 [proposed]

While the caret is in the dialog's title field the scenario's name is only
half typed, so no tick saves it — a new scenario is not filed as a draft under
the first few letters of its name, and a rename is not saved part-way through.

The tick after focus moves on — to the body, or anywhere else — saves the name
as typed. A published scenario has no editable title, so nothing holds its
ticks back.

## S16: Canceling an Autosaved Edit Restores the Scenario @v1 [proposed]

Cancel undoes the dialog, ticks included. A scenario that was already saved —
a draft the user saved, or a proposed one — goes back to what the dialog
opened it as: the text typed since is taken back out of the feature even
though a tick had saved it.

A rename is taken back with the rest. A renamed draft is deleted under the
name the tick saved it under, because the `.feature.md` never held it, and a
save that leaves a draft out removes nothing.

A dialog no tick ever saved has nothing to restore — Cancel just closes it.

## S17: Canceling Autosaved Scenario Deletes it @v1 [published]

Given: a new scenario was autosaved and is in 'draft' status
When: I Cancel from the scenario editor
Then: This draft scenario is deleted

Note: if this scenario was saved by the user, next Cancel jsut exits the editor dialog.

## S18: Configure Autosave in Settings @v1 [proposed]

Given: the Settings page (user-info-settings.feature.md "open Settings")
When: the user changes "Autosave interval (seconds)" in its Specs section
Then: the new interval is saved to .specdriven/settings.json as soon as the
field is left, with no Save button, and the open editor autosaves at it
- the interval is at least 3 seconds and 6 seconds by default; a smaller
  one typed in the field turns back into the 6 second default, and the
  server refuses one below 3 and changes nothing

This replaces "Configure Autosave in Settings Dialog": Settings is a page in
the editor pane, not a dialog.

[test: the autosave interval defaults to 6 seconds and never goes below 3 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/editorAutosave.test.ts#L57 ]
[test: Settings saves the autosave interval to the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/editorAutosave.test.ts#L81 ]
[test: an interval below the 3 second minimum is refused, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/editorAutosave.test.ts#L90 ]

## S19: Configure Autosave in Settings Dialog @v1 [published]

The SpecDriven App Settings contain a setting for Autosave Interval in Seconds.

The user can edit the Autosave Interval with a minimum of 3 seconds and a default of 6 seconds.

## S20: Publish Waits for an In-flight Autosave @v1 [proposed]

- Given: the scenario editor is open and an autosave is in flight or queued
- When: the user presses Publish
- Then: the publish goes through the same save queue, waits for that
  autosave, and places the scenario where the save left it, so a new scenario
  is not published twice and a rename does not overwrite the scenario beside it
- and the button shows "Publishing…" from the click
- and when the publish lands the user is still on that feature, it reopens it
  (or shows its conflict dialog); if they moved to another feature it only
  refreshes the listing and does not pull them back
- and a feature whose path was sealed meanwhile by a move, rename or delete
  is refused: nothing goes out

[test: another feature opened meanwhile — it publishes, but refreshes the listing instead of pulling the user back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L185 ]
[test: the path sealed while it waited (move, rename-file, delete) — nothing goes out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L228 ]
[test: placed from the render, Publish sends a second Beta and publishes that — the bug : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L51 ]
[test: queued after the tick, it publishes the one scenario the tick placed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L58 ]
[test: placed at the render's stale index, Publish writes over its neighbour — the bug : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L99 ]
[test: queued after the tick, it publishes its own entry and keeps every neighbour : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L104 ]
[test: still on its feature, it reopens that feature : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L168 ]
[test: a conflict for a feature no longer on screen is reported, not opened over the new page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/publishBehindTick.test.ts#L209 ]
