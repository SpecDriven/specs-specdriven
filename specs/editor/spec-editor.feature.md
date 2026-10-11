# Spec Editor

Spec editor will be separate pane that will replace the feature pane with '<' icon to go back to the feature.

## Adding Scanarios @v1 [proposed]

A small dialog should pop up to enter scenario name.

Scenario name should appear above top left corner of the editor.

## Create New Spec @v1 [published]

Add scenario.
This opens the editor dialog for the new spec.
The new spec editor dialog has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete which removes the draft spec.
The escape key or the close-editor widget will leave the scenario in its autosaved state. Like the Settings dialog, the bottom of the spec dialog will have a note to use Esc to exit.
Two buttons available: "Propose" and "Publish".

## Create New Spec @v2 [proposed]

Add scenario.
This opens the editor dialog for the new spec.
The new spec editor dialog has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete which removes the draft spec.
The escape key or the close-editor widget will leave the scenario in its autosaved state. The bottom of the spec dialog will have a note to use Esc to exit.
Two buttons available: "Propose" and "Publish".
When the spec is proposed or published, the application stores the author who created the spec.

## Delete Published Spec @v1 [published]

Given: deletion of published specs is turned on in user settings
When: click on Delete button
Then: a confirmation dialog pops up with: "Danger Zone! Confirm that you want to delete a Published" spec.
The user can click "Confirm" button or the "Cancel" button.
If confirmed, the spec is deleted.

## Spec body resize @v1 [proposed]

A scenario with a table, a ticket and many test links leaves the body editor
too little room.

- The body editor never shrinks to make room for what is below it: it stays
  at least 40% of the window tall, and the pane's column scrolls instead, so
  the whole pane has a scrollbar once its contents outgrow it.
- The body editor's lower boundary is a grab strip, like the feature
  description editor's (feature-editor.feature.md): dragging it makes the
  body taller or shorter. The height is clamped — never so short that the
  editor is unusable, never so tall that the strip leaves the window.
- Double-clicking the strip restores the default height. The strip is
  focusable, and the up and down arrow keys nudge the height.
- The chosen height is remembered on the device, like the sidebar width — a
  per-device preference, not shared team state — and every scenario opens at
  it.

[test: a height between the bounds is taken as dragged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bodyResize.test.ts#L12 ]
[test: dragging up past the minimum stops at it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bodyResize.test.ts#L16 ]
[test: dragging down stops while the strip is still on the window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bodyResize.test.ts#L21 ]
[test: fractional pixels from the pointer land on whole ones : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bodyResize.test.ts#L25 ]
[test: on a window too short for both bounds the minimum wins, so the body stays usable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/bodyResize.test.ts#L30 ]

## Spec editor pane @v1 [proposed]

The scenario editor is a pane, not a dialog. Opening a scenario — Edit on its
card, Enter on the highlighted card, New version, or "+ Add Scenario" —
replaces the feature pane with the editor in the same slot. The sidebar stays
where it is and keeps working.

- The bar at the top holds, left to right: a `<` back button whose tooltip
  names the feature ("Back to <feature name>"); the scenario title — an
  editable title for a draft or proposed scenario, a read-only heading for a
  published one — with the version and the status badge beside it; and on the
  right, where the feature page keeps Publish All, the actions: Delete, set
  apart from Propose and Publish. A published scenario carries no save action
  at all: the only things it can still change are its links, and those save
  themselves (traceability.feature.md).
- Below the bar one column scrolls: the body's Markdown editor, at least 40%
  of the window tall and growing to fill the pane, then the scenario table,
  the ticket link and the automated tests.
- `<` and ESC go back to the feature, leaving the scenario in its autosaved
  state — one last save catches what was typed since the previous tick, a url
  still sitting in the test-link fields included (traceability.feature.md).
  The scenario's card is highlighted and scrolled into view; a new scenario
  that was never saved highlights nothing.
- There is no "esc to exit" note: the way back is on screen.
- The pane is a page of its own: while it is open the address bar carries the
  scenario's url and the window title names it (navigation.feature.md), so the
  card's "Copy link" shares the page that is on screen, and a link to it opens
  the pane straight away. The feature's url comes back when the pane does.
- The pane is part of the page, not a dialog: nothing behind it is dimmed, it
  is not resizable, and Tab reaches the sidebar — it traps no focus. A delete
  confirmation still opens above it as a dialog, traps Tab and takes the ESC
  while it is up; so do the sidebar's menus and Settings.
- Navigating while the pane is open — a feature in the sidebar, a search
  result, the data folder name, browser Back — first saves what the pane
  holds, exactly like `<`, then navigates.
- The reload cron and Sync leave an open pane alone, as they did the dialog.
