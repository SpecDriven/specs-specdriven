# specs-authorship

We would like to store the user names of the creators of each atomic spec (scenario).
Creator maybe an agent who was given permission via a GH token.
We would also like to store the user name of the last person who updated the spec.

Add a setting scoped per specs repo as to whether store author info in the feature markdown file.

The Automerge document of every feature (datastore/automerge-store.feature.md)
keeps a `createdBy` and an `updatedBy` on the feature and on each of its
scenarios. The name is the identity the app works as — the GitHub login it is
signed in with (an agent given its own token signs as that account), else the
git user name, else the OS user — the same identity that authors the app's
commits and signs notebook entries (app/engineering-notebook.feature.md).

## A rename keeps the creator @v1 [published]

- Given: a scenario created by ada
- When: grace renames it in the app
- Then: ada is still its creator and grace is its last updater
- and a new version of a published scenario is a new scenario, created by
  whoever proposed it

[test: a rename keeps the scenario's creator : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L156 ]

## A save that changes nothing signs nothing @v1 [published]

- Given: a feature last updated by ada
- When: grace saves it without changing anything
- Then: ada is still the last updater of the feature and of every scenario

[test: a save that changes nothing signs nothing : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L149 ]

## An agent signs as its own account @v1 [published]

- Given: an agent works in the app with a GitHub token of its own
- When: it creates or edits a feature or a scenario
- Then: the record names the agent's GitHub login, not the person who
  started it
- and without a GitHub sign-in the git user name signs, else the OS user

[test: a feature created in the app is signed by the app's user, in the document only by default : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L249 ]

## Authorship lines in the markdown @v1 [published]

- Given: a `.feature.md` with `**Created by:** <name>` and
  `**Updated by:** <name>` lines directly under the feature's `#` heading or
  a scenario's `##` heading (after any external link line)
- When: the file is parsed
- Then: the lines are read as authorship and are not part of the
  description or the scenario's body
- and the same line further down a body is ordinary content

[test: authorship lines under a heading are lifted out of the description and the body : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L64 ]
[test: a line further down the body is ordinary content : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L99 ]

## Edits found on disk sign nobody @v1 [published]

- Given: a `.feature.md` changed outside the app — on GitHub, in an editor,
  by an older app version
- When: the sync tick absorbs it
- Then: nobody is stamped as its updater, since the app cannot know who
  wrote it
- and authorship lines in the file only fill in what the document never
  recorded; a stale line never moves an existing creator or last updater
- and a file found on disk for the first time that names its authors gives
  the new document them

[test: drift absorbed without an author stamps nobody, and stale lines never rewrite the record : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L166 ]
[test: a file found on disk that names its authors gives the document them : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L181 ]

## Merging keeps the side that changed the authorship @v1 [published]

- Given: ada and grace both have the feature, and grace edits a scenario
- When: the three-way merge of a pull folds grace's edit into ada's copy
- Then: the scenario's and the feature's last updater are grace

[test: the three-way merge keeps the side that changed the authorship : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L203 ]

## New feature has author @v1 [published]

save author at the feature level

- Given: the user creates a feature in the app
- Then: the feature records the user as its creator and last updater
- and every scenario it starts with records the user as its creator
- and the feature's last updater moves whenever anything in it changes —
  its name, description, links, assignee, or any scenario added, edited or
  removed

[test: New feature has an author — an import signed by its writer records it at the feature level : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L115 ]
[test: a feature created in the app is signed by the app's user, in the document only by default : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L249 ]

## new spec has an author @v1 [published]

New spec has an author

- Given: a feature created by ada
- When: grace adds a scenario and saves
- Then: the new scenario is created and last updated by grace
- and the scenarios grace did not touch keep their own creator and updater
- and the feature keeps ada as its creator, with grace as its last updater

[test: new spec has an author — a scenario added by a save is created by whoever saved : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L122 ]

## The feature page shows who wrote what @v1 [published]

- Given: a feature with recorded authorship
- When: the user opens it
- Then: an authorship chip in the header reads `Created by ada · updated by
  grace`, or `Created and last updated by ada` when they are the same
- and each scenario card carries the same line for that scenario
- and a feature made before the app kept authorship shows no chip until
  somebody edits it

[test: the page reads authorship as one short line : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L290 ]

## The last updater is whoever changed it @v1 [published]

- Given: a scenario created by ada
- When: grace edits its body, name, version, status, tests or link
- Then: grace is its last updater and ada is still its creator

[test: an edit records its author as the last updater and keeps the creator : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L140 ]

## Write authors into feature files @v1 [published]

- Given: the data repo's `project-settings.json`
- When: it has `"authors-in-markdown": true` — the "Write authors into
  feature files" switch in Settings
- Then: every feature and scenario in the `.feature.md` files carries
  `**Created by:**` and `**Updated by:**` lines
- and off — the default, and any value other than `true` — the app still
  records and shows authorship, but the markdown leaves it out
- and the setting belongs to the data repo, not the device, so every peer
  of the repo writes the same form
- and switching it rewrites every feature file at once; the next sync
  pushes them, and a peer that has not pulled the change yet reads either
  form as a projection rather than an edit
- and turning it on or off keeps the settings file's other keys, and a
  settings file that is not valid JSON is refused, not overwritten

[test: authors-in-markdown is off unless the settings file says true : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L218 ]
[test: turning the setting on keeps the file's other keys : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L226 ]
[test: a settings file that is not JSON is refused, not overwritten : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L240 ]
[test: with authors-in-markdown on, the files carry the lines, and switching reprojects them : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L265 ]
[test: the projection writes authors only when asked, and either form is a projection : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specsAuthorship.test.ts#L192 ]

## A proposal is signed by the visitor, not the bot @v1 [proposed]

- Given: the spec proposals service (app/spec-proposals.feature.md), which
  pushes every proposal with one bot token
- When: ada proposes a scenario there
- Then: ada is its creator and the feature's last updater, and ada authors
  the commit
- and the bot's account signs nothing it did not write itself

[test: inside actAs getAuth answers the visitor; outside it, the stored sign-in : https://github.com/SpecDriven/specdriven_app/blob/main/tests/auth.test.ts#L265 ]
[test: a proposed scenario lands on main as [proposed], authored by the visitor and signed in its document : https://github.com/SpecDriven/specdriven_app/blob/main/tests/proposals.test.ts#L381 ]
