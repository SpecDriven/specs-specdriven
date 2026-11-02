# Integration with Claude agent

## S1: API proxy settings are filled in from Claude Code's settings @v1 [proposed]

- Given: Claude Code's own settings (`~/.claude/settings.json`, the data
  folder's `.claude/settings.json` or `.claude/settings.local.json`) or the
  environment carry `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN`
- When: the Settings page's Claude section is on "API proxy"
- Then: it shows the detected address and where it came from, with "Use
  detected" to make that address and token the app's own
- and the token is kept in the app's own owner-only store (OS-encrypted in
  the desktop app), never shown on the page, and never sent back to it
- and Claude Code's own files are never edited
- and "Check connection" asks the proxy for its model list and reports OK,
  a refused token, or an unreachable proxy
- and the "Proxy timeout" of agent-pane.feature.md lives in this section

Claude Code applies the `env` block of its settings files over the
environment it is started with, so the app passes its choice as the
`--settings` flag, which wins over every file; empty values there clear a
proxy for the account sign-in.

[test: detectClaudeProxy reads the environment, then each settings file, the later winning : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L114 ]
[test: checkProxy asks for the model list and reads the answer : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L235 ]
[test: GET /api/claude/auth reports the mode, the login and the proxy — never a token : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L485 ]
[test: POST /api/claude/proxy/check reports on the proxy with the token in use : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L554 ]
[test: a store round-trips through an owner-only file under the app's home, and is cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L39 ]
[test: a corrupt or empty file reads as no secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L58 ]

## S2: Choose how Claude signs in @v1 [proposed]

- Given: the Settings page (user-info-settings.feature.md "open Settings")
- Then: its Claude section offers "Sign in with": "Claude account (email
  login)" or "API proxy"
- and with no choice stored the app picks "API proxy" when Claude Code's own
  settings or the environment name a base url, else "Claude account", and
  says the choice was made for it
- and the choice saves itself the moment it changes, to
  .specdriven/settings.json, and applies to every Claude the app starts
  afterwards — the coding agent pane, a task estimate, a self-fix
- and the section is not offered in the demo or the hosted app, which run
  no Claude of their own

[test: the stored choice is normalized; a url must be http(s) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L143 ]
[test: with nothing stored the mode is inferred — a proxy when Claude Code names one, else the account : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L158 ]
[test: the launch steers Claude Code through --settings, which beats its own files : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L185 ]
[test: the demo and the hosted app have no Claude sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L613 ]
[test: the sign-in Settings chose reaches Claude as a --settings override (src/auth.ts) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1070 ]

## S3: Claude cli options @v1 [proposed]

In the target repo, locate folder '.specdriven'
If it contains a file called specdriven-config.json
Check for:
{
  "claude-cli": "--settings .claude/hai-claude-config.json --dangerously-skip-permissions"
}

https://www.testmuai.com/support/docs/agent-assurance-overview/

## S4: Create a prompt from selected specs @v1 [published]

Given: a code session has been created with at least one spec in it
When: we start a code gen session with Agent (Claude)
Then: we generate a prompt and pass it to the Claude Cli to execute and exit

If error code, display the error to the user in an error box.

## S5: Create and name a code gen session @v1 [published]

- Select one or more specs
- 'Start session' (click button)
- Name the session

## S6: Create and name a code gen session @v2 [published]

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
  is under way the page says so and links the sign-in page, in case the
  browser did not open by itself; "Cancel" stops it
- and once the browser comes back the page shows the account signed in
- and when the sign-in page was opened from that link rather than by the
  CLI, it shows a code instead; pasted into the page under "Finish sign-in",
  it completes the sign-in, and a code the CLI refuses ends it with why
- and "Sign out" signs the CLI out; "Switch account…" starts another sign-in
- and with no Claude Code on this machine and none published for its
  platform, the section says to install the claude command instead

[test: parseLoginStatus reads what claude auth status --json says : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L268 ]
[test: loginStatus asks the CLI without any key in its environment, and says missing without one : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L278 ]
[test: startLogin drives claude auth login, hands the page the url, and is done once the CLI says so : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L355 ]
[test: a sign-in can be cancelled, and a bad email or a platform with no Claude Code is refused : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L383 ]
[test: the sign-in routes drive the CLI and poll it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L579 ]
[test: a sign-in page opened from the printed link finishes with the code it shows, pasted in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L401 ]
[test: POST /api/claude/login/code hands the CLI the code from the sign-in page : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L597 ]

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
[test: a download that matches its pin is kept as the Claude Code to run, executable : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/binary.test.ts#L72 ]
[test: a download that does not match its pin is refused and leaves nothing behind : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/binary.test.ts#L100 ]
[test: with no Claude Code anywhere, the email sign-in fetches this platform's first, then signs in with it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L426 ]
[test: a sign-in whose download fails ends with why : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L465 ]
[test: an agent with its own Settings section offers the way there, and only that agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L108 ]
[test: a turn that failed to sign in offers the agent's Settings section beside its hint : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L120 ]

## S9: Switch code generation on or off @v1 [published]

Given: the Settings dialog
When: the 'Code generation' switch is turned off and the settings are saved
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.

## S10: Switch code generation on or off @v2 [proposed]

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

[test: a sign-in, a sign-out and another mode, proxy or token each tell the app Claude's sign-in changed : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L600 ]
[test: a sign-in change restarts an idle Claude session under it, carrying the conversation on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L829 ]
[test: a turn under way finishes before the session picks up a new sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L867 ]
[test: a new sign-in that cannot start Claude leaves the session as it was, saying so : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L883 ]
[test: a provider's sign-in change is a change for its id alone; an unknown id is nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L89 ]
[test: an agent's sign-in change has every open page re-read the agents : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L100 ]
[test: a provider's sign-in change refreshes its live sessions, and a failed refresh is a note : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L278 ]

## S12: Choose the model Claude starts on @v1 [proposed]

- Given: the Settings page's Claude section
- Then: it offers "Default model", a field that takes an alias Claude Code
  knows (`opus`, `sonnet`, `haiku`, `opusplan`, suggested as you type)
  or a full model id, such as `claude-sonnet-4-5[1m]` or a Bedrock id
- and where Settings → AI agents lists Claude's models, the field is a
  dropdown of them alone, the first chosen until another is, and a new Claude
  starts on the one it shows; an empty field with no list leaves Claude
  Code's own default, whatever its settings or account pick
- and the value saves itself when the field loses focus or Enter is pressed,
  to the plugin's section of .specdriven/settings.json, and a value with
  spaces or that is no model id is refused with a message
- and every Claude session the coding agent pane starts afterwards runs on
  that model, as do the app's one-shot Claude calls that ask for no model of
  their own
- and a session carried on under a new sign-in keeps the model it ran on,
  even when the default changed meanwhile

[test: the default model is an alias or a model id, trimmed; anything else is Claude Code's own : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L163 ]
[test: the plugin adopts the old top-level keys once, and PUT /api/claude/settings changes its own section : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L685 ]
[test: a new session starts on the default model Settings names, else the first listed, else Claude Code's own : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1043 ]
[test: a session carried on under a new sign-in keeps the model it ran on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1059 ]
[test: a model asked for wins over the default Settings names : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L67 ]
[test: no model asked for is the default Settings names : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L73 ]
[test: no model asked for and none in Settings leaves Claude Code's own default : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L79 ]
[test: no model asked for and none in Settings is the first the data repo lists for Claude : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/oneShot.test.ts#L85 ]
