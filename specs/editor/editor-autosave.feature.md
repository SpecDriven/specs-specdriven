# Editor Autosave

## A Failed Autosave Is Retried @v1 [proposed]

Only a save that lands counts as saved: a failed autosave leaves the editor's
contents unsaved and reports on the status banner, and the next tick tries
again.

## A Scenario With No Name Is Not Autosaved @v1 [proposed]

A scenario with no name has nothing to be saved under, so no tick saves it.
The dialog's Save button is disabled for the same reason.

## An Invalid Autosave Interval Is Refused @v1 [proposed]

Settings refuses an interval below the 3 second minimum with a message, and
changes nothing — the interval already saved keeps running. The interval is
stored in whole seconds.

## Autosave Cannot Be Switched Off @v1 [proposed]

The setting is how often autosave runs, not whether it runs. Autosave is
independent of autosync (ui.feature.md "Settings"): turning autosync off,
which stores a frequency of 0, leaves autosave running at its own interval.

## Autosave Never Adds the Same Scenario Twice @v1 [proposed]

The first tick on a new scenario places it in the feature; every later tick
rewrites that same entry.

The saved feature comes back in canonical order (by name, then version), so
the open dialog follows its scenario to wherever it moved, finding it by name
and version. A new scenario keeps deriving its version while the dialog is
open, with its own entry left out of the derivation — otherwise it would bump
past itself on every tick.

If the entry has gone from underneath the dialog, the next save re-adds the
work rather than overwriting a neighbour.

## Autosave Only Saves What Changed @v1 [proposed]

A tick compares what the dialog holds — the name, the version it will save
under, the status, the body and the test links — against what it last had
saved. Nothing new, no save.

A scenario that is opened and read but not edited is never re-saved, so
reading a scenario costs no commit.

## Autosave Runs Only While the Editor Is Open @v1 [proposed]

Ticks start when the scenario edit dialog opens and stop when it closes.

A save still in flight when the dialog closes never stamps its result onto
whatever the user opened next.

Changing the interval in Settings takes effect on the editor that is already
open.

## Autosave Saves Locally @v1 [proposed]

An autosave writes the feature locally, exactly as the dialog's Save does; the
reload cron commits and pushes it (git-operations.feature.md).

Autosave is not a publish — a tick never advances a scenario's status.

## Autosave Stands Down While a Delete Is Confirmed @v1 [proposed]

From the moment Delete is pressed in the scenario edit dialog until the
removal is away, no tick fires: a save landing while the confirmation is on
screen would put the scenario back, or re-sort the feature under the entry
being deleted.

## Autosave Ticks Never Overlap @v1 [proposed]

A tick that comes round while the previous save is still in flight does
nothing.

## Autosave Triggered at Defined Interval @v1 [published]

SpecDriven saves the contents of the editor at the defined interval.

## Autosave Waits While the Name Is Being Typed @v1 [proposed]

While the caret is in the dialog's title field the scenario's name is only
half typed, so no tick saves it — a new scenario is not filed as a draft under
the first few letters of its name, and a rename is not saved part-way through.

The tick after focus moves on — to the body, or anywhere else — saves the name
as typed. A published scenario has no editable title, so nothing holds its
ticks back.

## Canceling an Autosaved Edit Restores the Scenario @v1 [proposed]

Cancel undoes the dialog, ticks included. A scenario that was already saved —
a draft the user saved, or a proposed one — goes back to what the dialog
opened it as: the text typed since is taken back out of the feature even
though a tick had saved it.

A rename is taken back with the rest. A renamed draft is deleted under the
name the tick saved it under, because the `.feature.md` never held it, and a
save that leaves a draft out removes nothing.

A dialog no tick ever saved has nothing to restore — Cancel just closes it.

## Canceling Autosaved Scenario Deletes it @v1 [published]

Given: a new scenario was autosaved and is in 'draft' status
When: I Cancel from the scenario editor
Then: This draft scenario is deleted

Note: if this scenario was saved by the user, next Cancel jsut exits the editor dialog.

## Configure Autosave in Settings @v1 [proposed]

Given: the Settings page (user-info-settings.feature.md "open Settings")
When: the user changes "Autosave interval (seconds)" in its Specs section
Then: the new interval is saved to .specdriven/settings.json as soon as the
field is left, with no Save button, and the open editor autosaves at it
- the interval is at least 3 seconds and 6 seconds by default; a smaller
  one typed in the field turns back into the 6 second default, and the
  server refuses one below 3 and changes nothing

This replaces "Configure Autosave in Settings Dialog": Settings is a page in
the editor pane, not a dialog.

[test: the autosave interval defaults to 6 seconds and never goes below 3 : https://github.com/SpecDriven/specdriven_app/blob/main/tests/editorAutosave.test.ts#L56 ]
[test: Settings saves the autosave interval to the settings file : https://github.com/SpecDriven/specdriven_app/blob/main/tests/editorAutosave.test.ts#L80 ]
[test: an interval below the 3 second minimum is refused, and changes nothing : https://github.com/SpecDriven/specdriven_app/blob/main/tests/editorAutosave.test.ts#L89 ]

## Configure Autosave in Settings Dialog @v1 [published]

The SpecDriven App Settings contain a setting for Autosave Interval in Seconds.

The user can edit the Autosave Interval with a minimum of 3 seconds and a default of 6 seconds.
