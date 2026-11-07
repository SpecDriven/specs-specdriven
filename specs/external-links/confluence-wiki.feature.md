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

[test: a token Confluence accepts is kept, owner-only, and listed without the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L270 ]
[test: a token Confluence refuses, or answers as anonymous, is not kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L280 ]
[test: a blank token or a url that is no wiki is refused before Confluence is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L290 ]
[test: a saved token replaces the wiki's last one, and can be removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L296 ]
[test: lists each wiki with who its token signs in as, never the token, each with Remove : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L217 ]
[test: offers the url and token fields, the token as a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L225 ]
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

[test: each scenario leads with its number, an anchor linking to itself, as in the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L152 ]

## S4: Publish a feature to Confluence @v1 [proposed]

- Given: a feature is open and this device holds a token for the wiki
- When: the user picks "Publish to Confluence…" in the feature's ⋯ menu,
  pastes the parent page's url, chooses "All scenarios" or "Published
  scenarios only", and presses Publish
- Then: the feature becomes a page titled with the feature's name, a child of
  the parent page, in the parent's space
- and the feature links the page: a Confluence chip in the feature header
  opens it, and the toast offers "Open page"
- and the dialog offers, next time, the parent page that feature was last
  published under: each feature keeps its own, as features may live under
  different parent pages, recorded in the feature's Automerge document only
  — never in the `.feature.md` or this device's settings
- and Publish waits for a page url on a wiki with a token, and says what is
  missing; what Confluence refuses (no such page, a token it no longer
  accepts, a space the token cannot write to) is shown as one sentence

[test: creates the page under the parent, links it from the feature and remembers the parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L307 ]
[test: each feature remembers the parent it was last published under : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L339 ]
[test: a /display/ parent url is found by its title : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L412 ]
[test: what cannot be published says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L453 ]
[test: a token Confluence no longer accepts is reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L468 ]
[test: a modal dialog with the parent url, the two scopes and Publish : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L26 ]
[test: Publish waits for a page url on a wiki with a token, and says what is missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L90 ]
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

[test: publishing again updates the page it links, as a new version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L361 ]
[test: the linked page moves under a new parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L377 ]
[test: a linked page that was deleted is found again by its title, or created anew : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L385 ]
[test: a page of the same title under the parent is updated rather than duplicated : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L394 ]
[test: a page of the same title elsewhere in the space is not taken over : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L402 ]
[test: a feature published before says publishing again updates its page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L48 ]
[test: a page url lies on its wiki, and not on another : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceUrl.test.ts#L79 ]

## S6: Publishing is off in the hosted app @v1 [proposed]

- Given: the hosted app
- When: the user opens a feature's ⋯ menu or Settings
- Then: there is no "Publish to Confluence…" and no Confluence section, and
  the server refuses the Confluence routes — the hosted app keeps no device
  secrets

[test: the hosted app refuses Confluence altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L502 ]

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
[test: with Publish external links off the file has no line, but the feature keeps the link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L445 ]

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
- and an image on the web is shown by its url; one stored in the app, which
  Confluence cannot reach by url, is uploaded as the page's attachment and
  shown from there, named by the bytes it holds: its stored name while the
  file is as it was stored (the name carries a hash of it), else its stem
  with the hash of the bytes it holds now — a file edited in place keeps its
  name in the repo, and the page must show the edit. An attachment already
  there under the name is the same file and is left alone. A failed upload
  leaves the page published, with a warning
- and everything taken from the feature is escaped, and a link to anything
  but a web, mail or relative address stays text

[test: description, then each scenario with its steps and tests, no status or version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L131 ]
[test: published only leaves the rest out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L143 ]
[test: a draft shows no status or version, and a scenario name is escaped : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L165 ]
[test: says so when no scenario is in scope : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L173 ]
[test: escapes text, attributes included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L41 ]
[test: bold, italic, strike and code : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L46 ]
[test: links only to safe destinations; a javascript url stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L68 ]
[test: a bare url is a link, its trailing punctuation left outside : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L75 ]
[test: a test reference is a link named after the test : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L80 ]
[test: an image on the web is embedded by url; one the app stores as the page's attachment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L86 ]
[test: names each stored image a page shows once, in order; escaped text is not one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L223 ]
[test: the images a page shows are uploaded as its attachments, once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L516 ]
[test: a page whose images cannot be attached is still published, with a warning : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L584 ]
[test: several images go up in one upload : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L539 ]
[test: an image edited in place since it was stored is attached again under the name its bytes earn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L555 ]
[test: renames the attachments it is told to, every use of each, and leaves the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L232 ]
[test: a stored name while the bytes are the ones it was given for; its stem with their hash once they differ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/images.test.ts#L63 ]
[test: paragraphs keep their soft breaks : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L98 ]
[test: headings are shifted under the heading they sit in, at most h6 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L102 ]
[test: lists, task items, quotes and rules : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L106 ]
[test: tables, with a header row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L113 ]
[test: fenced code is a code macro, CDATA kept whole : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L119 ]
[test: published only leaves the proposed scenarios out of the page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L420 ]

## S9: Changes made in Confluence are not overwritten unnoticed @v1 [proposed]

- Given: a feature published before, whose page has since been changed in
  Confluence itself — its version there is past the one the app last wrote
- When: the user publishes it again
- Then: the publish stops before writing, and the dialog says the page was
  changed in Confluence since it was last published from here — its version,
  who made it and when, as Confluence reports them — with a link to the page
  and "Publish anyway", which replaces those changes with the feature
- and the page's version is recorded in the feature's document after each
  publish, never in the `.feature.md`, so a teammate's publish recorded
  there is no conflict, and saving the file leaves the record alone
- and a page published before versions were recorded is updated as before,
  and the record starts with that publish
- and each version the app writes carries "Published from SpecDriven" in the
  page's history, so a version made by hand stands out there
- and a page changed between the check and the update is refused by
  Confluence itself, and the publish says so without replacing anything

[test: publishing records the page's version in the feature's document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L611 ]
[test: a page changed in Confluence since the last publish is not overwritten without confirmation : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L623 ]
[test: Publish anyway replaces the changes and records the new version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L644 ]
[test: a version recorded by a teammate's publish is no conflict : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L658 ]
[test: a page published before versions were recorded publishes and starts recording : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L671 ]
[test: the app's versions carry a message in the page history : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L698 ]
[test: saving the file keeps the recorded version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L707 ]
[test: a page changed between the version check and the update is reported, not replaced : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L714 ]
[test: says who changed the page in Confluence and offers to publish anyway : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L64 ]

## S10: A failed publish shows what Confluence answered, and is logged @v1 [proposed]

- Given: a feature is open and this device holds a token for the wiki
- When: a publish fails because Confluence answers with an error (a server
  error such as HTTP 500, a refusal, something that is not JSON) or cannot be
  reached at all
- Then: the dialog shows the failure as one line, and clicking it opens a
  detail box: the request (method and url, never the token), Confluence's
  status, its content type, when it failed, and what Confluence answered —
  an HTML error page read as text, a long answer cut to a readable length
- and a wiki that could not be reached says there was no answer, and why
- and the same failure is recorded in the app's log under the "confluence"
  component — server errors and unreachable wikis as errors, refusals
  Confluence explains as warnings — and "Open in Observability" in the detail
  box opens the Observability page narrowed to it
- and a failure the app finds before asking Confluence (no token, a url that
  names no page) stays a plain sentence with no detail box

[test: answers the request, the status and Confluence's answer as details : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L756 ]
[test: is recorded in the log under confluence, with what Confluence answered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L775 ]
[test: a refusal Confluence explains is logged as a warning : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L792 ]
[test: an unreachable wiki is logged with the network error, and no status : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L801 ]
[test: a long answer is cut to a readable length : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L815 ]
[test: shows the error as a line that opens the details on click, and closes them again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceFailureDetails.test.tsx#L93 ]
[test: an unreachable wiki says there was no answer, and shows no body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceFailureDetails.test.tsx#L120 ]
[test: an error without details stays plain text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceFailureDetails.test.tsx#L133 ]

## S11: Publish a folder's features to Confluence @v1 [proposed]

- Given: a folder of features, and this device holds a token for the wiki
- When: the user picks "Publish to Confluence…" in the folder's ⋯ menu,
  pastes the parent page's url, chooses "All scenarios" or "Published
  scenarios only", and presses Publish
- Then: each feature under the folder, its subfolders' included, becomes its
  own page under the parent page — flat, in the sidebar's order — exactly as
  publishing it from its own ⋯ menu would: a feature published before
  updates its page, and each feature links its page and remembers the parent
- and the dialog lists the features, each checked, with whether it updates
  its page or makes a new one; the user can leave some out
- and the parent offered is the one most of the folder's features were last
  published under
- and each feature's row says how its publish went as it goes: published or
  updated with a link to the page, changed in Confluence since the last
  publish, or what failed — a failure of one feature does not stop the rest
- and pages changed in Confluence are left alone until the user presses
  "Publish anyway", which replaces those pages only
- and a failure every feature would meet — a token Confluence refuses, a
  wiki that does not answer — stops the run there
- and once every checked feature is published the dialog closes and a toast
  says how many pages were published and updated
- and the hosted app and the demo offer no such menu entry, and the server
  refuses the folder's Confluence route there

[test: a folder lists its features, its subfolders' too, with the page each links and its last parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L478 ]
[test: the hosted app refuses Confluence altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L502 ]
[test: a modal dialog with the parent url, the two scopes and each feature checked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L122 ]
[test: waits for the folder's features, and says when there are none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L139 ]
[test: Publish waits for a page url on a wiki with a token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L145 ]
[test: offers the parent most of the folder's features were last published under : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L151 ]
[test: publishes each feature in turn, forcing only the conflicts the user chose to replace : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L166 ]
[test: stops at a failure every feature would meet, a refused token or a wiki with no answer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L190 ]
[test: Publish sends the checked features not yet published in this run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceDialogs.test.tsx#L208 ]

## S12: An agent session can read the wiki @v1 [proposed]

- Given: this device holds a token for a wiki (S1)
- When: a coding agent session starts — in the agent pane, on Claude or Codex
- Then: the app hands the session an MCP server of its own,
  `specdriven-confluence`, with three read-only tools: `get_page` (a
  page's title and body as markdown, by its link or its id), `search_pages`
  (the pages whose text matches words or a phrase, within one space when
  asked) and `list_children` (the pages under a page)
- and the server reads the tokens from the app's own token file on every
  call — a token saved while the session runs counts at once — and no token
  is handed to the agent itself: the agent learns the wikis' urls, never a
  token
- and without a token for any wiki the session gets no such server; the demo
  and the hosted app hand out none
- and the agent's own permission rules cover the tools as they cover any
  other: in a mode that asks, the first call raises the pane's permission
  card

[test: no wiki token, no server : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentMcp.test.ts#L54 ]
[test: a saved token brings the Confluence server, run from the app's source with its token file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentMcp.test.ts#L58 ]
[test: the demo and the hosted app hand out none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentMcp.test.ts#L72 ]
[test: a saved wiki token hands the provider the Confluence MCP server; none, nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L525 ]
[test: three read-only tools, under the server's name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L217 ]
[test: the instructions name the saved wikis, never a token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L208 ]
[test: no token saved is said in so many words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L152 ]
[test: one wiki needs no base; several do, and the message lists them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L156 ]
[test: the MCP servers the app asks for reach the SDK as stdio servers, by name : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1278 ]
[test: the MCP servers the app asks for reach `codex app-server` as -c overrides, never the user's config : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L980 ]

## S13: The agent reads a page by its link @v1 [proposed]

- Given: an agent session with the wiki server
- When: the agent calls `get_page` with a page's link —
  `…/spaces/KEY/pages/ID/Title`, `…/pages/viewpage.action?pageId=ID` or
  `…/display/KEY/Title` — or with a page id, and the wiki's url when several
  wikis are saved
- Then: it gets the page's title, url, id, space and version, then the body
  as markdown — the export view, macros expanded, as "Import from
  Jira/Wiki" reads it — cut at 60,000 characters with a line saying so
- and a page the wiki has not, a link to a wiki without a token, a link that
  names no page, or a token the wiki rejects comes back as the tool's
  refusal in one sentence, never as a broken session

[test: get_page reads a page by its link as markdown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L226 ]
[test: get_page reads a page by id, and by its title url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L239 ]
[test: a page Confluence has not is a refusal, not a crash : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L245 ]
[test: a rejected token is said as Confluence's refusal : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L255 ]
[test: a page url picks its wiki and names the page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L163 ]
[test: an id names a page on the one wiki, or on base : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L173 ]
[test: a long page is cut and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L183 ]
[test: a page says where it is before what it says : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L202 ]

## S14: Searching the wiki, and walking it, from the agent @v1 [proposed]

- Given: an agent session with the wiki server
- When: the agent calls `search_pages` with words or a phrase, a space key
  to narrow to and how many pages at most; or `list_children` with a page's
  link or id
- Then: it gets a list of pages — each a link with its title, id, space and
  who last changed it when the wiki says — the pages whose text matches, in
  the wiki's own ranking (CQL `text ~`), or the pages directly under the
  page, in the wiki's order, following the wiki's pages of results up to the
  limit
- and an empty answer says so in one line

[test: search_pages finds pages by text, within a space when asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L262 ]
[test: list_children lists the pages under a page, following Confluence's pages of results : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L274 ]
[test: hits are a list of links, with who changed them when known : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L189 ]

## S15: A terminal session uses the same server @v1 [proposed]

- Given: the app runs from a checkout, with a wiki token saved
- When: the user registers the server with their agent's command line —
  `claude mcp add specdriven-confluence -- bun <checkout>/src/mcp/confluence.ts`
- Then: that session has the same three tools, reading the same token file
  (`~/.specdriven/confluence-tokens.json`);
  `SPECDRIVEN_CONFLUENCE_TOKENS_FILE` in its environment names another file
- and over stdio nothing the server or the app's modules print reaches the
  agent as protocol: everything the console would say goes to stderr

[test: over stdio, as an agent starts it, the tokens come from the file the environment names : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L313 ]
[test: the environment names it, else the app's own file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L300 ]
[test: a missing or corrupt file is no wikis; a good one its records : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mcpConfluence.test.ts#L305 ]

## S16: The desktop app lends its tokens only while it runs @v1 [proposed]

- Given: the desktop app, which keeps the tokens OS-encrypted
- When: an agent session starts
- Then: the app writes the wikis and their tokens to an owner-only file in
  its own data folder (`confluence-tokens.mcp.json`) and starts the server
  from its own bundle, run as node, pointed at that file
- and the file follows the tokens — one saved or removed rewrites it — and is
  removed when the app quits
- and a session of the web server points the server at the app's own token
  file instead

[test: the desktop app says how to run it and where the tokens are : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentMcp.test.ts#L81 ]
[test: the desktop app's export is the records, tokens included, owner-only, and follows every change : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentMcp.test.ts#L95 ]
