# qTest test cases

A feature, and each of its scenarios, can link to the qTest test case that
covers it. The link works like the ticket link
(external-links/project-management-jira-linear-etc.feature.md): one link per
feature and one per scenario, a url plus an id, held in the feature's
Automerge document (datastore/automerge-store.feature.md) and written into
the `.feature.md` while "Publish external links" is on. A feature or
scenario can carry a ticket and a test case at the same time.

qTest is a setting of its own, off by default, so a team that doesn't use
qTest never sees it.

## S1: Link a qTest test case to a feature @v1 [proposed]

- Given: a feature is open and qTest test cases are switched on
- When: the user presses the "Link qTest" chip in the feature header
- Then: a popover asks for the test case's url and its id (TC-42)
- and Save writes the link to the feature; clearing both fields and saving
  removes it
- and the chip then reads "qTest:TC-42", with a button beside it that opens
  the test case in a new tab
- and a url with no id, an id with no url, or a url that is not http or
  https blocks the save with a message instead of being saved half-done

[test: the chip reads the program and the test case id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L303 ]
[test: empty is no link; a whole one resolves; anything partial is an error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L288 ]
[test: writes the test cases into the file while links are published : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L223 ]

## S2: Link a qTest test case to a scenario @v1 [proposed]

- Given: a scenario is open in the editor pane and qTest test cases are
  switched on
- When: the user fills in the "qTest test case" field: the url and the id
- Then: the link is saved with the scenario, like its ticket link, and
  autosave carries it
- and the scenario's card shows a "qTest:TC-42" chip that opens the test
  case in a new tab
- and the field is there for a published scenario too, because the link is
  bookkeeping around the spec, not the spec

[test: the autosave signature follows the test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L315 ]
[test: a published scenario's test case stays editable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L196 ]
[test: publishing keeps the scenario's test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L257 ]

## S3: Publishing qTest links into projected feature (.md) files @v1 [proposed]

- Given: user setting 'Publish external links' is ON
- Then: the link is written on a line of its own under the heading it
  belongs to, after the ticket line
- Write format: [qTest:<test case id>](<web link>)
- Example: [qTest:TC-42](https://acme.qtestnet.com/p/12/portal/project#tab=testdesign&object=1&id=901)
- and with the setting OFF the line is left out of the file, while the app
  still shows and keeps the link
- and a line whose program is qTest, in any case, is read as the test case,
  never as the ticket; the line may come before or after the ticket line
- and a second qTest line under the same heading is ordinary content

[test: is lifted into qtest, beside the ticket, under the heading it belongs to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L101 ]
[test: survives a parse/serialize round trip : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L115 ]
[test: may come before the ticket line, and is written back after it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L121 ]
[test: is told from a ticket by its program, whatever its case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L129 ]
[test: a second qTest line is content, not a second link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L138 ]
[test: is left out with the ticket links when links are not written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L144 ]
[test: keeps them out of the projection while Publish external links is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L162 ]
[test: with links unpublished the file has none, but the editor sees and keeps them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L230 ]

## S4: qTest is reserved for test cases @v1 [proposed]

- Given: Settings → Project management
- When: the user names qTest as the project management tool, or a ticket
  link is entered under the tool name qTest
- Then: it is refused with a message, because a ticket written as
  [qTest:…](…) would be read back as the test case

[test: a ticket can't be recorded under qTest, where it would read back as the test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L307 ]
[test: qTest can't be the project management tool : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L376 ]

## S5: Settings switch qTest test cases on @v1 [proposed]

- Given: Settings → qTest
- When: the user turns **qTest test cases** on
- Then: features and scenarios offer a qTest link: a chip in the feature
  header, a field in the scenario editor, a chip on the scenario card
- and it is off unless turned on; off is stored as an absent key
- and turning it off hides the chip and field; links already recorded stay
  in the documents, and in the files while "Publish external links" is on,
  and show again when it is turned back on

[test: is off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L355 ]
[test: round-trips through the settings file, off stored as an absent key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L362 ]

## S6: The links survive everything the markdown does @v1 [proposed]

- Given: a feature or scenario carries a qTest test case link
- When: it is saved or published; a teammate whose "Publish external links"
  is set the other way rewrites the file; a plain-markdown push changes the
  link; or two sides change the link at once
- Then: the document keeps the link, and changes to it are absorbed like any
  other edit
- and a file written with or without the qTest lines counts as the same
  document, so peers with different settings never take turns rewriting it
- and when both sides of a merge change the link, ours wins; a change on one
  side only stands
- and replacing or clearing the test case leaves the ticket link alone

[test: holds the qTest links and projects them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L155 ]
[test: reconcile sets, replaces and clears them — and a no-op stays one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L171 ]
[test: a three-way merge keeps the side that changed it, ours when both did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L203 ]
[test: a file in the other form is not drift; an edited test case is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L244 ]

## S7: The test case id comes from the url when the url gives it away @v1 [proposed]

- Given: the qTest fields are open
- When: the user pastes a url that contains the id, e.g. `…&id=TC-42`
- Then: the id field fills in with TC-42
- and a url that names the test case only by qTest's internal number leaves
  the id for the user to type
- and an id typed by hand is never replaced when the url changes; an id
  that came from the old url follows the new one

[test: the id comes off the url only when the url spells it out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L270 ]
[test: a url fills the id while it is empty or derived; a typed one stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L279 ]

## S8: A qTest token is kept per site on this device @v1 [proposed]

- Given: Settings → qTest, with qTest test cases switched on
- When: the user enters the qTest site's url (or any module url of it) and
  an API token — the bearer token qTest shows under the user's name → API &
  SDK — and presses "Save token"
- Then: the token is checked against the site before it is kept: the
  projects it can see are listed, and a token qTest refuses, or that sees no
  project at all, is not kept and the form says why
- and the site is listed with how many projects the token reaches, never the
  token itself, with a Remove button
- and saving a token for a site that already has one replaces it
- and the token is kept on this device only: an owner-only file, or the OS
  key store in the desktop app — never settings.json or the data repo
- and the hosted app, which keeps no device secrets, offers none of this

[test: a token qTest accepts is kept, owner-only, and listed without the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L258 ]
[test: a token qTest refuses, or that sees no project, is not kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L268 ]
[test: a blank token or a url that is no site is refused before qTest is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L278 ]
[test: a saved token replaces the site's last one, and can be removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L284 ]
[test: the hosted app refuses qTest altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L472 ]
[test: lists each site with what its token reaches, never the token, each with Remove : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestDialogs.test.tsx#L74 ]
[test: offers the url and token fields, the token as a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestDialogs.test.tsx#L82 ]
[test: is the site itself, or any portal url of it, normalized : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L55 ]
[test: two urls are the same site by origin and context path, whatever the case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L62 ]

## S9: Create qTest test cases from a feature @v1 [proposed]

- Given: a feature is open, qTest test cases are switched on, and this
  device holds a token for the site
- When: the user picks "Create qTest test cases…" in the feature's ⋯ menu,
  pastes the url of the parent module from qTest's Test Design tree
  (`…/p/12/portal/project#tab=testdesign&object=2&id=901`), chooses "All
  scenarios" or "Published scenarios only", and presses "Create test cases"
- Then: a module named after the feature is created under the parent module,
  and each scenario in scope becomes one test case in it
- and the feature links the module and each scenario its test case, as
  `[qTest:MD-66](url)` and `[qTest:TC-51](url)` lines, the way the links are
  entered by hand; the toast says how many were created and offers "Open
  module"
- and the dialog offers the last parent module used on this device next time
- and Create waits for a module url on a site with a token, and says what is
  missing; what qTest refuses (no such module, a token it no longer accepts,
  a project the token cannot write to) is shown as one sentence
- and a test case qTest refuses stops the run there: the ones written before
  it are linked back and the dialog stays open on a line naming the
  scenario, so creating them again carries on from it

[test: makes a module named after the feature under the parent, a test case per scenario, and links them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L295 ]
[test: published only leaves the proposed scenarios out, keeping the numbers the app shows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L395 ]
[test: a test case qTest refuses stops the run, and what was written is still linked back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L407 ]
[test: a test case refused mid-run keeps the dialog open on it, with its details : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestFailureDetails.test.tsx#L135 ]
[test: what cannot be published says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L448 ]
[test: a token qTest no longer accepts is reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L464 ]
[test: a modal dialog with the parent module url, the two scopes and Create test cases : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestDialogs.test.tsx#L18 ]
[test: Create waits for a module url on a site with a token, and says what is missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestDialogs.test.tsx#L56 ]
[test: names the site, the project and the module : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L21 ]
[test: keeps an on-premise context path in the site : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L26 ]
[test: is not a test case url, a project url or a non-url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L31 ]

## S10: Each scenario becomes one test case @v1 [proposed]

- Given: a scenario is being created as a test case
- Then: its name is the scenario's number and name, "S3: Declined Quote", so
  qTest's listing keeps the feature's order; a draft, which has no number, is
  named as is
- and its description names the feature and its file in SpecDriven
- and its Given lines are the test case's precondition
- and each When line starts a test step, with the "and" lines that follow it;
  the Then lines after it are that step's expected result
- and a Then with no When before it is a step that checks the outcome; a
  scenario with no steps at all gets one step, since qTest wants one
- and the test case carries every required field of the project's test cases
  at its default value, read from qTest rather than assumed, so a project
  with its own required fields accepts it

[test: read bold keywords, colon keywords, and And lines alike : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L19 ]
[test: a wrapped line continues the line before; a plain list item joins the block : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L33 ]
[test: text before any keyword counts as Given : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L45 ]
[test: has the Given lines as precondition, a step per When, and the Then lines as expected : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L53 ]
[test: a Then with no When before it becomes a check step; a Given between steps sets the next one up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L75 ]
[test: a scenario with no steps gets one step so qTest accepts it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L92 ]
[test: a draft, which has no number, is named as is; a long name is cut to what qTest keeps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestSteps.test.ts#L101 ]
[test: is built the way qTest shows it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L48 ]

## S11: Creating the test cases again updates them @v1 [proposed]

- Given: a feature whose scenarios were created as test cases before
- When: the user creates them again
- Then: a scenario that links a test case still in qTest is updated there —
  a new version with the scenario's current steps — and moved into the
  feature's module when it sat elsewhere; a scenario whose test case was
  deleted in qTest, or that links none, gets a new one
- and the module the feature links is reused, and moved under the new parent
  when another is named; with that module gone, a child of the parent with
  the feature's name is reused, else one is made anew
- and nothing is written to the feature when every link already stands

[test: creating them again updates the test cases the scenarios link, in the module the feature links : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L345 ]
[test: the linked module moves under a new parent; a deleted one is found by name, or made anew : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L366 ]
[test: a scenario whose test case was deleted in qTest gets a new one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L384 ]
[test: with Publish external links off the file has no line, but the feature keeps the links : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L438 ]
[test: a feature whose test cases were created before says creating them again updates its module : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestDialogs.test.tsx#L40 ]
[test: gives its numeric id, and a module url none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestUrl.test.ts#L42 ]

## S12: Every call to qTest is logged, and a failure shows what qTest answered @v1 [proposed]

- Given: qTest test cases are switched on and this device holds a token for
  the site, or a token is being added in Settings → qTest
- When: the app calls qTest — checking a token, finding or making the
  module, reading the required fields, writing a test case
- Then: each call is recorded in the app's log under the "qtest" component,
  for the Observability page: the method and url, what the app sent, qTest's
  status and answer, and how long it took — never the token
- and an answered call is logged as info, a refusal qTest explains (400, 401,
  403, 404) as a warning, and a server error or a site that could not be
  reached as an error
- and when Create test cases or Save token fails because of what qTest
  answered, the dialog shows the failure as one line, and clicking it opens
  a detail box: the request (method and url), what the app sent, qTest's
  status, its content type, when it failed, and what qTest answered; a site
  that could not be reached says there was no answer, and why
- and "Open in Observability" in the qTest dialog's detail box opens the
  Observability page narrowed to qTest's entries
- and a failure the app finds before asking qTest (no token, a url that names
  no module) stays a plain sentence with no detail box

[test: every call to qTest is logged under qtest, with both bodies and never the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L484 ]
[test: a failure answers the request and qTest's answer as details, and is logged as a warning : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L505 ]
[test: a token qTest refuses comes back with what qTest answered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L521 ]
[test: an unreachable site is logged as an error with the network error, and no status : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestPublish.test.ts#L528 ]
[test: shows the error as a line that opens the request, what was sent and qTest's answer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestFailureDetails.test.tsx#L104 ]
[test: an error without details stays plain text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestFailureDetails.test.tsx#L160 ]
[test: shows qTest's answer on click : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestFailureDetails.test.tsx#L169 ]
