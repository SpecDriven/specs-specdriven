# Related pull requests

**Assigned:** David

**Created by:** vydra

**Updated by:** vydra

A feature, and each of its scenarios, can carry links to the pull requests
that implement it. The links are bookkeeping around the spec, not the spec:
they are stored in the feature's Automerge document only
(datastore/automerge-store.feature.md) and never written into the
`.feature.md`, so the projected markdown on GitHub stays what it was —
by default. Writing them into the markdown would let GitHub readers see the
PRs without the app, the way published ticket links do; the cost is a file
that changes every time a PR is linked, and a projection that no longer
depends on the document alone. That is left as a possible later setting, off
until somebody needs it.

Each link is a url and an optional title. A GitHub pull request url is
recognised (`https://github.com/<owner>/<repo>/pull/<n>`), and so is a
GitLab merge request; any other https url is accepted as it stands.

## S1: Link a pull request to a feature @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a feature is open
- When: the user presses the "Pull requests" chip in the feature header
- Then: a popover lists the feature's pull requests with a ✕ each, and an
  add field for another
- and the chip reads the count — "2 PRs" — or "Link PR" when there are none
- and adds and removes are stored on the feature's document the way a
  scenario's are, and the markdown is left alone

[test: the feature's own links, the same way : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L224 ]
[test: a scenario not in the document is null; the feature itself is target null : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L168 ]
[test: links a scenario, then the feature, answering with the editor's view : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L340 ]

## S2: Link a pull request to a scenario @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a scenario is open in the editor pane and has been saved at least
  once, so the document holds an entry for it
- When: the user pastes a pull request url into the "Pull requests" field
  and presses Add
- Then: the link is stored on the scenario's entry in the feature's
  document — a direct document operation, like deleting a draft — and the
  reload cron commits and pushes the document as it does any other saved
  change
- and the field lists it as `#<n> <title>`, linking to the pull request in a
  new tab, with a ✕ that removes it the same way
- and the `.feature.md` is not touched: its bytes are what the document
  projects, pull requests or none
- and the field is there for a published scenario too — its name, version
  and body stay read-only, its pull requests do not

[test: the document holds the links, the file does not, the editor sees them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L211 ]
[test: links a scenario by name and version, and by its document id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L158 ]
[test: a draft scenario can carry links, and the editor's view shows them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L246 ]
[test: a feature from before documents gets one minted first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L238 ]
[test: lists the links with a remove button each, and an add row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L34 ]
[test: before the scenario is in the document the field waits, and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L44 ]
[test: linking a pull request is not an edit autosave has to write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L78 ]

## S3: The links survive everything the markdown does @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a scenario or feature carries pull request links
- When: the scenario is saved, proposed or published; the feature is
  renamed or moved; a teammate's plain-markdown push is absorbed as drift;
  or a conflict is resolved from the merged document
- Then: the links are still on the document's entry afterwards — none of
  those writes can see them, so none of them can drop them
- and two peers linking different pull requests to the same scenario at the
  same time end with both, on both peers
- and a scenario deleted from the feature takes its links with it

[test: a save that edits the body, and one that renames the scenario by id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L263 ]
[test: publishing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L276 ]
[test: renaming the feature : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L284 ]
[test: an outside edit absorbed as drift : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L292 ]
[test: a three-way merge reconciled into the document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L302 ]
[test: a scenario deleted from the feature takes its links with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L314 ]
[test: two peers linking different pull requests concurrently end with both : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L188 ]
[test: the projection never carries them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L200 ]

## S4: The scenario card lists its pull requests @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a scenario carries pull request links
- When: the user looks at its card
- Then: a "Pull requests" list at the end of the card, after the automated
  tests, shows each as `#<n> <title>` — the number from the url, the title
  when one was stored, else the url's own path — linking to it in a new tab
- and a scenario with none shows no list

[test: each as #n title, linking to it in a new tab : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L19 ]
[test: a scenario with none shows no list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L28 ]
[test: the label is #n and the title, and never empty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L126 ]
[test: the number is read from GitHub and GitLab urls alike : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L119 ]

## S5: The title is read from GitHub @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a GitHub pull request url is typed into the add field
- Then: the app asks GitHub for the pull request, with the app's sign-in
  token first and the GitHub CLI's else, and fills the Title field with its
  title
- and a hint under the field says whether it is open, merged, closed or a
  draft
- and a title the user has typed by hand is kept; when nothing can be read —
  no such pull request, a private repository without a sign-in, demo mode —
  the link can still be added with the title typed or left empty, and the
  hint says why nothing was found
- and a url that is not an http or https url cannot be added, nor one the
  list already holds

[test: the pull request endpoint is asked, with the sign-in token, and the state is read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L404 ]
[test: merged, closed and draft are told apart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L415 ]
[test: no such pull request is a 200 with a reason, so the link can still be added : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L424 ]
[test: a url that is not a GitHub pull request is a 400 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L432 ]
[test: demo mode never asks GitHub : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L438 ]
[test: the hint words the state read from GitHub, or why nothing was read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L65 ]
[test: only an http(s) url the list does not already hold can be added : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequestField.test.tsx#L57 ]
[test: the route accepts only rows with an http(s) url and a string title : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L138 ]
[test: a GitHub pull request url is taken apart; anything else is not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pullRequests.test.ts#L100 ]
