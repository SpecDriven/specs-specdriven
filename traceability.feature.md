# Store links to automated tests

- store links as github urls.
- Ex: `https://github.com/vydra/GatlingGen/blob/main/app/src/test/java/ggen/GatlingCodeGeneratorTest.java`
- Display in a table; links are clickable
- Display at the end of each scenario
- These links are optional and will be controlled by user setting; default is ON

The links live inside the feature file, as a trailing `### Tests` markdown
table in the scenario — like the scenario table (scenario-table.feature.md),
plain markdown that renders naturally on GitHub:

```
### Tests

| Test |
| --- |
| https://github.com/vydra/GatlingGen/blob/main/app/src/test/java/ggen/GatlingCodeGeneratorTest.java |
```

- The parser lifts the urls out of that section; the scenario card shows them
  as an "Automated tests" table at the end, each row a clickable link opening
  in a new tab. The link text is the url without the `https://github.com/`
  prefix.
- Only GitHub urls (`https://github.com/…`) are accepted.
- The scenario editor pane has an "Automated tests" field: the current links,
  each with an ✎ to correct it in place and a remove ✕, plus an add field.
  A published scenario is immutable, but its test links are not part of the
  spec — they keep changing as the tests get written — so the field stays
  editable there too. Nothing else about a published scenario is: its name,
  version and body stay read-only.
- Because of that the `[published]` card's link reads `Edit`, never `View`:
  the pane it opens always has something to offer, if only the ticket link
  (external-links/project-management-jira-linear-etc.feature.md).
- A url is rejected — Add and the ✎ row's Save stay disabled — unless it is a
  GitHub url the scenario does not already list.
- The links need no save action of their own: autosave writes them out
  locally like any other edit (editor/editor-autosave.feature.md) and the
  reload cron or Sync pushes them, which is why a published scenario's pane
  has no save button (editor/spec-editor.feature.md).
- Leaving the pane with a url still sitting in the add field or in an open ✎
  row folds it into that last save as if Add (or the row's Save) had been
  pressed — it is never silently dropped. Propose and Publish fold it in the
  same way, and there a pending url that can't join the list blocks the save
  with a message under the field; on the way out of the pane there is nowhere
  left to show that message, so such a url is left behind rather than holding
  the pane open.
- The Settings dialog has a "Show links to automated tests" checkbox; the
  choice is saved to the settings file (`showTestLinks`), and only hides the
  tables on the cards and the Automated tests field in the editor pane —
  the stored links are untouched.

## add a test link @v1 [proposed]

- When: the user pastes a GitHub url into the editor pane's Automated tests
  field and presses Add
- Then: the link joins the scenario's list, and the next autosave writes it
  into the scenario's `### Tests` table in the feature file

## correct a test link @v1 [proposed]

- When: the user presses ✎ on a link, edits the url and presses Save
- Then: that row holds the new url, and the others are untouched

## edit the tests of a published scenario @v1 [proposed]

- Given: "Show links to automated tests" is ON (the default)
- When: the user opens a `[published]` scenario through its card's `Edit` link
- Then: the pane's Automated tests field takes adds, ✎ edits and ✕ removes,
  and they reach the feature file with no save action pressed — autosaved
  locally, pushed on Sync — while the name, version and body stay read-only

## hide the test links @v1 [proposed]

- When: the user turns "Show links to automated tests" off in Settings
- Then: the tables disappear from the scenario cards and the Automated tests
  field disappears from the editor pane, but the links stay in the
  feature files
- And: a `[published]` scenario keeps its `Edit` link — the pane still holds
  its ticket link — and that pane offers no save action and no Close: `<` and
  ESC are the way back (editor/spec-editor.feature.md)

## see the tests of a scenario @v1 [proposed]

- When: a scenario has test links and the setting is ON (the default)
- Then: the end of the scenario card shows a table of clickable links

## the add field still holds a url when the pane is left @v1 [proposed]

- When: the user pastes a GitHub url into the Automated tests add field and
  leaves the pane — `<`, ESC or navigating away — without pressing Add
- Then: the link is saved all the same, by the save that leaving makes
- And: an invalid or duplicate url there is left behind rather than saved,
  since the pane is closing and there is nowhere to report it
- And: pressing Propose or Publish instead folds the pending url in the same
  way, but an invalid or duplicate one blocks that save with a message under
  the field instead of being dropped
