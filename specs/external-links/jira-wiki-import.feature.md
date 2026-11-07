# Import from Jira/Wiki

A feature can be proposed from what a team already wrote: up to five Jira
issues and up to five Confluence pages, named by their urls. The app reads
them with the personal access tokens this device holds, hands the text to an
agent plugin's one-shot turn — Claude or Codex, whichever is installed, the
user choosing when both are — and shows the proposed title and up to fifteen
scenarios for review. "Create feature" writes the kept ones as proposed
scenarios into a new `.feature.md` in the chosen folder and opens it. When
the sources describe more than fifteen scenarios the dialog says so and asks
before fetching the rest.

Jira and Confluence are each switched on in Settings, off until then. With
both on the importer reads either kind in one dialog; with only one on it
reads only that kind, and the ⋯ menu item names it ("Import from Jira
here…" / "Import from Confluence here…"); with neither on there is no import
item at all.

Jira is Server / Data Center (REST API 2), reached with a personal access
token kept per site on this device like the Confluence token
(external-links/confluence-wiki.feature.md) — never in settings.json or the
data repo. Nothing is written to Jira or Confluence.

Not in this version: Jira Cloud (an API token with the user's email, and
descriptions in Atlassian Document Format), linking the feature or its
scenarios back to the issues as ticket links, and importing the same sources
again into the feature they made.

## S1: A Jira token is kept per site on this device @v1 [proposed]

- Given: Settings → Jira
- When: the user enters the Jira site's url (or any issue url of it) and a
  personal access token — made in Jira under the profile → Personal Access
  Tokens — and presses "Save token"
- Then: the token is checked against the site before it is kept: Jira says
  who it signs in as, and a token Jira refuses, or that names nobody, is not
  kept and the form says why
- and the site is listed with the user the token signs in as, never the
  token itself, with a Remove button
- and saving a token for a site that already has one replaces it
- and the token is kept on this device only: an owner-only file, or the OS
  key store in the desktop app — never settings.json or the data repo

[test: a token Jira accepts is kept, owner-only, and listed without the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L188 ]
[test: a token Jira refuses, or that names nobody, is not kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L201 ]
[test: a blank token or a url that is no site is refused before Jira is asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L211 ]
[test: a saved token replaces the site's last one, and can be removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L217 ]
[test: lists each site with who its token signs in as, never the token, each with Remove : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L263 ]
[test: offers the url and token fields, the token as a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L272 ]
[test: a bare site url, or any issue url of it, normalized : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L47 ]
[test: two urls are the same site by origin and context path, whatever the case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L54 ]

## S2: The sources: up to five issues and five pages @v1 [proposed]

- Given: a folder's ⋯ menu, with "Import from Jira/Wiki here…"
- When: the user pastes a Jira issue url — `https://jira.acme.com/browse/ABC-123`,
  the project's issue page, or a board's selected issue — or a Confluence
  page url, and presses Add
- Then: the url joins the list as a Jira or Confluence row, named by the
  issue key or the page, each with Remove
- and Add waits for an issue or page url on a site with a token, and says
  what is missing: not such a url, already in the list, no token for that
  site, or the fifth issue or page already there
- and a Jira issue is read with its rendered description, its other text
  fields (Acceptance Criteria, say) by name, its sub-tasks and comments; a
  Confluence page as its export view — all converted to the markdown the
  editor reads, long ones cut to a readable length

[test: reads /browse/KEY, the project's issue page and a board's selected issue : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L9 ]
[test: keeps a context path and a port in the site's base, and upper-cases the key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L23 ]
[test: is null for what names no issue : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L30 ]
[test: is built the way Jira shows it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraUrl.test.ts#L41 ]
[test: is a Jira issue, a Confluence page, or nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L34 ]
[test: at most five of each kind, and at least one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L49 ]
[test: a modal dialog with the url field, Add and Analyze, naming the folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L77 ]
[test: a url is a Jira issue or a Confluence page row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L147 ]
[test: Add waits for an issue or page url on a site with a token, within the limits : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L162 ]
[test: refuses more than five of a kind, an unknown url, and a site without a token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L228 ]
[test: reads the issues and the page as markdown into one turn, and answers the proposal : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L253 ]

## S3: The agent that reads them @v1 [proposed]

- Given: the import dialog
- When: an agent plugin that answers one-shot turns is installed and its CLI
  is on PATH
- Then: with one such agent the dialog names it ("Analyzed by Claude"); with
  two or more, an "Analyze with" choice offers them, the last choice kept
  for next time; with none, Analyze is off and the dialog says to install
  the Claude or Codex agent plugin
- and the turn is one no-tools turn held to a JSON answer — a title, at most
  fifteen scenarios, and the names of the rest — with no session record and
  nothing to ask permission for
- and the Codex plugin answers such turns through `codex exec`, read-only
  and ephemeral, the answer's shape given as a schema

[test: the agent is named with one, chosen with two, missed with none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L90 ]
[test: the agents offered are the available ones that answer one-shot turns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L142 ]
[test: needs an agent that answers one-shot turns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L244 ]
[test: asks the chosen agent, and passes the names already shown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L290 ]
[test: is held to a title, at most fifteen scenarios and the names of the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L74 ]
[test: the provider advertises one-shot turns and answers them through codex exec : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/oneShot.test.ts#L101 ]
[test: runs `codex exec` read-only, ephemeral, on the work dir, with the schema and out file; the prompt on stdin : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/oneShot.test.ts#L106 ]
[test: a non-zero exit is an error carrying the stderr tail : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/oneShot.test.ts#L164 ]
[test: a child that never answers is killed when the time is up : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/oneShot.test.ts#L198 ]

## S4: A proposed title and scenarios, reviewed before the feature is written @v1 [proposed]

- Given: the sources are listed and an agent can read them
- When: the user presses Analyze
- Then: the agent proposes a feature title and up to fifteen scenarios, each
  a name and a body of Given/When/Then bullets, each naming the issue or
  page it rests on
- and the dialog shows them for review: the title to edit, each scenario
  with a checkbox to keep it, its name to edit and its body to show; Back
  returns to the sources
- and "Create feature" writes the kept scenarios as `## S<n>: <name> @v1
  [proposed]`, numbered in the agent's order of importance, into a new
  feature file in the folder, named after the title, a first line
  naming the sources — `_Imported from Jira [ABC-123](url) and Confluence
  [Page](url)._` — and each scenario's sources on its last line; the file is
  committed and opened, and the toast says what was imported
- and a name loses what the heading syntax would misread, repeats are told
  apart, and no body line reads as a heading, a test link or a ticket link
- and the Jira issues it was read from are the feature's tickets, one
  `[Jira:ABC-123](url)` line each above the sources line, at most five
  (external-links/project-management-jira-linear-etc.feature.md "Several
  tickets on a feature"); a Confluence-only import links none

[test: shows each source with its url and text, and what has been proposed already : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L64 ]
[test: a structured answer, or JSON in the words, is a proposal; anything else is none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L81 ]
[test: loses what the heading syntax would misread, and is one line : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L116 ]
[test: the Jira issues as tickets, the sources under the title, each scenario proposed and numbered in the agent's order, with its body and where it came from : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L131 ]
[test: a key read from the url when the title is not one; repeats dropped; no more than five : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L202 ]
[test: a wiki-only import links no ticket : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L219 ]
[test: names are cleaned and repeats told apart; a body keeps nothing the parser would lift out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L180 ]
[test: the title, each scenario kept or dropped with its body on show, and Create with the count : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L233 ]
[test: Create waits for a title and a kept scenario : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L255 ]
[test: writes the kept scenarios as proposed into the folder, naming the sources : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L469 ]

## S5: More than fifteen is asked about before continuing @v1 [proposed]

- Given: the sources describe more scenarios than one answer holds
- When: the agent answers with fifteen and names the rest
- Then: the review step says how many more the sources describe, and which,
  with Continue
- and Continue asks the agent again, telling it which scenarios have been
  proposed already, and appends the next batch — until nothing is left
- and an answer with more than fifteen scenarios in it is cut to fifteen

[test: more than fifteen is said, with Continue; nothing when the sources hold no more : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L248 ]
[test: more than fifteen scenarios in one answer are cut to fifteen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L108 ]
[test: asks the chosen agent, and passes the names already shown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L290 ]

## S6: What cannot be imported says why @v1 [proposed]

- Given: the import dialog
- When: a site has no token, Jira or Confluence refuses the token, an issue
  or page is not there, no agent can answer, the agent's turn fails or
  answers with something else, or a feature file of that name is already in
  the folder
- Then: the dialog shows one sentence saying which — the url that failed,
  the agent that could not, the file already there by its name — and
  nothing is written

[test: what Jira or Confluence refuses says which url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L337 ]
[test: a text-only JSON answer parses too; a failed turn or an odd answer says who could not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L319 ]
[test: a feature already there is reported, not replaced; a title or scenarios are needed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L479 ]

## S7: Off in the demo and the hosted app @v1 [proposed]

- Given: the demo, or the hosted app
- When: the user opens a folder's ⋯ menu or Settings
- Then: there is no "Import from Jira/Wiki here…" and no Jira section, and
  the server refuses the Jira and import routes — the demo has no token and
  no agent, the hosted app keeps no device secrets

[test: the hosted app refuses Jira and the import altogether : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L493 ]
[test: demo mode refuses the import : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L508 ]

## S8: The model the agent answers with @v1 [proposed]

- Given: the import dialog, with an agent that can read the sources
- When: the user looks under the agent
- Then: a Model field, a dropdown of the models that agent offers — what
  its sign-in lists (the account's models, a proxy's `GET /v1/models`, its
  CLI's aliases), narrowed to the names the agent's own Settings section
  lists for it (Settings → Claude's "Claude models") — holds the agent's own
  default (Settings → Claude's), else the first; Analyze asks the agent with
  the model the field shows, and choosing another agent fills in that
  agent's; an agent that offers no models gets its Settings list
  alone, and with none listed either the field offers none and the agent
  answers with its own default
- and a model name that is not one (spaces, a leading dash) is said under
  the field and refused by the server; the Claude plugin passes the model to
  Claude Code, the Codex plugin to `codex exec -m`

[test: a Model field of the models the agent offers, its default chosen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L102 ]
[test: a model name is a CLI's name for one; anything else is said : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L133 ]
[test: the model asked for reaches the agent; none leaves the agent's own; an odd name is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraImport.test.ts#L306 ]
[test: a model asked for is the query's model : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L60 ]
[test: no model asked for and none in Settings leaves Claude Code's own default : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L79 ]
[test: no model asked for and none in Settings is the first the data repo lists for Claude : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L85 ]
[test: a model asked for goes to codex exec as -m; none asked for leaves Codex's own default : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/oneShot.test.ts#L144 ]
