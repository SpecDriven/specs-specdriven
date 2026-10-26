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

[test: detectClaudeProxy reads the environment, then each settings file, the later winning : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L111 ]
[test: checkProxy asks for the model list and reads the answer : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L232 ]
[test: GET /api/claude/auth reports the mode, the login and the proxy — never a token : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L381 ]
[test: POST /api/claude/proxy/check reports on the proxy with the token in use : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L450 ]
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

[test: the stored choice is normalized; a url must be http(s) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L140 ]
[test: with nothing stored the mode is inferred — a proxy when Claude Code names one, else the account : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L155 ]
[test: the launch steers Claude Code through --settings, which beats its own files : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L182 ]
[test: the demo and the hosted app have no Claude sign-in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L493 ]
[test: the sign-in Settings chose reaches Claude as a --settings override (src/auth.ts) : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L963 ]

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
- and "Sign out" signs the CLI out; "Switch account…" starts another sign-in
- and with no Claude CLI on PATH the section says so instead

[test: parseLoginStatus reads what claude auth status --json says : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L265 ]
[test: loginStatus asks the CLI without any key in its environment, and says missing without one : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L275 ]
[test: startLogin drives claude auth login, hands the page the url, and is done once the CLI says so : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L330 ]
[test: a sign-in can be cancelled, and a bad email or a missing CLI is refused : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L350 ]
[test: the sign-in routes drive the CLI and poll it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L475 ]

## S8: Switch code generation on or off @v1 [published]

Given: the Settings dialog
When: the 'Code generation' switch is turned off and the settings are saved
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.

## S9: Switch code generation on or off @v2 [proposed]

Given: the Settings page (user-info-settings.feature.md "open Settings")
When: the 'Code generation' switch in its Workflow section is turned off —
it saves itself the moment it changes
Then: no selection checkboxes are offered on sidebar entries or scenario
cards, 'Start session' is gone, and any selection is cleared

The server refuses a code gen session while the switch is off, so nothing can
reach the Claude CLI even outside the UI.

The switch is ON by default, and the setting is saved to
.specdriven/settings.json.
