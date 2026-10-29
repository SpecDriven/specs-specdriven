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
  immutability keys on; list position — the `S<n>` a scenario shows
  (ui.feature.md "Number scenarios and move them up or down") — is not
  identity: a move changes it, so it must never appear in a url.
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

A link opened while the app is closed loads its first window straight at
the linked path, so the app shell has to work from a nested url as well as
from `/`: its asset urls are root-relative however deep the path, never
relative to the page (a `./index.js` under `/features/app/desktop-app` would
ask for `/features/app/index.js`, get the shell back as the script, and
leave the window blank).

## S1: a second link reuses the running app @v1 [proposed]

- Given: the desktop app is already open on some other feature
- When: another `specdriven://` link is opened
- Then: the same window moves to the linked feature — no second window and no
  reload — and Back returns to what was on screen before

[test: a link moves the open window to the linked page, with no second window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L120 ]
[test: the move is a history push, so Back returns and nothing reloads : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L128 ]
[test: a minimized window is restored and brought to the front : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L140 ]
[test: a window still loading takes the link once it has loaded : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L146 ]
[test: a second launch carrying a link shows it in the running window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L154 ]
[test: a second launch without a link only brings the window to the front : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L162 ]

## S2: browser history and reload @v1 [proposed]

- Selecting a feature (sidebar click, create) pushes a history entry —
  back/forward walk through previously viewed features
- Moving the scenario selection with the up/down keys rewrites the url in
  place (no history entry per keystroke)
- Reloading the page restores the same feature and scenario selection

## S3: deep link before a data repo is configured @v1 [proposed]

- Given: the settings file has no data repo recorded
- When: the user opens a deep link
- Then: the repo prompt shows first (app-start.feature.md); once a repo is
  accepted, the deep link is honored

## S4: link to a missing feature @v1 [proposed]

- When: the user opens `/features/<something>` that doesn't resolve to a file
- Then: the app shows home with a note that the feature was not found and the
  link may be stale

## S5: link to a missing scenario @v1 [proposed]

- When: the feature resolves but the scenario slug (or version) doesn't
- Then: the feature opens with no selection and a note that the scenario was
  not found — it may have been renamed or deleted

## S6: move a scenario to another feature @v1 [proposed]

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

[test: a proposed scenario moves with its body and tests, and leaves the source : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L77 ]
[test: a draft moves as a draft — into the target's document, never its file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L93 ]
[test: pull request links, which only the document holds, move with the scenario : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L103 ]
[test: a move that would collide with a scenario already in the target is refused, and nothing changes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L111 ]
[test: with deleting published specs off, a published scenario does not move : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L137 ]
[test: with deleting published specs on, a published scenario moves and stays published : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L146 ]
[test: the feature picker filters by label or by path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/moveScenario.test.ts#L168 ]

## S7: open a feature link @v1 [proposed]

- Given: the data repo contains `navigation.feature.md`
- When: the user opens `/features/navigation`
- Then: the app shows that feature — same view as clicking it in the sidebar

## S8: open a link from outside the app @v1 [proposed]

- Given: the desktop app is installed and a data repo is configured
- When: the user clicks a `specdriven://features/<feature>` link in another
  app — chat, email, a browser, the terminal
- Then: SpecDriven comes to the front showing that feature, launching first if
  it wasn't running; the app the link was clicked in stays where it was

[test: the url host is the first path segment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L16 ]
[test: folder paths and scenario refs survive intact : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L22 ]
[test: the three-slash form carries the whole path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L31 ]
[test: surrounding whitespace from a copy-paste is tolerated : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L44 ]
[test: other schemes and non-urls are not deep links : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L48 ]
[test: finds the link among the launch arguments : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L206 ]
[test: with no window open, a link opens one on the linked page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L170 ]
[test: a launch by link opens the first window on that page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L179 ]
[test: a specdriven link in a spec body is followed in place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L60 ]
[test: the installed app declares the specdriven scheme to the OS : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linuxPackage.test.ts#L51 ]

## S9: open a scenario link @v1 [proposed]

- Given: `navigation.feature.md` contains scenario "ESC key closes the dialog" v2
- When: the user opens `/features/navigation/scenarios/esc-key-closes-the-dialog/v2`
- Then: the feature opens with that scenario card selected, highlighted, and
  scrolled into view — the same highlight the up/down keys move, so Enter
  opens the full body

## S10: Scenario navigation @v1 [proposed]

When we are in a scenario editor pane, we want to easily switch to another scenario wihout going back to the feature view.
In the tree on the left show scenarios under the feature.
Allow to collapse and expand the outline.

## S11: share a link @v1 [proposed]

- Sidebar entries and scenario card titles are real links: right-click →
  Copy Link and cmd-click → new tab work natively
- Every scenario card has a "Copy link" action that puts the absolute
  versioned url on the clipboard; its label flips to "Copied" for 2 sec
- The feature page's ⋯ menu has the same "Copy link" action for the
  feature url, just like the scenario card's ⋯ menu
- The scenario editor's top bar has a "Copy link" icon button beside its
  previous / next arrows for the open scenario's url; its icon flips to a
  check mark for 2 sec
- In the desktop app those actions copy the `specdriven://` form, so what
  lands on the clipboard is a link that works anywhere else on the machine

[test: a link built by the app round-trips back to its route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L56 ]
[test: desktopLink puts the folder's display name first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L74 ]
[test: a link round-trips to its folder and its route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L97 ]

## S12: versionless scenario link opens the latest version @v1 [proposed]

- Given: the scenario name exists as v1 and v2
- When: the user opens the scenario url without the `/v<N>` segment
- Then: v2 is selected, and the address bar shows the canonical `/v2` url

## S13: window title follows the selection @v1 [proposed]

- `SpecDriven` on home, `SpecDriven — <feature name>` on a feature,
  `SpecDriven — <feature name> · <scenario name> v<N>` with a selection —
  chat and email link previews show the title

## S14: reorder scenarios by dragging them in the tree @v1 [proposed]

- Given: a feature's scenario outline expanded in the sidebar tree
- When: the user drags one of its scenarios onto another row of the same
  outline
- Then: a line shows where it will land — just above the row while the
  pointer is on its upper half, just below on its lower half — and dropping
  puts the scenario there; every number follows its new position and the
  feature is saved in its new order, as `Move up` / `Move down` save it
  ("Number scenarios and move them up or down" in ui.feature.md)
- And: this works for any feature in the tree, not only the open one, and
  while an editor pane is open — the pane saves what it holds and stays on
  the scenario it showed
- A scenario dragged onto another feature moves there instead ("drag a
  scenario to another feature"); dragged onto a folder it does nothing.

[test: a scenario dropped on the upper half of a row lands just before it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioDragReorder.test.ts#L24 ]
[test: a scenario dropped on the lower half of a row lands just after it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioDragReorder.test.ts#L31 ]
[test: a drop that leaves the scenario where it was is no move : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioDragReorder.test.ts#L38 ]
[test: scenarios are found by name and version, so a draft between them keeps its place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioDragReorder.test.ts#L47 ]
[test: a scenario no longer in the feature moves nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioDragReorder.test.ts#L61 ]

## S15: a link launches the app straight onto its page @v1 [proposed]

- Given: the desktop app is not running
- When: the OS starts it for a `specdriven://features/<feature>` link
- Then: the first window loads at that nested path and renders — the app
  mounts, with the feature, or the repo prompt when no data repo is
  configured yet — rather than staying blank because the shell's scripts
  were looked for under the link's path

[test: the built shell's ./ asset urls become root-relative, so they load at any path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L60 ]
[test: absolute, remote and parent-relative urls pass through untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L72 ]
[test: single-quoted attributes are rewritten too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/static.test.ts#L79 ]
[test: the last of two or more launches is by link to a nested page; a lone launch is plain : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L59 ]
[test: a launch by link must have loaded the window at the linked path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L68 ]

## S16: clone a scenario @v1 [proposed]

- Given: a scenario's ⋯ menu — on its card in the feature view, or on its row
  in the sidebar tree — which offers Clone for every scenario, published
  ones included
- When: the user picks Clone
- Then: a copy lands right after it as a draft at v1 named "<name> (clone)",
  or "(clone 2)", "(clone 3)"… when that name is taken, and opens in the editor
  pane, ready to be renamed and changed
- And: the copy carries the scenario's body, feature flags, custom attributes
  and polarity, but not its tests, ticket, qTest case, pull request, Aha! or
  Figma links or stored files — those belong to the original
- And: the original is left exactly as it was

[test: the copy is a draft named '<name> (clone)' at v1, right after the original : https://github.com/SpecDriven/specdriven-app/blob/main/tests/cloneScenario.test.ts#L67 ]
[test: the copy carries the spec text, flags, attributes and polarity, not the tests, ticket or pull requests : https://github.com/SpecDriven/specdriven-app/blob/main/tests/cloneScenario.test.ts#L83 ]
[test: a second copy takes the next free name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/cloneScenario.test.ts#L105 ]
[test: a published scenario can be cloned, and stays as it was : https://github.com/SpecDriven/specdriven-app/blob/main/tests/cloneScenario.test.ts#L113 ]
[test: the route clones by name and version, and refuses what is not there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/cloneScenario.test.ts#L124 ]

## S17: drag a scenario to another feature @v1 [proposed]

- Given: a scenario card on a feature page, or a scenario row in the sidebar
  tree's outline
- When: the user drags it onto another feature's row in the sidebar
- Then: that row is outlined while the pointer is over it, and dropping moves
  the scenario there, last in its order — the same move as `Move to…`, with
  everything it carries and on the same terms: a published scenario only
  while Settings allows deleting or moving published specs, otherwise it
  drops nowhere
- And: dragged onto a row of another feature's expanded outline, or out of
  another feature's outline onto a card of the open feature, a line shows
  where it will land — above the row or card on its upper half, below on its
  lower half — and dropping moves it in at that spot
- And: dragged onto another card of its own feature page, it lands before or
  after that card, as dragging it in the outline reorders it ("reorder
  scenarios by dragging them in the tree")
- And: the toast that confirms the move offers Undo, which moves it back to
  the place it left

[test: a scenario dropped on the upper half of a row in another feature's outline lands just before it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L53 ]
[test: a scenario dropped on the lower half of a row in another feature's outline lands just after it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L61 ]
[test: a drop next to a scenario the feature no longer has lands last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L66 ]
[test: a draft dropped between two scenarios keeps that place, in the document only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L71 ]
[test: the move route takes the drop's place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L77 ]
[test: undo puts a moved scenario back in its old place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L94 ]
[test: a scenario's place is before the one after it, or after the one before it when last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L104 ]
[test: a dragged scenario drops into any other feature, a published one only while Settings allows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dragScenarioToFeature.test.ts#L115 ]
