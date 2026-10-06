# Navigation

**Assigned:** David

We want to have a unique url for each Feature and each scenario, to enable
deep linking and navigation from outside the app as well as sharing the links
via chat and email.

## URL scheme

| URL                                             | Meaning                                       |
| ----------------------------------------------- | --------------------------------------------- |
| `/`                                             | Home — no feature selected                    |
| `/features/<feature>`                           | Feature page for `<feature>.feature.md`       |
| `/features/<feature>/scenarios/<scenario>`      | Page of the latest version of that scenario   |
| `/features/<feature>/scenarios/<scenario>/v<N>` | Page of exactly version N — a permalink       |

A scenario url addresses that scenario's own page — the spec editor pane
(editor/spec-editor.feature.md) — not the feature page with a card picked out
of it. Highlighting a card is not a page and keeps the feature url; opening
the scenario, from a link or from the feature page, is what the scenario url
means.

Identity comes from the data repo, never from session state:

- `<feature>` is the feature file path minus `.feature.md`, and spans several
  url segments when the feature lives in a folder (folders.feature.md):
  `/features/billing/quotes` is `billing/quotes.feature.md`.
  File stems are already slugified and unique within their folder, so the
  mapping is bijective. The first `scenarios` segment with a following slug
  separates the feature path from the scenario ref, so a folder literally
  named `scenarios` is not addressable by deep link.
- Moving a feature to another folder changes its urls — folder placement is
  part of the path identity. A link to the old path reports "not found" like
  any stale link.
- `<scenario>` is `slugify(scenario name)` — the same slugify used for file
  stems. Scenario identity is (name, version), the same identity publish
  immutability keys on; list position is unstable across syncs and must never
  appear in a url.
- Because published scenarios are immutable, a versioned scenario url is a
  permalink that cannot rot. A url resolves to the same content on any
  instance pointed at the same data repo, so links can be shared via chat and
  email. This is the spec-ref identity `<file>#<name> vN` in url form.

## Resolution rules

- A versionless scenario url resolves to the highest version of that name.
- Segments after the scenario version are ignored, and a bare trailing
  `/scenarios` is dropped (chat and email clients often mangle link tails).
  Other unrecognized segments read as folder path — they resolve to "feature
  not found" rather than being silently truncated. A path that doesn't start
  with a known shape is home. A `<feature>` stem that already carries a
  `.feature.md` or legacy `.feature` suffix is accepted.
- If several scenario names slugify identically, the first match in file
  order wins.
- After resolution the address bar shows the canonical url of the page that
  is open: the feature url, or the versioned scenario url while a scenario
  page is open.
- The server serves the app shell for every `/features/...` path; resolution
  happens in the client via `/api/features/:file`.

## Links from outside the app

The desktop app (desktop-app.feature.md) serves the frontend from an
internal `app://specdriven` origin that nothing outside its own window can
reach, so it also registers `specdriven://` with the operating system. The two
carry the same paths — the scheme is a prefix swap, and every rule above
applies to both:

| Where the link is used       | Form                               |
| ---------------------------- | ---------------------------------- |
| Web version                  | `http://<host>/features/<feature>` |
| Desktop app, shared anywhere | `specdriven://features/<feature>`  |

So `specdriven://features/accidents/record-car-accident` addresses exactly
what `/features/accidents/record-car-accident` does, versioned scenario
permalinks included. Registering the scheme is what lets the OS route such a
link — clicked in chat, email, a browser or a terminal — to the app.

## a second link reuses the running app @v1 [proposed]

- Given: the desktop app is already open on some other feature
- When: another `specdriven://` link is opened
- Then: the same window moves to the linked feature — no second window and no
  reload — and Back returns to what was on screen before

[test: a link moves the open window to the linked page, with no second window : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L120 ]
[test: the move is a history push, so Back returns and nothing reloads : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L128 ]
[test: a minimized window is restored and brought to the front : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L140 ]
[test: a window still loading takes the link once it has loaded : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L146 ]
[test: a second launch carrying a link shows it in the running window : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L154 ]
[test: a second launch without a link only brings the window to the front : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L162 ]

## browser history and reload @v1 [proposed]

- Selecting a feature (sidebar click, create) pushes a history entry —
  back/forward walk through previously viewed features
- Moving the scenario selection with the up/down keys rewrites the url in
  place (no history entry per keystroke)
- Reloading the page restores the same feature and scenario selection

## deep link before a data repo is configured @v1 [proposed]

- Given: the settings file has no data repo recorded
- When: the user opens a deep link
- Then: the repo prompt shows first (app-start.feature.md); once a repo is
  accepted, the deep link is honored

## link to a missing feature @v1 [proposed]

- When: the user opens `/features/<something>` that doesn't resolve to a file
- Then: the app shows home with a note that the feature was not found and the
  link may be stale

## link to a missing scenario @v1 [proposed]

- When: the feature resolves but the scenario slug (or version) doesn't
- Then: the feature opens with no selection and a note that the scenario was
  not found — it may have been renamed or deleted

## move a scenario to another feature @v1 [proposed]

- Given: a scenario's ⋯ menu in the sidebar tree, which offers Move to… next
  to Delete and on the same terms — a published scenario only while Settings
  allows deleting or moving published specs
- When: the user picks Move to… and chooses another feature from the list,
  filtered by name or path as they type
- Then: the scenario leaves its feature and joins the chosen one with its
  name, version, status, body, tests, ticket and pull request links
- And: a draft stays a draft and a proposed scenario stays proposed, saved
  locally like any edit; a published one stays published, and both features
  are committed and pushed right away
- And: a move that would give the chosen feature a second scenario with the
  same name and version is refused, and nothing changes
- And: an editor pane open on the moved scenario follows it to its new
  feature, keeping what was typed

[test: a proposed scenario moves with its body and tests, and leaves the source : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L77 ]
[test: a draft moves as a draft — into the target's document, never its file : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L92 ]
[test: pull request links, which only the document holds, move with the scenario : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L102 ]
[test: a move that would collide with a scenario already in the target is refused, and nothing changes : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L110 ]
[test: with deleting published specs off, a published scenario does not move : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L136 ]
[test: with deleting published specs on, a published scenario moves and stays published : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L145 ]
[test: the feature picker filters by label or by path : https://github.com/SpecDriven/specdriven_app/blob/main/tests/moveScenario.test.ts#L167 ]

## open a feature link @v1 [proposed]

- Given: the data repo contains `navigation.feature.md`
- When: the user opens `/features/navigation`
- Then: the app shows that feature — same view as clicking it in the sidebar

## open a link from outside the app @v1 [proposed]

- Given: the desktop app is installed and a data repo is configured
- When: the user clicks a `specdriven://features/<feature>` link in another
  app — chat, email, a browser, the terminal
- Then: SpecDriven comes to the front showing that feature, launching first if
  it wasn't running; the app the link was clicked in stays where it was

[test: the url host is the first path segment : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L16 ]
[test: folder paths and scenario refs survive intact : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L22 ]
[test: the three-slash form carries the whole path : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L31 ]
[test: surrounding whitespace from a copy-paste is tolerated : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L44 ]
[test: other schemes and non-urls are not deep links : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L48 ]
[test: finds the link among the launch arguments : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L206 ]
[test: with no window open, a link opens one on the linked page : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L170 ]
[test: a launch by link opens the first window on that page : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L179 ]
[test: a specdriven link in a spec body is followed in place : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linkRouting.test.ts#L60 ]
[test: the installed app declares the specdriven scheme to the OS : https://github.com/SpecDriven/specdriven_app/blob/main/tests/linuxPackage.test.ts#L51 ]

## open a scenario link @v1 [proposed]

- Given: `navigation.feature.md` contains scenario "ESC key closes the dialog" v2
- When: the user opens `/features/navigation/scenarios/esc-key-closes-the-dialog/v2`
- Then: the feature opens with that scenario card selected, highlighted, and
  scrolled into view — the same highlight the up/down keys move, so Enter
  opens the full body

## Scenario navigation @v1 [proposed]

When we are in a scenario editor pane, we want to easily switch to another scenario wihout going back to the feature view.
In the tree on the left show scenarios under the feature.
Allow to collapse and expand the outline.

## share a link @v1 [proposed]

- Sidebar entries and scenario card titles are real links: right-click →
  Copy Link and cmd-click → new tab work natively
- Every scenario card has a "Copy link" action that puts the absolute
  versioned url on the clipboard; its label flips to "Copied" for 2 sec
- The feature name in the editor bar has the same "Copy link" action for
  the feature url
- In the desktop app those actions copy the `specdriven://` form, so what
  lands on the clipboard is a link that works anywhere else on the machine

[test: a link built by the app round-trips back to its route : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L56 ]
[test: desktopLink puts the folder's display name first : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L74 ]
[test: a link round-trips to its folder and its route : https://github.com/SpecDriven/specdriven_app/blob/main/tests/deepLink.test.ts#L97 ]

## versionless scenario link opens the latest version @v1 [proposed]

- Given: the scenario name exists as v1 and v2
- When: the user opens the scenario url without the `/v<N>` segment
- Then: v2 is selected, and the address bar shows the canonical `/v2` url

## window title follows the selection @v1 [proposed]

- `SpecDriven` on home, `SpecDriven — <feature name>` on a feature,
  `SpecDriven — <feature name> · <scenario name> v<N>` with a selection —
  chat and email link previews show the title
