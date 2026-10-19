# UI

- Feature page lists scenarios, up to 8 lines per scenario.
- Scenarios are ordered by name, then by version.
- Published scenarios are shaded differently from un-published, editable scenarios.

Edit scenarios in the editor pane (editor/spec-editor.feature.md "Spec editor
pane").

If a scenario is already published, then it is immutable — apart from its
links to automated tests, which stay editable (traceability.feature.md).

## Main page

- Top of page should say 'Scenario: <scenario name>' Ex: `Feature: Quote Policy`
- Left side should also use abbreviated scenario name, not file name.
- We don't need to display the file name.
- Add `Edit` link next to the feature name which allows editing the feature
  name — the file's `# <name>` heading — not the file name.
  - the edited name is committed and pushed to GitHub
  - the file name stays unchanged, so links to the feature keep working

Sync button (immediate data reload), disabled with a progress label while
running. No "Syncing…"/"Synced" status update at the bottom of the page.

Publish button: same style and behavior as the Sync button (disabled with a
progress label while running). No "Published" status update at the bottom of
the page.

## Scenario List

- render scenario body as Markdown preview; up to n lines
- make the number of Markdown lines to render for each scenario in list configurable in Settings

## Feature entry menu

Each feature entry in the left pane has a drop down menu (a `⋯` button,
visible on hover) with three items: `File`, `Move to folder…` and `Delete`.

- `File` opens a submenu with `Rename`, which renames the underlying file.
  - Only the file name to the left of the dot is shown and edited; the
    `.feature.md` extension is hidden and always kept. The folder is kept too.
  - The rename is a `git mv`, committed and pushed.
  - The feature's url follows the file name, so links to the old name go stale.
  - The feature name (the `# <name>` heading) is untouched — renaming that is
    the Edit link on the main page.
- `Move to folder…` offers the list of folders and moves the file into the
  chosen one — the same `git mv` as dragging it there (folders.feature.md).
- `Delete` deletes the feature file after a confirmation — a `git rm`,
  committed and pushed.
- ESC or a click anywhere else closes the menu.
- Folder rows carry a `⋯` menu of their own — folders.feature.md.

## Folders

Features can be organized into folders — see folders.feature.md.

## Sidebar resize

A deep folder hierarchy needs more room than the default left-pane width.

- The divider between the left pane and the editor is draggable; dragging it
  resizes the left pane.
- The width is clamped: the pane can never collapse away nor swallow the
  editor.
- Double-clicking the divider restores the default width. The divider is
  focusable, and arrow keys nudge the width.
- The chosen width is remembered on the device (browser/app storage), like
  the theme — a per-device preference, not shared team state.

## Select scenarios via up and down keys

- Highlight selected scenario
- Edit on Enter key press

## Dialogs

Applies to every dialog (Settings, the merge conflict dialog, prompts and
confirmations, and any future dialog).

- While a dialog is in the foreground, Tab and Shift+Tab must only move focus
  between the fields inside that dialog. Focus wraps from the last field back to
  the first, and from the first back to the last.
- Focus must never reach elements behind the dialog while it is open.
- On open, focus moves into the dialog; on close, focus returns to the element
  that opened it.
- Native browser prompts and confirms are never used — creating a feature or
  folder, renaming a file, and every delete confirmation are in-app dialogs.
  (The desktop runtime has no `prompt()` at all —
  desktop-app.feature.md.) Prompts submit on Enter and can't submit
  empty; delete confirmations carry a solid-red destructive button.
- Dialogs can stack — a delete confirmation above the scenario editor
  pane, or above another dialog.
  Tab cycles only the top dialog, and ESC closes only the top dialog.

## Scenario editor

> Superseded for the editor's chrome by editor/spec-editor.feature.md "Spec
> editor pane": the editor is a pane in the feature's slot, not a dialog — not
> resizable, no "esc to exit" note. The field rules below (title, version,
> Draft, Delete, published read-only, ESC in a ✎ row) still apply.

- Name must be the title of the pane
- Must be resizable
- Scenario body must contain an embedded Markdown Editor
- Below the body, a table widget edits the scenario's markdown table
  (scenario-table.feature.md)
- ESC key to get back to Scenario screen.
- The version is never user editable; it is shown read-only and assigned by
  the app.
- A new scenario takes the next version for the name it is given: one past the
  highest version already carrying that name, or v1 when the name is new.
  Retyping the name in the pane re-derives the version.
- Editing an existing scenario keeps the version it was already assigned.
- A `Draft` checkbox marks the scenario a draft: saved in the app but kept out
  of the `.feature.md` (crud-feature.feature.md "Scenario status"). It is
  editable for a new scenario and for an existing draft; the status moves one
  way, so once a scenario has been saved as proposed the checkbox is disabled.
- A new scenario — including the next version of a published one — starts with
  `Draft` checked. Autosave (editor-autosave.feature.md) writes the open pane
  out on its own, and only the user unchecking the box proposes the scenario
  into the `.feature.md`.
- Draft scenarios show a `draft` badge on their cards.
- A `[proposed]` scenario has a `Delete` button: after a confirmation, the
  scenario is removed from the feature. The removal is saved like any edit —
  auto-saved locally and pushed by the reload cron.
- A draft has the same `Delete` button, so a scenario autosave has placed can
  always be taken back out. A draft is deleted from the app's document rather
  than by saving the feature without it (automerge-store.feature.md), because
  the `.feature.md` never held it; the reload cron pushes that deletion like
  any other save. Delete appears once the scenario is part of the feature —
  a pane on a scenario that has never been saved has nothing to delete, and
  `<` or ESC is the way out of it.
- Published scenarios have no Delete button — they are immutable. Their name,
  version, body and scenario table are read-only; only the Automated tests
  field can be changed (traceability.feature.md), and there is no save button
  at all — those links are autosaved locally and pushed on Sync.
- ESC inside a field that is editing one row in place (the ✎ on a test link)
  backs out of that row; a second ESC leaves the pane.

## Merge conflict dialog

A publish that hits a merge conflict (crud-feature.feature.md "Publish
feature with merge conflict") opens a dialog — not a strip appended to the
feature page, where a long feature pushes the resolutions out of sight.

- The dialog is sized to the window and resizable; it never grows with the
  feature.
- `Yours` and `Theirs` sit side by side, each scrolling in its own pane. A
  file that does not exist on the remote says so instead of showing an
  empty pane.
- Below them, a `Manual edit` box, pre-filled with yours, that can also be
  reset to either side.
- The three resolutions — `Accept yours`, `Accept theirs`, `Publish manual
  edit` — are pinned at the bottom and always visible, no matter how long
  the two sides are. They disable with a progress label while one runs.
- `Cancel`, ESC, or a click outside closes the dialog without resolving. The
  unfinished merge stays in the repo (git-operations.feature.md), so
  Publish brings the same choices back.

## Settings

A settings page in the editor pane, at `/settings` — not a dialog, which was
too small (user-info-settings.feature.md "open Settings"). Its sections are
listed on the left: General, Sync, Specs, Project management, Workflow and
Git & GitHub. Settings are saved to the settings file
(`.specdriven/settings.json`, app-start.feature.md) the moment a control
changes; there is no Save button. The plugins have their own page
(app/plugins.feature.md).

- Autosync: a sliding switch, on by default, with the autosync frequency in
  seconds below it while it is on. Off stores a frequency of 0, which stops
  the automatic pull/push.
- Appearance: System, Light or Dark. Applied immediately and remembered on
  the device (browser/app storage), not in the settings file — theme is a
  per-device preference, not shared team state. "System" follows the OS.
- ESC or the ✕ in the page header goes back to what was open before.

## Design system

Warm, minimal, Claude-like. All colors flow through CSS tokens in
`src/frontend/index.css` — components never hardcode a color, so both themes
come from one token block.

- Surfaces: ivory app canvas, oat sidebar, white cards (light); warm
  charcoal equivalents (dark).
- Accent: terracotta. Primary buttons are filled terracotta; secondary
  buttons are quiet outlines; sidebar actions stay muted.
- Type: serif display faces for feature/scenario/dialog titles (Source
  Serif 4), Inter for UI text, JetBrains Mono for code and file names. The
  fonts ship with the app — no network fetch, so the desktop app renders
  identically offline.
- Published scenarios keep their distinct shading: a parchment wash.
- Focus is always visible: accent rings on fields, focus outlines on
  controls.

## Add A Scenario Button @v1 [published]

* With a feature file editor open, the button to add a scenario should always be visible. The button should be located with respect to the editor window viewport.

## Feature Text Scroll Pane @v1 [published]

The detail view for a feature file should provide a scroll pane for the feature text, rendering Markdown to HTML, and providing specd for the scenario cards and the "add a scenario" button.

## Feature Text Scroll Pane @v2 [published]

The free markdown between the feature heading and the first scenario (the
feature description) renders as Markdown — same pipeline as scenario bodies —
in its own pane on the feature page:

- The pane has its own scrollbar and scrolls independently of the scenario
  list.
- The pane is never taller than a third of the screen; a longer description
  scrolls inside the pane instead of pushing the scenario list off screen.
- A short description hugs its content — a one-line description does not
  reserve a third of the screen.
- The scenario list keeps its own scroll pane below; a tall scenario list
  never squeezes the description pane, and the "add a scenario" button stays
  reachable.

## Hide the editor pane @v1 [proposed]

While the coding agent pane is open beside it, the editor column can be hidden
so the pane has the window — the same kind of control the agent pane itself
has, one column to its left.

- The icon sits just left of the coding agent's, in the sidebar header and in
  the collapsed rail, so the two toggles read in the order their panes do —
  editor, then agent pane. It toggles on `\` — the key between `[` and `]`,
  which hide the sidebar and the agent pane. It is in the command palette too,
  listed ahead of the coding agent's commands.
- It is offered only while the coding agent pane is showing as a column. With
  that pane closed there is nothing to give the room to, and on a one-pane
  screen the pane already covers the page — so the icon is absent in both
  cases and the editor always shows.
- Hiding the editor drops its column entirely: the sidebar keeps its width and
  the agent pane takes the rest of the window, past the width limit that
  normally keeps it clear of the editor. The divider between them goes with
  the editor, since there is no longer anything to size against it.
- The choice is remembered on the device (browser/app storage), like the
  sidebar width and the theme — a per-device preference, not shared team
  state. Closing the agent pane brings the editor back without forgetting the
  choice: reopening the pane hides it again.

## Recent folder list @v1 [published]

There is a small down arrow icon next to folder name that brings up a list of previous folders. Support up to 10 folders.

## Spec will be added under the folder that is highlighted @v1 [published]

When adding a spec (scenario), it must be added under the highlighted (focused) folder.

## User-entered text is cleaned up @v1 [proposed]

Whatever the user types is saved without the whitespace they did not mean —
a pasted title's trailing blank, a double space, the empty lines around a
message — in every field, dialog and prompt, and by the server too, so an
agent or plugin calling the API gets the same.

- When: the user saves a one-line value — a feature, folder, scenario, page,
  issue, test plan, case or run name, a title, a session name, a tool name,
  a test-run note
- Then: it is saved trimmed, with every inner run of spaces, tabs or line
  breaks collapsed to one space: `"  Quote   Policy "` is saved as
  `Quote Policy`
- When: the user saves multi-line text — an issue body or comment, a test
  case body, a plan description, a message to the coding agent
- Then: the blank lines before and after it and the trailing spaces are
  dropped, but its indentation and everything inside it stay as typed; text
  that turns out to be one line is trimmed whole
- Paths and urls — a data directory, a plugin folder, a marketplace or ticket
  url — are only trimmed: a folder name may hold a double space.
- Whitespace alone counts as empty: a required field refuses it, an optional
  one is left blank.
- A published scenario keeps its heading exactly as published, since it is
  immutable; names are still compared cleaned, so it is found under either
  spelling.
- Whole documents — the README, a page, a scenario's body — are saved as
  written (a scenario body's surrounding blank lines already go when the
  feature file is parsed), and a Markdown field being autosaved never loses
  the newline just typed under the caret.

[test: loses the whitespace around it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/text.test.ts#L7 ]
[test: keeps one space for every inner run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/text.test.ts#L12 ]
[test: keeps the first line's indentation and everything inside : https://github.com/SpecDriven/specdriven-app/blob/main/tests/text.test.ts#L31 ]
[test: that is one line after all is trimmed whole : https://github.com/SpecDriven/specdriven-app/blob/main/tests/text.test.ts#L40 ]
[test: a feature's typed name is saved cleaned : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L131 ]
[test: a published scenario keeps its heading verbatim : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioRenameLoop.test.ts#L130 ]
[test: a plugin folder and a trusted folder are saved trimmed, inner spaces kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsWhitespace.test.ts#L53 ]
