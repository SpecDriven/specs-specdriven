# Integration with Codex agent

**Created by:** David Vydra

**Updated by:** David Vydra

OpenAI's Codex CLI in the coding agent pane, through the `agent-codex` plugin. Like Claude, Codex signs in with either an account (ChatGPT) or the API, chosen in Settings → Codex, and offers the models that sign-in lists. Written against Codex CLI 0.169.

## S1: A session follows the sign-in Settings chose @v1 [proposed]

- Given: Settings → Codex is on "API", with an address or a key of the app's own
- When: the app starts a Codex — a session in the coding agent pane, or a
  one-shot turn such as a task estimate
- Then: Codex is started with the API as a model provider of its own, for
  that process alone: its address, and its key in an environment variable,
  never on the command line
- and Codex's own files (`~/.codex/config.toml`, `~/.codex/auth.json`) are
  never edited
- and with nothing of the app's own stored, Codex is left to its own config
- Given: Settings → Codex is on "ChatGPT account"
- Then: Codex is started on its own OpenAI provider with the ChatGPT login,
  and an API address or key in Codex's own config or the environment is set
  aside
- and while no ChatGPT account is signed in the session is refused, saying
  to sign in or choose the API in Settings → Codex, and no Codex is started
- and the pane header's chip says which the session spends: the API and its
  host, or the ChatGPT account

Codex keeps one sign-in, and signs it out when a launch requires the other
kind (an API key on file is deleted when the ChatGPT login is required, and
the other way round). So the ChatGPT login is only ever required of a Codex
that `codex login status` found signed in to one; and where one is on file,
an API launch says so, so that a `forced_login_method = "api"` in Codex's
own config does not sign the account out.

[test: the API sign-in reaches Codex as a model provider of its own, its key in the environment and on no command line : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L243 ]
[test: the API sign-in keeps a ChatGPT login on file from being signed out, and leaves Codex alone with nothing of its own : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L268 ]
[test: the account sign-in sets the API aside, and is never required of a Codex with no ChatGPT login : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L285 ]
[test: describeSignIn says the API and its host, or the ChatGPT account and whether it is signed in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L305 ]
[test: a session under the API sign-in starts Codex with the API as its provider and the key in its environment : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L491 ]
[test: a session under the account sign-in is refused while no ChatGPT account is signed in, and starts once one is : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L509 ]
[test: a codex session spawns `codex app-server` on the work dir and is ready once its thread starts : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L590 ]

## S2: API settings are filled in from Codex's own config @v1 [proposed]

- Given: Codex's own config (`~/.codex/config.toml`) names a model provider
  with an address, or an `openai_base_url`, and an API key is in
  `~/.codex/auth.json` or in the environment variable that provider names
- When: the Settings page's Codex section is on "API"
- Then: it shows the detected address and where it came from, with "Use
  detected" to make that address and key the app's own
- and the key is kept in the app's own owner-only store (OS-encrypted in
  the desktop app), never shown on the page, and never sent back to it
- and a pasted key a request cannot carry — with a space, a line break or a
  letter outside ASCII in it, such as a pasted "Bearer sk-…" — is refused
  with why, and no answer to the page quotes a key, however a request with
  it failed
- and with a key and no address anywhere, the API is OpenAI's own
  (`https://api.openai.com/v1`)
- and "Check connection" asks the API for its model list (`GET
  <address>/models`) and reports OK, a refused key, or an unreachable API

[test: readCodexConfig reads the provider Codex's config names, its address and key variable, and a required login : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L186 ]
[test: detectCodexApi finds the address in Codex's config and the key in its auth.json, or in the variable the provider names : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L198 ]
[test: Check connection asks the API for its model list with the key, and never says the key back : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L397 ]
[test: GET /api/codex/auth reports the mode, the ChatGPT login and the API — never a key : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L417 ]
[test: Check connection and the model list go to the address in effect with the stored key : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L474 ]
[test: the API shows what Codex's own config carries, with Use detected, and never calls the address a proxy : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L343 ]
[test: with no address anywhere the API is OpenAI's own, called with the key : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L359 ]

## S3: Choose how Codex signs in @v1 [proposed]

- Given: the Settings page's Codex section
- When: the user picks "Sign in with": "ChatGPT account" or "API"
- Then: the choice saves itself the moment it changes, to the app's
  settings file, and applies to every Codex the app starts afterwards — the
  coding agent pane, a task estimate
- and until one is picked the section says which was chosen for it: the
  API when Codex's own config carries an address or a key, else the ChatGPT
  account
- and the section holds what Claude's does: the sign-in, the account or the
  API, the default model, and the data repo's list of Codex models

Note: this section is not offered in the demo or the hosted app, which run no Codex of their own

[test: the stored choice is normalized; an address must be http(s), and loses its trailing slash : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L157 ]
[test: with no choice stored, an API in Codex's own config means the API, and nothing there means the ChatGPT account : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L219 ]
[test: PUT /api/codex/settings stores the choice, the address, the model and the effort, and null forgets them : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L446 ]
[test: the sign-in routes refuse in the demo and the hosted app : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L465 ]

## S4: Codex lists the models its sign-in offers @v1 [proposed]

- Given: Settings → Codex is on "API", with an address in effect
- When: a model menu for Codex opens — the coding agent pane's model
  picker, or Settings → Codex's "Default model"
- Then: it offers the models the API lists: `GET <address>/models`, with
  the key as a bearer token
- and on "ChatGPT account" it offers the models Codex itself lists for the
  signed-in account
- and the data repo's list of Codex models (`.specdriven/codex-models.json`)
  narrows the menu to the names it lists, in its order, when it lists any
- and the answer is kept a while per sign-in, and asked for again when the
  sign-in changes or "Check connection" finds the API answering
- and a new session starts on the default model — the one Settings names,
  else the data repo's first, else Codex's own — and a pick in the pane
  becomes that default, and switches a running session from its next turn
- and with no list from the sign-in and none in the data repo, "Default
  model" is a field for a model id

[test: the API's models are read from GET <address>/models with the key as a bearer token : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L59 ]
[test: an API that refuses the key, or fails, says why without the key : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L73 ]
[test: a ChatGPT account's models are Codex's own list for it, the hidden ones left out : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L85 ]
[test: the sign-in's models are asked for once per sign-in, kept a while, and asked again when it changes : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L165 ]
[test: the data repo's list for Codex narrows the menu, in its own order, keeping a name the sign-in does not offer : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L189 ]
[test: the pane's model menu is what the sign-in offers, narrowed by the data repo's list, with the default a session starts on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L214 ]
[test: a Codex with no Settings of its own offers no model menu and no account chip : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L260 ]
[test: Default model is a dropdown of the models the API lists, with Codex's own default until one is chosen : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L305 ]
[test: Default model is narrowed to the Codex models the data repo lists, the first until another is chosen : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L317 ]
[test: with no model list from the sign-in and none in the data repo, Default model is a field for a model id : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L330 ]
[test: a session starts on the model Settings → Codex names, and reports the sign-in it started under : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L635 ]

## S5: Sign in to Codex with a ChatGPT account @v1 [proposed]

- Given: Settings → Codex is on "ChatGPT account"
- Then: the section says plainly who is signed in, or that nobody is
- When: the user clicks "Sign in"
- Then: the Codex CLI opens the browser on ChatGPT's sign-in page, the
  section says to finish there and offers Cancel, and it updates by itself
  once Codex has the account; nothing is pasted into the app
- and "Check sign-in" asks the CLI afresh who is signed in and has a Codex
  started under the account list its models
- and "Sign out" signs the ChatGPT account out, and is refused while none
  is signed in — Codex keeps one sign-in, and a sign-out would take an API
  key instead
- and while Codex's own API key is on file the section says that signing in
  replaces it, and to copy it to the API first ("Use detected")
- and while Codex's own config requires API keys the section says that a
  Codex started outside the app signs the account out again

[test: loginStatus asks `codex login status` as the account sign-in starts Codex: a ChatGPT login, or none : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L323 ]
[test: the sign-in check's finding is one sentence: the account, and whether Codex lists its models : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L413 ]

[test: Sign in runs `codex login` under the account sign-in, and is done once the CLI reports a ChatGPT login : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L351 ]
[test: a cancelled sign-in stops the CLI, and Sign out is refused unless a ChatGPT account is signed in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/signIn.test.ts#L377 ]
[test: signed out, the ChatGPT account says so plainly and offers Sign in and a check : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L375 ]
[test: signed in, the ChatGPT account names the account and offers Check sign-in and Sign out : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L382 ]
[test: the ChatGPT account warns that signing in replaces Codex's own API key, and of a config that signs the account out : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L389 ]
[test: while the browser sign-in is under way the section says to finish it there and offers only Cancel; with no Codex there is nothing to sign in with : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L403 ]

## S6: Codex offers the reasoning efforts its model takes @v1 [proposed]

- Given: the coding agent pane's model picker is on Codex
- When: a model is picked
- Then: the Effort row offers the levels that model takes, as Codex says:
  its own list for the signed-in ChatGPT account, and for the API's models
  the catalog shipped with the Codex CLI (`codex debug models --bundled`,
  read offline and under no sign-in)
- and a model Codex knows no efforts for — one its catalog lacks, such as
  `gpt-4.1` — has no Effort row, and Codex levels the pane has no name for
  (`minimal`, `ultra`) are left out
- and a new session runs at the default effort — the one Settings → Codex's
  "Default effort" names, else the model's own — and a pick in the pane
  becomes that default, and switches a running session from its next turn
- and the effort goes to Codex with each turn, lowered to the highest level
  below it that the turn's model takes, else the model's lowest
- and back on "Default" after another level, the next turn asks for the
  model's own default effort, since Codex keeps the last effort it was given

Note: Codex keeps a session's effort for the turns after, even across a switch to a model that takes none, and offers no way to clear it

[test: a ChatGPT account's models carry the efforts Codex lists for each, as the pane's levels : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L101 ]
[test: an API's models take the efforts Codex's own catalog gives them, and none for a model it does not know : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L126 ]
[test: an effort a model does not take is lowered to the highest it takes below it, else its lowest : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L154 ]
[test: the pane's model menu is what the sign-in offers, narrowed by the data repo's list, with the default a session starts on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L214 ]
[test: Default effort is a select of the pane's levels, the model's own default when none is set : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/models.test.tsx#L293 ]
[test: a session runs at the effort Settings → Codex names, lowered to one its model takes, and the pane's pick goes with the next turn : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L657 ]
[test: with no effort picked, turns ask Codex for none : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L687 ]
