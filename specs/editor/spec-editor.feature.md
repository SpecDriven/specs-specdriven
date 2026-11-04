# Spec Editor

**Created by:** jmeowmeow

**Updated by:** vydra

Spec editor will be separate pane that will replace the feature pane with '<' icon to go back to the feature.

## S1: Adding Scanarios @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

A small dialog should pop up to enter scenario name.

Scenario name should appear above top left corner of the editor.

## S2: Create New Spec @v1 [published]

**Created by:** jmeowmeow

**Updated by:** jmeowmeow

Add scenario.
This opens the editor dialog for the new spec.
The new spec editor dialog has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete which removes the draft spec.
The escape key or the close-editor widget will leave the scenario in its autosaved state. Like the Settings dialog, the bottom of the spec dialog will have a note to use Esc to exit.
Two buttons available: "Propose" and "Publish".

## S3: Create New Spec @v2 [proposed]

**Created by:** RomanWright

**Updated by:** vydra

Add scenario.
This opens the editor dialog for the new spec.
The new spec editor dialog has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete which removes the draft spec.
The escape key or the close-editor widget will leave the scenario in its autosaved state. The bottom of the spec dialog will have a note to use Esc to exit.
Two buttons available: "Propose" and "Publish".
When the spec is proposed or published, the application stores the author who created the spec.

## S4: Create New Spec @v3 [proposed]

Add scenario.
This opens the editor pane for the new spec.
The new spec editor pane has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete, in the pane's `⋯` menu, which removes the draft spec.
The escape key or the feature crumb will leave the scenario in its autosaved state.
Propose and Publish are picked from the status chip beside the title; the status moves one way.
When the spec is proposed or published, the application stores the author who created the spec.

[test: a new draft is persisted in the Automerge document, out of the file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditor.test.ts#L35 ]
[test: Delete removes the draft spec : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditor.test.ts#L47 ]
[test: Propose lands the draft in the feature file as proposed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditor.test.ts#L63 ]
[test: Publish promotes a draft scenario straight to published : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditor.test.ts#L77 ]
[test: new spec has an author — a scenario added by a save is created by whoever saved : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsAuthorship.test.ts#L123 ]

## S5: Delete Published Spec @v1 [published]

**Created by:** vydra

**Updated by:** vydra

Given: deletion of published specs is turned on in user settings
When: click on Delete button
Then: a confirmation dialog pops up with: "Danger Zone! Confirm that you want to delete a Published" spec.
The user can click "Confirm" button or the "Cancel" button.
If confirmed, the spec is deleted.

## S6: Previous and next scenario @v1 [proposed]

From one scenario's editor, step to the scenario before or after it without
going back to the feature.

- The bar's actions start with two icon buttons, `‹` (previous scenario) and
  `›` (next scenario), ahead of the bar's other actions.
- They walk every scenario in the order the sidebar tree lists them —
  folders before features, features by name, each feature's scenarios in
  file order — crossing from the last scenario of one feature into the first
  of the next. The open feature's scenarios come from what it holds now, so
  a draft is part of the walk (navigation.feature.md "Scenario navigation").
- The keys are `<` and `>`, and `k` and `j` as on the feature page, while the
  caret is not in a text field. The keyboard cheat sheet and the command
  palette list them.
- Each button's tooltip names the scenario it opens, with its version.
- Stepping leaves the open scenario the way every navigation out of the pane
  does: it saves what the pane holds first, then opens the other scenario's
  page, whose url and window title follow.
- At the first scenario `‹` is disabled, at the last `›` is. A scenario
  is found by its name and version as last saved, so a different version of
  the same name is a different stop. A new scenario that has not been saved
  yet is not in the walk, and both buttons are disabled until it is.

[test: the walk follows the tree — folders first, then features by name, each in file order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L175 ]
[test: the open feature contributes its live scenarios, drafts included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L187 ]
[test: the neighbours cross from one feature into the next : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L192 ]
[test: the first scenario has no previous, the last no next : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L198 ]
[test: a scenario that isn't in the tree yet — new, never saved — has neither : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L203 ]
[test: a scenario is told apart from its other versions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L210 ]

## S7: Spec body resize @v1 [proposed]

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

## S8: Spec editor pane @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

The scenario editor is a pane, not a dialog. Opening a scenario — Edit on its
card, Enter on the highlighted card, New version, or the header's `+` —
replaces the feature pane with the editor in the same slot. The sidebar stays
where it is and keeps working.

- The bar at the top holds, left to right: the breadcrumb trail, whose last
  crumb is the feature and the way back ("Back to <feature name>"); and on
  the right the actions as icon buttons, all one size — previous / next
  scenario, copy link, `+` for a new scenario after this one (key `c`: the
  pane saves first, then the new draft opens right after where that save
  landed the scenario), the save indicator, and a `⋯` menu with New scenario
  below, Clone, New version (published only), Copy link and, last and
  red, Delete (where the scenario can be deleted). No filled button sits in
  the bar.
- Under the bar, beside the title — editable for a draft or proposed
  scenario, a read-only heading for a published one — and the version, the
  status is a chip: a draft's opens a menu with Propose and Publish, a
  proposed scenario's with Publish only, and a published scenario's is a
  plain badge — or, while Settings allows deleting published specs, a menu
  with Back to proposed ("Unpublish a Spec"). `⌘⏎` picks the first of them. A published scenario carries no
  save action at all: the only things it can still change are its links, and
  those save themselves (traceability.feature.md).
- Below the bar one column scrolls: the body's Markdown editor, at least 40%
  of the window tall and growing to fill the pane, then the ticket link and
  the automated tests.
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

[test: shows its editor, not the error note : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioEditorDraft.test.tsx#L140 ]
[test: insertAfterScenario lands right after the pane's scenario, wherever its save left it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioOrderClient.test.tsx#L62 ]
[test: single letters stand down in a contenteditable body and behind a false `when` : https://github.com/SpecDriven/specdriven-app/blob/main/tests/shortcuts.test.tsx#L125 ]
[test: an opened, unedited scenario's card is where it was : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditorPane.test.tsx#L26 ]
[test: a new scenario is appended (or placed at insertAt) and its card is found there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditorPane.test.tsx#L34 ]
[test: a scenario the feature never received highlights nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specEditorPane.test.tsx#L62 ]
[test: with the setting off, it is a plain badge : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublishChip.test.tsx#L167 ]

## S9: Unpublish a Spec @v1 [proposed]

Given: deletion of published specs is turned on in user settings
When: in a published scenario's editor pane, open the status chip and pick
Back to proposed
Then: the scenario is [proposed] again — same name, version, place, body,
tests and links — and its body and title are editable in the pane. The change
is committed and pushed right away, as a Delete Published Spec is. Publish
takes it back to [published].

With the setting off, the published status chip is a plain badge, and the
server refuses the change too.

[test: with the setting off, the unpublish route refuses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublish.test.ts#L71 ]
[test: with the setting on, a published spec moves back to proposed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublish.test.ts#L80 ]
[test: an unpublished spec is editable again, and Publish takes it back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublish.test.ts#L96 ]
[test: only a published scenario can be unpublished : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublish.test.ts#L106 ]
[test: an unpublished spec keeps its tests and links, and the change is committed and pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublish.test.ts#L114 ]
[test: with the setting on, Back to proposed moves the scenario back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublishChip.test.tsx#L149 ]
[test: with the setting off, it is a plain badge : https://github.com/SpecDriven/specdriven-app/blob/main/tests/unpublishChip.test.tsx#L167 ]
