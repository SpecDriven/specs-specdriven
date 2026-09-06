# Tests

**Assigned:** David

A lightweight test management module for a small team, in the spirit of
TestRail and Xray: a library of test cases, test plans that group them, and
executions that record one result per case of a plan. Test cases may
reference features or scenarios. The module takes an entire pane, and all of
its data is stored in an Automerge document — no markdown files.

## A run records one result per case @v1 [proposed]

- Given: a plan with at least one case
- When: the user presses "Start run"
- Then: a run named "Run N — <date>" opens, holding the plan's cases as they
  stand at that moment — cases added to or removed from the plan later leave
  the run alone
- and every case starts untested; each row offers the verdicts untested,
  passed, failed, blocked and skipped, a comment field, and shows who
  recorded the result and when
- and with a row focused, the keys `p` `f` `b` `s` `u` record the verdict
- and the run's progress bar and counts follow — "1/3 run · 1 passed"
- and "Finish run" closes it, asking first when cases are still untested;
  a finished run is read-only until "Reopen"
- and a run outlives its plan: deleting the plan keeps its runs, listed
  under a deleted plan, while deleting a run takes its results with it

## A test case may reference features or scenarios @v1 [proposed]

- Given: the Tests tab is open
- When: the user creates a test case with "+ New case" and gives it a title
- Then: the case opens in the pane with a priority chip (low, medium or
  high; medium by default), a "Covers" row and a Markdown body for
  preconditions, steps and expected results
- and "Add reference" offers every feature of the data folder, each
  unfolding to its scenarios with their status glyphs, with a filter field;
  ticking a feature or a scenario adds it as a chip, which links to the
  feature page or the scenario's own page; a chip's ✕ removes it
- and a case may carry many references, or none
- and the body autosaves on the editor's interval, the way a roadmap page
  does, with the same Saved / Unsaved indicator
- and the case library at `/tests/cases` lists every case with its priority,
  what it covers, and its latest result

## A test plan groups test cases @v1 [proposed]

- Given: some test cases exist
- When: the user creates a plan with "+ New plan" and names it
- Then: the plan opens in the pane with a Markdown description (autosaved),
  the "Assigned" chip when assignments are on, and an empty case table
- and "Add cases" lists the library's cases not yet in the plan; a click
  adds one, and "+ New case in this plan" creates one and adds it
- and the cases keep the order they were added in; ↑ ↓ reorder them and ✕
  removes one from the plan without deleting it from the library
- and deleting a case from the library removes it from every plan
- and the plan list at `/tests` shows each plan's case count, assignee,
  latest run and that run's progress, and the ten most recent runs of all
  plans below

## Test data lives in one Automerge document, not in markdown @v1 [proposed]

- Given: the setting is on
- When: the first test case or plan is created in a data folder
- Then: the module's document is minted under
  `.specdriven/automerge/testing/project/` — a `docid`, a snapshot and
  change chunks — and no `.md` file is written anywhere
- and the sync tick commits and pushes those chunks like any other saved
  change
- and merely opening the Tests tab in a folder that has no test data writes
  nothing
- and two people who each start using the module before either has pushed
  still end up with one document: the document's first change is identical
  on every machine, so their chunks merge on the next pull instead of
  forking into two lineages

## Test management is controlled by a switch in user settings @v1 [proposed]

- Given: the "Test management" setting is off, as it is by default
- When: the user looks at the sidebar
- Then: there is no Tests tab, and nothing in the app asks for test data
- When: the user turns the setting on
- Then: a Tests tab appears beside Specs (and Roadmap and Tasks, when those
  are on), with a Tests link in the collapsed rail and `g e` as its shortcut
- and turning it off again hides the tab; a tests view that is open leaves
  the pane and the app goes home. Test data already recorded stays in the
  data folder untouched
- and the server refuses the module's requests while the switch is off, not
  just the UI

## Tests views have urls and find their way into search @v1 [proposed]

- Given: the setting is on
- When: the user opens `/tests`, `/tests/cases`, `/tests/cases/<id>`,
  `/tests/plans/<id>` or `/tests/runs/<id>`
- Then: that view opens with the sidebar on the Tests tab, the window title
  names it, and Back / Forward move between views; an id that names nothing
  shows a note with a way back to the list
- and every row's ⋯ menu offers Rename, Copy link and Delete, the way
  roadmap pages do
- and the command palette opened from the Tests tab searches plans, cases
  (title and body) and runs by name, in that order, and opens the one picked
- and the reload tick refreshes the module from the data folder, except
  while a case or plan editor holds an unsaved draft

## The latest run speaks for a case @v1 [proposed]

- Given: a plan run more than once
- When: the user looks at the plan, the case library or a case
- Then: each case shows the result from the most recent run that included
  it — a case left out of the latest run keeps the verdict of the run before
- and the plan's row and the sidebar's plan entry show the latest run's
  progress
- and a case's page names the plans it is in and its latest result, linking
  to that run
