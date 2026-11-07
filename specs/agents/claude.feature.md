# Integration with Claude agent

**Created by:** vydra

**Updated by:** David Vydra

Claude Agent can use either a Claude Account and an API Proxy.

## S1: API proxy settings are filled in from Claude Code's settings @v1 [proposed]

- Given: Claude Code's own settings (`~/.claude/settings.json`, the data
  folder's `.claude/settings.json` or `.claude/settings.local.json`) or the
  environment carry `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN`
- When: the Settings page's Claude section is on "API proxy"
- Then: it shows the detected address and where it came from, with "Use
  detected" to make that address and token the app's own
- and the token is kept in the app's own owner-only store (OS-encrypted in
  the desktop app), never shown on the page, and never sent back to it
- and a pasted token a request cannot carry — with a space, a line break or
  a letter outside ASCII in it, such as a pasted "Bearer sk-…" — is refused
  with why, and no answer to the page quotes a token, however a request
  with it failed
- and Claude Code's own files are never edited
- and "Check connection" asks the proxy for its model list and reports OK,
  a refused token, or an unreachable proxy
- and the "Proxy timeout" of agent-pane.feature.md lives in this section

Claude Code applies the `env` block of its settings files over the
environment it is started with, so the app passes its choice as the
`--settings` flag, which wins over every file; empty values there clear a
proxy for the account sign-in.

[test: detectClaudeProxy reads the environment, then each settings file, the later winning : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L127 ]
[test: a store round-trips through an owner-only file under the app's home, and is cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L39 ]
[test: a corrupt or empty file reads as no secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L58 ]
[test: a pasted token never comes back in Check proxy's words or the model list's error, whatever it holds : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/signIn.property.test.ts#L708 ]
[test: whatever a failed request says, Check proxy and the model list say it without the token : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/signIn.property.test.ts#L732 ]

## S2: Choose how Claude signs in @v1 [published]

**Updated by:** David Vydra

Choose  "Sign in with": "Claude account (email login)" 

![image](/api/images/image-f2cf34a8.png)

or "API Proxy"

![image](/api/images/image-d17863c2.png)

Click 'Check Connection' to validate connectivity to the LLM API service.

The choice saves itself the moment it changes, to
  .specdriven/settings.json, and applies to every Claude the app starts
  afterwards — the coding agent pane, a task estimate, a self-fix

Note: this section is not offered in the demo or the hosted app, which run no Claude of their own

[test: the stored choice is normalized; a url must be http(s) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L156 ]
[test: the sign-in Settings chose reaches Claude as a --settings override (src/auth.ts) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1262 ]

## S3: Claude cli options @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

In the target repo, locate folder '.specdriven'
If it contains a file called specdriven-config.json
Check for:
{
  "claude-cli": "--settings .claude/hai-claude-config.json --dangerously-skip-permissions"
}

https://www.testmuai.com/support/docs/agent-assurance-overview/

## S4: Create a prompt from selected specs @v1 [published]

**Created by:** vydra

**Updated by:** vydra

Given: a code session has been created with at least one spec in it
When: we start a code gen session with Agent (Claude)
Then: we generate a prompt and pass it to the Claude Cli to execute and exit

If error code, display the error to the user in an error box.

## S5: Create and name a code gen session @v1 [published]

**Created by:** vydra

**Updated by:** vydra

- Select one or more specs
- 'Start session' (click button)
- Name the session

## S6: Create and name a code gen session @v2 [published]

**Created by:** vydra

**Updated by:** vydra

- Select a whole spec with the checkbox on its sidebar entry
- Select single scenarios with the checkbox on their scenario cards
- A spec whose scenarios are partly selected shows a dash in the sidebar;
  selecting every scenario of a spec is the same as selecting the spec
- The selection holds while moving between specs, and what it adds up to is
  shown next to 'Start session'
- 'Start session' (click button)
- Name the session
- A spec cut down to selected scenarios is marked as such in the prompt

## S7: Sign in to Claude with an email @v1 [proposed]

- Given: the Claude section on "Claude account"
- Then: it says who the Claude CLI is signed in as, or that nobody is
- When: an email is entered and "Sign in…" pressed
- Then: the CLI's browser sign-in opens, prefilled with that email; while it
  is under way the page says to finish signing in in the browser window
  Claude Code opened and that nothing is pasted into the app; "Cancel"
  stops it
- and the sign-in finishes in the browser alone: once it is approved there
  the page shows the account signed in — there is no code field, no link
  to a sign-in page of the app's own (the page the CLI prints for a browser
  that did not open shows a code meant for a terminal), and the app opens
  no browser page beside the CLI's
- and every step of the sign-in — its start, the page the CLI printed, the
  CLI's exit, the end, and a sign-in ended without a login with the CLI's
  last words — is in the app's log under "agent-claude"
- and the section says plainly who is signed in — "Signed in as" the
  account's email — or "Not signed in", with the email field and "Sign in"
  only while nobody is
- and "Sign out" signs the CLI out, after which the section says "Not
  signed in" and offers "Sign in" again; another account takes a sign-out
  first, there is no "Switch account"
- and "Check sign-in" asks the CLI afresh who is signed in and, with an
  account, starts a Claude Code under it for the models it offers — saying
  what it found, who or nobody, and why Claude Code could not answer when it
  could not — and "Default model" shows that fresh list from then on
- and once a sign-in is done, "Default model" asks for the account's models
  again, even for the account that was signed in before
- and with no Claude Code on this machine and none published for its
  platform, the section says to install the claude command instead
- and a bearer token in Claude Code's own settings files is not a sign-in:
  with the Claude account chosen the section says "Not signed in" while
  only such a token is there — the CLI is asked the way every Claude the
  app starts sees it, with the proxy and keys set aside — and the account
  is the one the CLI names, never a profile left in its global config by
  another Claude
- and "Check sign-in" reports what the Claude Code it started found: with
  no account, token or key it says "Not signed in", whatever `claude auth
  status` said, and lists no models

[test: an API key is not an account: the section says so and asks for a sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/accountSection.test.tsx#L38 ]
[test: the check's finding is one sentence: the account, and whether Claude Code lists its models : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/accountSection.test.tsx#L50 ]

[test: a bearer token in Claude Code's settings file is not a sign-in: the CLI is asked with the account's --settings : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L323 ]

[test: a sign-in finishes in the browser alone: a CLI that also offers to take a code gets none, and the ended sign-in is logged : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L464 ]
[test: there is no route for a sign-in code: the browser finishes the sign-in, the page only polls : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L780 ]

[test: parseLoginStatus reads what claude auth status --json says : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L298 ]
[test: while the browser sign-in is under way, the section says to finish it there and offers only Cancel — no code field, no link : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/accountSection.test.tsx#L75 ]

## S8: Sign in to Claude without Claude Code installed @v1 [proposed]

- Given: no `claude` command on PATH
- Then: the agent pane says Claude Code is not set up, with "Set up Claude",
  which opens Settings at the Claude section
- and the Claude section asks for the email of the Claude account, saying
  the app downloads Claude Code first
- When: an email is entered and "Sign in…" pressed
- Then: the app downloads this platform's Claude Code from the npm
  registry — the binary the Claude Agent SDK publishes — and the page shows
  how far it has got; "Cancel" stops the sign-in
- and the download is kept only when it matches the checksum pinned in the
  plugin for that SDK version, under ~/.specdriven/claude-code
- and then the browser sign-in opens as when the CLI is installed
- and every Claude the app starts runs that Claude Code from then on, while a
  `claude` on PATH, when there is one, still comes first
- and a turn that fails because Claude is not signed in offers "Open
  Settings" beside its hint

[test: the pins are for the SDK version package.json depends on, one per platform : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/binary.test.ts#L46 ]
[test: with no Claude Code anywhere, the email sign-in fetches this platform's first, then signs in with it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L482 ]
[test: a failed turn keeps why it failed even when the reply already said it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L112 ]
[test: an agent with its own Settings section offers the way there, and only that agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L108 ]
[test: a turn that failed to sign in offers the agent's Settings section beside its hint : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L120 ]

## S9: Switch code generation on or off @v1 [published]

**Created by:** vydra

**Updated by:** vydra

Given: the Settings dialog
When: the 'Code generation' switch is turned off and the settings are saved
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.

## S10: Switch code generation on or off @v2 [proposed]

**Created by:** vydra

**Updated by:** vydra

Given: the Settings page (user-info-settings.feature.md "open Settings")
When: the 'Code generation' switch in its Workflow section is turned off —
it saves itself the moment it changes
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.

## S11: The agent pane follows a new Claude sign-in without a restart @v1 [proposed]

- Given: the agent pane open while Claude is not set up or not signed in, or
  a Claude session in it running under one account
- When: Claude signs in with an email in Settings → Claude, signs out, or
  switches between the Claude account and an API proxy
- Then: the pane drops what it showed of the old sign-in at once — "Claude
  Code is not set up" gives way to a new session once the sign-in fetched it
- and every live Claude session restarts Claude Code under the new sign-in,
  carrying its conversation on, with a note saying so and the account chip
  showing the new account
- and a turn under way, or a Claude still starting, finishes first
- and a session whose Claude cannot start under the new sign-in keeps
  running as it was, with a note saying to start a new session

[test: a sign-in, a sign-out and another mode, proxy or token each tell the app Claude's sign-in changed : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L746 ]
[test: a sign-in change restarts an idle Claude session under it, carrying the conversation on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L891 ]
[test: a turn under way finishes before the session picks up a new sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L929 ]
[test: a new sign-in that cannot start Claude leaves the session as it was, saying so : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L945 ]
[test: a provider's sign-in change is a change for its id alone; an unknown id is nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L89 ]
[test: an agent's sign-in change has every open page re-read the agents : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L100 ]
[test: a provider's sign-in change refreshes its live sessions, and a failed refresh is a note : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L477 ]

## S12: Choose the model Claude starts on @v1 [proposed]

- Given: the Settings page's Claude section
- Then: it offers "Default model", a field that takes an alias Claude Code
  knows (`opus`, `sonnet`, `haiku`, `opusplan`, suggested as you type)
  or a full model id, such as `claude-sonnet-4-5[1m]` or a Bedrock id
- and while Claude signs in with a Claude account, the field is a dropdown
  of the models that account offers — what Claude Code's own model menu
  lists for it, such as "Opus 5.5 — opus" — asked of a Claude Code started
  for nothing else, which sends nothing to the API; it still shows the model
  a new Claude starts on, and the list is asked for again after a sign-in or
  a sign-out
- and while Claude goes through an API proxy, the dropdown offers the models
  the proxy lists instead — its `GET /v1/models` answer, read with the token
  the way Claude Code sends it, page by page, and asked for again after a
  proxy check; a proxy that lists none leaves Claude Code's aliases
- and where the "Claude models" list below it in Settings → Claude names
  models, the dropdown is narrowed to those alone, in the list's order, each
  under the name the sign-in gives it, the first chosen until another is,
  and a new Claude starts on the one it shows; a default chosen before the
  list left it out is not used — the dropdown shows the list's first, and
  the pane, the import and the app's other Claude calls start on it; an empty field with no list leaves Claude
  Code's own default, whatever its settings or account pick
- and that list is one row per model: the row Claude Code names for it
  (`default`, `opus`, `sonnet`, `haiku`, Fable), never the explicit ids of
  the same models an organization is also offered (`claude-opus-4-8`,
  `claude-opus-4-7`, …), which would only lengthen the menu
- and the proxy's model names in Claude Code's own settings or the
  environment (ANTHROPIC_MODEL, ANTHROPIC_DEFAULT_*_MODEL, the gateway's
  model discovery) are set aside for the account like its proxy and keys,
  so the list carries the account's models under their own names, not
  "Set by ANTHROPIC_DEFAULT_MODEL"
- and the value saves itself when the field loses focus or Enter is pressed,
  to the plugin's section of .specdriven/settings.json, and a value with
  spaces or that is no model id is refused with a message
- and every Claude session the coding agent pane starts afterwards runs on
  that model, as do the app's one-shot Claude calls that ask for no model of
  their own
- and a session carried on under a new sign-in keeps the model it ran on,
  even when the default changed meanwhile
- and a model picked in the coding agent pane's bar becomes this default, and
  the field shows it

[test: the default model is an alias or a model id, trimmed; anything else is Claude Code's own : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L169 ]
[test: signed in with a Claude account, the default model is a dropdown of every model the account offers : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L92 ]
[test: the proxy's GET /v1/models answer becomes the menu, page by page, with the token the way Claude Code sends it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L145 ]
[test: GET /api/claude/models lists the signed-in account's models, asked of Claude Code once per sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L676 ]
[test: Claude Code's model rows become account models, dropping what is no model id and repeats : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L185 ]
[test: the launch steers Claude Code through --settings, which beats its own files : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L209 ]
[test: a new session starts on the default model Settings names while the list holds it, else the first listed, else Claude Code's own : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1105 ]
[test: a default in Settings the data repo's list does not hold gives way to the list's first : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L94 ]
[test: a session carried on under a new sign-in keeps the model it ran on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1126 ]
[test: a model asked for wins over the default Settings names : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L67 ]

## S13: Pick Claude's model and effort in the pane @v1 [proposed]

- Given: the coding agent pane's bar with Claude chosen
- Then: the model menu offers what Settings → Claude's "Default model"
  would: every model the signed-in Claude account offers, else the models
  Settings → Claude lists, else Opus, Sonnet and Haiku
- and the effort menu offers the levels Claude Code says the model takes,
  every level when it does not say, none for a model that takes none
- and a new session starts on the model and effort the bar shows, else
  Settings → Claude's defaults; a pick in the bar, before or during a
  session, becomes those defaults
- and a running session switches model and effort without restarting Claude
  Code, and a session carried on under a new sign-in keeps both
- and Settings → Claude has "Default effort" beside "Default model": Claude
  Code's default, or a level

[test: a session starts on the model and effort the pane picked, over the defaults Settings names : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1145 ]
[test: the pane switches a running Claude's model and effort, and the pick becomes Settings → Claude's default : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1166 ]
[test: the pane's choices are the models the sign-in offers, else Claude Code's aliases, narrowed to the data repo's list : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1183 ]
[test: a session carried on under a new sign-in keeps the model and effort the pane switched to : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1253 ]
[test: an account model keeps the effort levels Claude Code lists for it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L217 ]

## S14: Claude reports its context usage to the pane @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a Claude session in the coding agent pane
- When: the session is ready, a turn ends, or Claude Code compacts the
  conversation
- Then: the plugin asks Claude Code for its /context count — the cheap one,
  from the last response and local estimates — and hands it to the pane's
  session popover (agent-pane.feature.md S11): the window's size, the
  tokens in it, where it compacts, and every category with what it is made
  of (MCP tools by server, memory files, skills, system tools)
- and the count is asked for after the turn has freed the message box, never
  before, and a count overtaken by a newer one is dropped
- and the popover's Refresh asks Claude Code to count every category with
  the API, shown as measured
- and a Claude Code too old to answer is asked once; from then on each
  turn's own figures stand in: the tokens the last request carried, of the
  model's window
- and the plugin runs on apps without the popover too, where it counts
  nothing

[test: Claude Code's context count becomes the pane's slices, with what each is made of : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1368 ]
[test: Claude counts its context once ready and after every turn, and Refresh measures it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1403 ]
[test: a Claude Code that cannot count its context is asked once, then each turn's totals stand in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1442 ]
