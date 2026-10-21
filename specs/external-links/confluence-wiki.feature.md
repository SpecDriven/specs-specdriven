# Confluence wiki

A feature can be published as a page of a Confluence Server or Data Center
wiki (REST API 7.19), as a child of a parent page the user names by its url —
`https://wiki.acme.com/spaces/ABC/pages/99999/Parent+Page`,
`…/pages/viewpage.action?pageId=99999` or `…/display/ABC/Parent+Page`. The
wiki is reached with a personal access token, one per wiki, kept on the
user's device and never in the data repo.

Once published, the feature links its page the way it links its ticket and
its qTest test case (external-links/qtest-test-cases.feature.md): one
`[Confluence:<page id>](<page url>)` line under the feature's heading, held
in the feature's Automerge document and written into the `.feature.md` while
"Publish external links" is on. Publishing again updates that page.

## S1: A Confluence token is kept per wiki on this device @v1 [proposed]

- Given: Settings → Confluence
- When: the user enters the wiki's url (or any page url of it) and a personal
  access token, and presses "Save token"
- Then: the token is checked against the wiki's current user before it is
  kept; a token the wiki refuses, or answers as the anonymous user, is not
  kept and the form says why
- and the wiki is listed with the user the token signs in as, never the token
  itself, with a Remove button
- and saving a token for a wiki that already has one replaces it
- and the token is kept on this device only: an owner-only file, or the OS
  key store in the desktop app — never settings.json or the data repo

[test: a token Confluence accepts is kept, owner-only, and listed without the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L193 ]
[test: a token Confluence refuses, or answers as anonymous, is not kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L203 ]
[test: a blank token or a url that is no wiki is refused before Confluence is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L213 ]
[test: a saved token replaces the wiki's last one, and can be removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L219 ]
[test: lists each wiki with who its token signs in as, never the token, each with Remove : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L70 ]
[test: offers the url and token fields, the token as a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L78 ]
[test: a bare wiki url, or any page of it, normalized : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L66 ]
[test: two urls are the same wiki by origin and context path, whatever the case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L73 ]

## S2: Confluence is reserved for the page link @v1 [proposed]

- Given: the ticket fields, or Settings → Project management
- When: the user names "Confluence" as the ticket's tool
- Then: it is refused — a ticket written under Confluence would read back as
  the published page

[test: Confluence can't be a ticket tool, where it would read back as the page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L106 ]

## S3: Each scenario is numbered and can be linked to @v1 [proposed]

- Given: a feature being published
- When: the page is written
- Then: each scenario's heading leads with its number, S1, S2, …, as on its
  card in the app
- and the number is an anchor on the page and a link to itself, so a reader
  clicks it to get a deep link to that scenario to share
- and a page of the published scenarios only keeps the numbers the app
  shows, so S3 is the same scenario in the app and in the wiki
- and a draft, which has no number in the app, has none on the page

[test: each scenario leads with its number, an anchor linking to itself, as in the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L136 ]

## S4: Publish a feature to Confluence @v1 [proposed]

- Given: a feature is open and this device holds a token for the wiki
- When: the user picks "Publish to Confluence…" in the feature's ⋯ menu,
  pastes the parent page's url, chooses "All scenarios" or "Published
  scenarios only", and presses Publish
- Then: the feature becomes a page titled with the feature's name, a child of
  the parent page, in the parent's space
- and the feature links the page: a Confluence chip in the feature header
  opens it, and the toast offers "Open page"
- and the dialog offers the last parent page used on this device next time
- and Publish waits for a page url on a wiki with a token, and says what is
  missing; what Confluence refuses (no such page, a token it no longer
  accepts, a space the token cannot write to) is shown as one sentence

[test: creates the page under the parent, links it from the feature and remembers the parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L230 ]
[test: a /display/ parent url is found by its title : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L310 ]
[test: what cannot be published says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L350 ]
[test: a token Confluence no longer accepts is reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L365 ]
[test: a modal dialog with the parent url, the two scopes and Publish : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L16 ]
[test: Publish waits for a page url on a wiki with a token, and says what is missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L54 ]
[test: reads the /spaces/KEY/pages/ID form, title or not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L10 ]
[test: keeps a context path and a port in the wiki's base : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L23 ]
[test: reads viewpage.action?pageId= and the short /pages/ID : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L30 ]
[test: reads /display/KEY/Title as a title to look up, + and %20 as spaces : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L38 ]
[test: is null for what names no page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L49 ]

## S5: Publishing again updates the page @v1 [proposed]

- Given: a feature that was published before
- When: the user publishes it again
- Then: the page it links gets a new version rather than a second page;
  under a different parent page, the page moves there
- and when the linked page was deleted, or the feature links none, a page of
  the same title under the parent is updated, else a new one is created
- and a page of the same title elsewhere in the space is not taken over: the
  publish is refused, naming that page

[test: publishing again updates the page it links, as a new version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L259 ]
[test: the linked page moves under a new parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L275 ]
[test: a linked page that was deleted is found again by its title, or created anew : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L283 ]
[test: a page of the same title under the parent is updated rather than duplicated : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L292 ]
[test: a page of the same title elsewhere in the space is not taken over : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L300 ]
[test: a feature published before says publishing again updates its page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L38 ]
[test: a page url lies on its wiki, and not on another : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L79 ]

## S6: Publishing is off in the hosted app @v1 [proposed]

- Given: the hosted app
- When: the user opens a feature's ⋯ menu or Settings
- Then: there is no "Publish to Confluence…" and no Confluence section, and
  the server refuses the Confluence routes — the hosted app keeps no device
  secrets

[test: the hosted app refuses Confluence altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L375 ]

## S7: The Confluence line in the feature file @v1 [proposed]

- Given: a feature published to Confluence
- When: the user looks at its `.feature.md`
- Then: a `[Confluence:<page id>](<page url>)` line sits under the heading,
  after the ticket and qTest lines
- and with "Publish external links" off the file has no such line, but the
  feature's document keeps the link and the chip still shows
- and the line belongs to the feature: under a scenario heading it is
  ordinary content
- and teammates' edits to the line merge like the ticket link's: replaced
  whole, the side that changed it wins

[test: is lifted into confluence, beside the ticket and the test case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L70 ]
[test: survives a round trip, written after the ticket and test case whatever the order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L78 ]
[test: is the page and never the ticket, whatever its case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L84 ]
[test: under a scenario heading it is content, not the scenario's ticket : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L93 ]
[test: is left out with the other links when links are not written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L101 ]
[test: holds the page link and projects it back, or leaves it out with links off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L113 ]
[test: reconcile sets, replaces and clears it — and a no-op stays one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L123 ]
[test: a three-way merge keeps the side that changed it, ours when both did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L143 ]
[test: writes the line while links are published; with links off the editor still sees it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L156 ]
[test: a file in the other form is not drift; an edited page link is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceLinks.test.ts#L168 ]
[test: with Publish external links off the file has no line, but the feature keeps the link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L342 ]

## S8: What the page contains @v1 [proposed]

- Given: a feature being published
- When: the page is written
- Then: it holds the feature's description, then each scenario in scope under
  its number and name, with its steps and links to its tests, named after
  the test
- and a scenario's status and version are not shown on the page
- and "Published scenarios only" leaves the proposed and draft ones out
- and markdown reads as it does in the app: emphasis, lists and task items,
  quotes, tables, links, and code blocks as Confluence code blocks
- and an image on the web is shown; one stored in the app, which Confluence
  cannot reach, is replaced by its alt text
- and everything taken from the feature is escaped, and a link to anything
  but a web, mail or relative address stays text

[test: description, then each scenario with its steps and tests, no status or version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L115 ]
[test: published only leaves the rest out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L127 ]
[test: a draft shows no status or version, and a scenario name is escaped : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L149 ]
[test: says so when no scenario is in scope : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L157 ]
[test: escapes text, attributes included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L41 ]
[test: bold, italic, strike and code : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L46 ]
[test: links only to safe destinations; a javascript url stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L52 ]
[test: a bare url is a link, its trailing punctuation left outside : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L59 ]
[test: a test reference is a link named after the test : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L64 ]
[test: an image on the web is embedded; one the app stores is its alt text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L70 ]
[test: paragraphs keep their soft breaks : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L82 ]
[test: headings are shifted under the heading they sit in, at most h6 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L86 ]
[test: lists, task items, quotes and rules : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L90 ]
[test: tables, with a header row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L97 ]
[test: fenced code is a code macro, CDATA kept whole : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L103 ]
[test: published only leaves the proposed scenarios out of the page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L318 ]
