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

## Link a qTest test case to a feature @v1 [proposed]

- Given: a feature is open and qTest test cases are switched on
- When: the user presses the "Link qTest" chip in the feature header
- Then: a popover asks for the test case's url and its id (TC-42)
- and Save writes the link to the feature; clearing both fields and saving
  removes it
- and the chip then reads "qTest:TC-42", with a button beside it that opens
  the test case in a new tab
- and a url with no id, an id with no url, or a url that is not http or
  https blocks the save with a message instead of being saved half-done

[test: the chip reads the program and the test case id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L296 ]
[test: empty is no link; a whole one resolves; anything partial is an error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L281 ]
[test: writes the test cases into the file while links are published : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L216 ]

## Link a qTest test case to a scenario @v1 [proposed]

- Given: a scenario is open in the editor pane and qTest test cases are
  switched on
- When: the user fills in the "qTest test case" field: the url and the id
- Then: the link is saved with the scenario, like its ticket link, and
  autosave carries it
- and the scenario's card shows a "qTest:TC-42" chip that opens the test
  case in a new tab
- and the field is there for a published scenario too, because the link is
  bookkeeping around the spec, not the spec

[test: the autosave signature follows the test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L308 ]
[test: a published scenario's test case stays editable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L189 ]
[test: publishing keeps the scenario's test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L250 ]

## Publishing qTest links into projected feature (.md) files @v1 [proposed]

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

[test: is lifted into qtest, beside the ticket, under the heading it belongs to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L94 ]
[test: survives a parse/serialize round trip : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L108 ]
[test: may come before the ticket line, and is written back after it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L114 ]
[test: is told from a ticket by its program, whatever its case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L122 ]
[test: a second qTest line is content, not a second link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L131 ]
[test: is left out with the ticket links when links are not written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L137 ]
[test: keeps them out of the projection while Publish external links is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L155 ]
[test: with links unpublished the file has none, but the editor sees and keeps them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L223 ]

## qTest is reserved for test cases @v1 [proposed]

- Given: Settings → Project management
- When: the user names qTest as the project management tool, or a ticket
  link is entered under the tool name qTest
- Then: it is refused with a message, because a ticket written as
  [qTest:…](…) would be read back as the test case

[test: a ticket can't be recorded under qTest, where it would read back as the test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L300 ]
[test: qTest can't be the project management tool : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L369 ]

## Settings switch qTest test cases on @v1 [proposed]

- Given: Settings → qTest
- When: the user turns **qTest test cases** on
- Then: features and scenarios offer a qTest link: a chip in the feature
  header, a field in the scenario editor, a chip on the scenario card
- and it is off unless turned on; off is stored as an absent key
- and turning it off hides the chip and field; links already recorded stay
  in the documents, and in the files while "Publish external links" is on,
  and show again when it is turned back on

[test: is off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L348 ]
[test: round-trips through the settings file, off stored as an absent key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L355 ]

## The links survive everything the markdown does @v1 [proposed]

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

[test: holds the qTest links and projects them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L148 ]
[test: reconcile sets, replaces and clears them — and a no-op stays one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L164 ]
[test: a three-way merge keeps the side that changed it, ours when both did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L196 ]
[test: a file in the other form is not drift; an edited test case is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L237 ]

## The test case id comes from the url when the url gives it away @v1 [proposed]

- Given: the qTest fields are open
- When: the user pastes a url that contains the id, e.g. `…&id=TC-42`
- Then: the id field fills in with TC-42
- and a url that names the test case only by qTest's internal number leaves
  the id for the user to type
- and an id typed by hand is never replaced when the url changes; an id
  that came from the old url follows the new one

[test: the id comes off the url only when the url spells it out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L263 ]
[test: a url fills the id while it is empty or derived; a typed one stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/qtestLinks.test.ts#L272 ]
