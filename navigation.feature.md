# Navigation

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

## browser history and reload @v1 [proposed]

- Selecting a feature (sidebar click, create) pushes a history entry —
  back/forward walk through previously viewed features
- Opening a scenario's page pushes one too, so Back leaves it exactly as `<`
  does; leaving it rewrites the url in place, and so does a rename that moves
  an open page's url — Back never lands on a url the rename retired
- Moving the highlight through the cards with the up/down keys does not touch
  the url (no history entry per keystroke)
- Reloading the page restores the same feature, and the scenario page if one
  was open

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

## open a scenario link @v1 [proposed]

- Given: `navigation.feature.md` contains scenario "ESC key closes the dialog" v2
- When: the user opens `/features/navigation/scenarios/esc-key-closes-the-dialog/v2`
- Then: that scenario's page opens directly — the editor pane on it, in the
  feature's slot (editor/spec-editor.feature.md) — with its card selected and
  scrolled into view underneath, so `<` and ESC land back on it
- The same for a link opened from outside the app, a search hit
  (search.feature.md), and a reload of the url

## share a link @v1 [proposed]

- Sidebar entries and scenario card titles are real links: right-click →
  Copy Link and cmd-click → new tab work natively
- Every scenario card has a "Copy link" action that puts the absolute
  versioned url on the clipboard; its label flips to "Copied" for 2 sec. What
  is shared is a link to that scenario's page: whoever opens it lands on the
  scenario, not on the feature it lives in
- The feature name in the editor bar has the same "Copy link" action for
  the feature url
- In the desktop app those actions copy the `specdriven://` form, so what
  lands on the clipboard is a link that works anywhere else on the machine

## versionless scenario link opens the latest version @v1 [proposed]

- Given: the scenario name exists as v1 and v2
- When: the user opens the scenario url without the `/v<N>` segment
- Then: v2 is selected, and the address bar shows the canonical `/v2` url

## window title follows the selection @v1 [proposed]

- `SpecDriven` on home, `SpecDriven — <feature name>` on a feature,
  `SpecDriven — <feature name> · <scenario name> v<N>` on a scenario page —
  the title names what the url names, so chat and email link previews of a
  scenario link show the scenario

## a scenario page carries its own url @v1 [proposed]

- Given: the feature page for `quotes.feature.md` is open, with the highlight
  on some card — the address bar reads `/features/quotes`
- When: the user opens the scenario "Basic quote" v2 — Edit on its card,
  Enter on the highlighted card, or a double-click
- Then: the address bar reads
  `/features/quotes/scenarios/basic-quote/v2`, the window title names the
  scenario, and "Copy link" on that card shares exactly the url of the page
  now on screen
- When: the user leaves the page with `<` or ESC
- Then: the address bar reads `/features/quotes` again, and browser Back from
  the open page would have done the same — saving what the pane held first
