# Aha! import

A feature can be imported from Aha!: the PM pastes the record's link from
Aha! — `https://acme.aha.io/features/APP-123`, the address Shift+L copies —
and the app reads the feature with the user's Aha! API key, writes it as a
new `.feature.md` with the requirements as scenarios, and links the feature
and each scenario to its Aha! record the way the links are added by hand
(external-links/aha-and-figma-links.feature.md). Aha! holds descriptions as
HTML, never Markdown, so they are converted on the way in.

The key is kept per site on this device, like a Confluence or qTest token
(external-links/confluence-wiki.feature.md), and never in the data repo.
Nothing is written to Aha!.

Not in this version: pushing a feature from inside Aha! (its "Send to"
integration webhook needs an endpoint reachable from the internet, so it
belongs to the hosted app's gateway), writing the spec's url back into an
Aha! custom field, importing an epic or idea, and importing the same record
again into the feature it made.

## S1: An Aha! API key is kept per site on this device @v1 [proposed]

- Given: Settings → Aha!
- When: the user enters the Aha! site's url (or any record url of it) and an
  API key — made in Aha! under the avatar → Settings → Personal → Developer
  → API keys — and presses "Save key"
- Then: the key is checked against the site before it is kept: Aha! says
  who it signs in as, and a key Aha! refuses, or that signs in as nobody, is
  not kept and the form says why
- and the site is listed with the user the key signs in as, never the key
  itself, with a Remove button
- and saving a key for a site that already has one replaces it
- and the key is kept on this device only: an owner-only file, or the OS
  key store in the desktop app — never settings.json or the data repo

[test: a key Aha! accepts is kept, owner-only, and listed without the key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L164 ]
[test: a key Aha! refuses, or that signs in as nobody, is not kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L174 ]
[test: a blank key or a url that is no site is refused before Aha! is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L184 ]
[test: a saved key replaces the site's last one, and can be removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L190 ]
[test: lists each site with who its key signs in as, never the key, each with Remove : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaDialogs.test.tsx#L59 ]
[test: offers the url and key fields, the key as a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaDialogs.test.tsx#L67 ]
[test: is the origin of the site url, or of any record url of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L82 ]
[test: two urls are the same site by origin, whatever the case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L88 ]

## S2: Import a feature by its url @v1 [proposed]

- Given: this device holds a key for the Aha! site
- When: the user picks "Import from Aha!…" — the folder's ⋯ menu, the button
  beside "New feature", or the palette — pastes the feature's url, or types
  its reference number (APP-123) against a site with a key, and presses
  Import
- Then: a new feature file lands in that folder, named after the Aha!
  feature, its description the Aha! description as markdown, with a first
  line naming the record, its status and its release: `_Imported from Aha!
  [APP-123](url) — In design, release "2026.4"._`
- and the file is committed, and opened
- and the toast says what was imported and offers "Open in Aha!"
- and Import waits for a feature url on a site with a key, and says what is
  missing

[test: names the site and the record: a feature, an epic, a requirement, an idea : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L16 ]

[test: creates the feature in the chosen folder from the Aha! record, its requirements as scenarios : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L201 ]
[test: a bare reference number is read against the site named with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L258 ]
[test: a name is one line with no folder separator; an empty one is the reference number : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L329 ]
[test: the status and release ride on the import line when Aha! gives them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L364 ]
[test: a modal dialog with the feature url, the two scopes and Import : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaDialogs.test.tsx#L18 ]
[test: Import waits for a feature url on a site with a key, and says what is missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaDialogs.test.tsx#L34 ]
[test: a site to read a bare reference number against is offered only with several keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaDialogs.test.tsx#L48 ]
[test: reads the reference in any case, with a trailing slash, and on a loopback stub : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L34 ]
[test: is not a site url, a list url, a non-url or another scheme : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L46 ]
[test: is read against the chosen site, its kind from its shape : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L62 ]
[test: is refused without a site, or when it is no reference number : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaUrl.test.ts#L74 ]

## S3: Requirements become scenarios @v1 [proposed]

- Given: the Aha! feature has requirements
- When: it is imported with "Requirements as scenarios", the default
- Then: each requirement is a scenario of the new feature, `## <name> @v1
  [proposed]`, its description the scenario's body as markdown, in Aha!'s
  order
- and a requirement's name loses what the heading syntax would misread — a
  leading `S3:` or a trailing `@v2 [published]` — and two requirements with
  one name are told apart
- and requirements Aha! lists without their descriptions are read from
  their own endpoint
- and "Description only" leaves the requirements out

[test: the feature links its Aha! record and each scenario its requirement, on the document only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L231 ]
[test: description only leaves the requirements out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L247 ]
[test: requirements the feature answer lists without descriptions are read from their own endpoint : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L266 ]
[test: a requirement's name loses what the heading syntax would misread, and repeats are told apart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L337 ]

## S4: The feature links its Aha! record @v1 [proposed]

- Given: a feature imported from Aha!
- Then: the feature carries an Aha! link to the record, and each scenario
  one to its requirement, titled with the reference number — on the
  feature's document only, never in the `.feature.md`, like links added by
  hand
- and Aha! links are switched on, so the chips and fields show without a
  trip to Settings
- and when the links cannot be written the feature is still there, and the
  answer says so

[test: the feature links its Aha! record and each scenario its requirement, on the document only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L231 ]

## S5: Descriptions come in as markdown @v1 [proposed]

- Given: an Aha! description — HTML with paragraphs, headings, lists,
  emphasis, links, images, code and tables
- When: it is imported
- Then: it reads as the markdown the editor shows: paragraphs, `###` and
  deeper headings (never the feature's `#` or a scenario's `##`), bullet and
  numbered lists, bold and italic, links, fenced code, pipe tables
- and an image becomes a link to it, since Aha! attachments are behind a
  sign-in
- and nothing in it is read as part of the spec's own syntax: a line that
  would start a scenario, a test link or a ticket link is escaped, and a
  "Tests" heading is demoted

[test: a description keeps nothing the feature parser would lift out of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L353 ]
[test: paragraphs are separated by a blank line, a br breaks the line inside one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L11 ]
[test: every level is demoted to at least three hashes, deeper ones kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L35 ]
[test: an h2 named like a scenario never becomes a scenario heading : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L41 ]
[test: nested bullet lists indent by two spaces per level : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L47 ]
[test: a numbered list counts from its start, a nested one from one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L53 ]
[test: bold, italic, strike and underline : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L77 ]
[test: a link is a markdown link; a bare url, a relative one or none stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L91 ]
[test: an image is a link to it, named by its alt, never an embedded picture : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L97 ]
[test: a pre block is fenced and keeps its whitespace and newlines : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L114 ]
[test: a regular table is a pipe table with the first row as its header : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L131 ]
[test: an irregular table writes each cell on its own line, rows apart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L140 ]
[test: named and numeric entities are decoded, unknown ones left as typed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L149 ]
[test: script, style, head and comments are dropped with their content : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L156 ]
[test: a line that would read as a heading, quote, item or rule is escaped : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L174 ]
[test: a line starting with a test link is escaped, text in code is not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L180 ]
[test: converts to the markdown the editor reads : https://github.com/SpecDriven/specdriven-app/blob/main/tests/html.test.ts#L187 ]

## S6: What cannot be imported says why @v1 [proposed]

- Given: the import dialog
- When: there is no key for the site, Aha! refuses the key, the record does
  not exist, the url names an epic, an idea or a requirement, or a feature
  file of that name is already there
- Then: the dialog shows one sentence saying which — the file already there
  by its name — and nothing is written
- and the hosted app, which keeps no device secrets, offers none of this:
  no Aha! section in Settings, and the routes refuse

[test: what cannot be imported says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L276 ]
[test: a key Aha! no longer accepts is reported as such : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L300 ]
[test: the hosted app refuses Aha! altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ahaImport.test.ts#L308 ]
