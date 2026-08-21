# Feature CRUD

Feature files are markdown files that we version in git.

Feature files contain atomic, immutable, and versioned specifications.

## Feature file format

Feature files are plain markdown, named `<slug>.feature.md`, so they render
nicely on GitHub and in any markdown viewer:

```markdown
# <name of the feature>

<optional feature description — any markdown>

## <name of scenario> @vNNN [proposed | published]

<scenario body — any markdown up to the next scenario heading>
```

The `@` prefixes the version so the scenario name and its version are easy to
tell apart, by eye and by parser — a name that itself ends in a version
(`## Migrate to v2 API @v1`) stays unambiguous. Headings written before the
prefix (`## Basic Quote v1 [published]`) are still read, and are rewritten with
the `@` the next time the feature is saved.

## Scenario status

A scenario has one status: `draft`, `proposed`, or `published`, advancing one
way in that order. Only `[proposed]` and `[published]` appear in feature
files, as chips after the version. `draft` is an internal status: a draft
scenario is stored in the feature's Automerge document only
(automerge-store.feature.md) and is never projected into the
`.feature.md`. A heading without a chip parses as a draft — save marks it
`[proposed]`, and a chipless scenario hand-added to the markdown is absorbed
into the document as a draft and leaves the file on the next sync tick.

A scenario created in the app starts as a `draft`. The editor autosaves the
open dialog (editor-autosave.feature.md), so anything else would put a
half-written scenario into the `.feature.md` without the user asking for it;
moving to `proposed` is the user's explicit action.

The scenario edit dialog has a `Draft` checkbox (ui.feature.md "Scenario
edit dialog"). A scenario saved with it checked keeps the `draft` status: the
editor and the server exchange it with an explicit `[draft]` chip — a wire
format that never lands in a `.feature.md` — and save stores it in the
document without writing it to the file. The editor keeps showing drafts:
reading a feature folds the document's drafts back in. `Publish All` leaves
drafts alone; publishing a draft from its own card is the explicit action
that promotes it into the file.

```markdown
## Basic Quote @v1 [proposed]

- **Given** a potential customer has an account
- **When** the customer requests a quote
- **Then** a quote is provided

| plan   | mobile UI |
| ------ | --------- |
| Gold   | yes       |
| Silver | yes       |
```

Feature files may live in folders — plain directories in the data repo
(folders.feature.md).

## Publishing scenarios

When a scenario is published it is marked with a `` [published] `` chip after
the version.

Ex:

```markdown
## Basic Quote @v1 [published]
```

## create new feature @v1 [proposed]

- Given: user is verified to have access to the data GH repo
- When: user asks to create a feature
- Then: new feature file is created in the local filesystem

## delete feature @v1 [proposed]

- from the feature entry's drop down menu in the left pane: Delete
- asks for confirmation first
- the file is removed with `git rm`, committed and pushed

## Deleting a draft targets one version @v1 [proposed]

A draft is identified by its name and its version, so two drafts under one
name are two scenarios and only the targeted one is deleted.

Asking to delete a draft that is not in the document is an error, and nothing
changes. The operation is for drafts only — a `[proposed]` scenario is deleted
by saving the feature without it (ui.feature.md "Scenario edit dialog").

## Headings without a version are plain content @v1 [proposed]

Only a `##` heading that ends with a version (`@vNNN`) starts a scenario.

- A `##` heading without a version is ordinary markdown — it stays part of
  the feature description or of the scenario body it appears in, and is
  rendered as a heading there, not treated as a scenario boundary. (This
  very file relies on it: "Feature file format" and "Scenario status" are
  sections, not scenarios.)
- Headings inside fenced code blocks (``` or ~~~) are content too, even
  when they carry a version — examples of the file format must not parse
  as real scenarios.

## publish each scenario individually; support Publish All @v1 [proposed]

- PM should be able to publish each scenario individually
- Upper right button will be Publish All
- for each 'proposed' scenario, links are 'Edit', 'Publish', 'Copy link' (in that order)

## Publish feature with merge conflict @v1 [proposed]

- Display conflict to the user and ask for a resolution
- Either accept yours, accept theirs, or manual edit
- do a git commit and push

## Publish feature with successful merge @v1 [proposed]

- Do a 'git pull' and merge all the files.
- If merge successful, proceed with saving
- do a git commit and push

## reload data on a cron job @v1 [proposed]

- automatically reload (git pull) data every n seconds
- specify reload frequency in the Settings screen

## rename feature file @v1 [proposed]

- from the feature entry's drop down menu in the left pane: File → Rename
- only the stem (left of the dot) is edited; the `.feature.md` extension and
  the folder are kept
- the rename is a `git mv`, committed and pushed
- links to the old file name go stale

## Scenario is automatically saved locally and pushed to GitHub @v1 [proposed]

- We don't want to loose any data.
- Based on data reload frequency in Settings
- after Git pull; do a commit and push of the saved scenarios; on the top banner of the UI,
- display 'Saved' message for 2 sec

Ex:

```markdown
## Customer calls accident hotline and talks to a live agent @v1 [proposed]

Customer calls accident hotline and leaves a message with accident info.
```
