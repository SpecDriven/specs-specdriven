# Agent plugins

The coding agent pane drives whatever coding agent a plugin supplies. The core defines the agent interface and keeps a registry of providers; Claude, Codex and Pi are providers like any a future plugin brings, and the app names none of them.

## A plugin supplies a coding agent @v1 [proposed]

- Given: a plugin whose host half registers an agent provider — a name, a
  rank, the command it needs on PATH, the permission modes it offers and
  the one it starts in, and what it can do (Stop, a mode change mid-session,
  an account chip, "Allow always")
- When: the user starts a session on it from the coding agent pane
- Then: the pane lists it with the other agents in rank order and starts the
  session through it; what the agent says, asks and answers flows through
  the same event log, permission cards and session record as any other
- and a provider whose command is missing is refused with its own words,
  and a session on a provider whose plugin is switched off ends with a note
  saying so

[test: lists providers by rank and describes them for the client : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L11 ]
[test: the default is the lowest-ranked provider whose CLI is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L35 ]
[test: oneShot picks the preferred provider when it answers, else the first by rank that does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L50 ]
[test: registering an id again replaces the provider, and a disposer removes only its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRegistry.test.ts#L65 ]
[test: a plugin's provider drives a session from start to end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L96 ]
[test: the pane's permission card answers the provider, and a withdrawn or ended question is denied : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L128 ]
[test: Stop and the mode go to the provider, and are refused where it declares neither : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L173 ]
[test: a turn the provider refuses is noted and the composer freed; a provider that dies ends the session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L212 ]
[test: the provider's facts reach the pane through ready and update, and quiet keeps them off the log : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L229 ]
[test: switching a plugin off ends the sessions its provider was driving : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L287 ]

## A session record from before agent plugins still opens @v1 [proposed]

- Given: a session record written before agents were plugins, whose agent
  says "pane", "pi-pane" or "codex-pane", or a settings file whose "Run in"
  choice says "pane"
- When: the app reads it
- Then: the record's agent is read as the provider id — Claude, Pi or Codex
  — and the choice as Claude; the files are not rewritten
- and a session whose provider has no plugin any more still lists and opens,
  with its name, no permission modes, no Stop, and a new session on it refused

[test: a session record's agent from before agent plugins reads as the provider id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentLegacy.test.ts#L10 ]
[test: a record whose plugin is gone renders through a fallback with no modes and no Stop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentLegacy.test.ts#L24 ]

## Agent plugins are installed for the commands found on this machine @v1 [proposed]

- Given: a first start of the app, not hosted and not the demo, with the
  marketplace index on hand
- When: the index names an agent plugin whose command is on PATH and the
  plugin is not installed
- Then: the app installs it the way Settings → Plugins would, once; a plugin
  the user removed is not installed again, and a start that cannot reach
  the marketplace installs nothing and says so in the log

## The core names no agent @v1 [proposed]

- Given: the app with no agent plugin installed
- When: the user asks for a session, a code gen target or an estimate
- Then: the app refuses with "No coding agent is installed — install one
  from Settings → Plugins", and nothing in the core mentions Claude, Codex or
  Pi by name
- and with plugins installed, a session asked for without an agent runs on
  the first one whose command is on PATH, by rank; the "Run in" choice of the
  session dialog lists the agents the plugins supply, by rank

[test: no plugin, the wrong plugin, or a missing CLI is a clear refusal, and the default is the first by rank : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L258 ]
[test: a session's agent is read off the descriptors, or a fallback when its plugin is gone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L15 ]
[test: only agents whose CLI is there are usable, in rank order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L31 ]
[test: the state's agents list is what the pane reads, and the old per-agent flags stand in for a server without one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L38 ]
[test: an unchosen dialog opens on the first usable agent by rank : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionDialog.test.tsx#L79 ]
[test: the session target is only ever desktop or a registered agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L162 ]

## The pane offers to install an agent plugin @v1 [proposed]

- Given: the coding agent pane with no agent that can start
- When: the pane opens on its empty state
- Then: with no agent plugin and nothing to offer, it says agents come as
  plugins and opens Settings → Plugins; with a plugin installed whose
  command is missing, it says so in the plugin's words; for a marketplace
  plugin whose command is on PATH, it offers to install the plugin, and for
  one whose command is not, it names the command to install first
- and nothing installs from the pane: the Plugins page has the consent step

[test: with no agent plugin and nothing to install, it says agents come as plugins : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L60 ]
[test: an installed plugin whose CLI is missing says so in the provider's words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L69 ]
[test: a marketplace offer is installable when its CLI is there, and names the CLI first when not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L77 ]

## The Settings page shows a plugin's section @v1 [proposed]

- Given: a plugin whose browser half registers a Settings section — a title,
  an order among the core's sections, and what it renders
- When: the user opens Settings
- Then: the section sits where its order puts it, with the page's own
  navigation entry, and what it changes goes through the plugin's own
  settings, kept under the plugin's name in the settings file
