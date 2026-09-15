# Related pull requests

**Assigned:** David

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

## Link a pull request to a feature @v1 [proposed]

- Given: a feature is open
- When: the user presses the "Pull requests" chip in the feature header
- Then: a popover lists the feature's pull requests with a ✕ each, and an
  add field for another
- and the chip reads the count — "2 PRs" — or "Link PR" when there are none
- and adds and removes are stored on the feature's document the way a
  scenario's are, and the markdown is left alone

## Link a pull request to a scenario @v1 [proposed]

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

## The links survive everything the markdown does @v1 [proposed]

- Given: a scenario or feature carries pull request links
- When: the scenario is saved, proposed or published; the feature is
  renamed or moved; a teammate's plain-markdown push is absorbed as drift;
  or a conflict is resolved from the merged document
- Then: the links are still on the document's entry afterwards — none of
  those writes can see them, so none of them can drop them
- and two peers linking different pull requests to the same scenario at the
  same time end with both, on both peers
- and a scenario deleted from the feature takes its links with it

## The scenario card lists its pull requests @v1 [proposed]

- Given: a scenario carries pull request links
- When: the user looks at its card
- Then: a "Pull requests" list at the end of the card, after the automated
  tests, shows each as `#<n> <title>` — the number from the url, the title
  when one was stored, else the url's own path — linking to it in a new tab
- and a scenario with none shows no list

## The title is read from GitHub @v1 [proposed]

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
