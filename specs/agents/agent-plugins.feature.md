# Agent plugins

The coding agent pane drives whatever coding agent a plugin supplies. The core defines the agent interface and keeps a registry of providers; Claude, Codex and Pi are providers like any a future plugin brings, and the app names none of them.

## S1: A plugin supplies a coding agent @v1 [proposed]

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
[test: a plugin's provider drives a session from start to end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L87 ]
[test: the pane's permission card answers the provider, and a withdrawn or ended question is denied : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L119 ]
[test: Stop and the mode go to the provider, and are refused where it declares neither : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L164 ]
[test: a turn the provider refuses is noted and the composer freed; a provider that dies ends the session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L203 ]
[test: the provider's facts reach the pane through ready and update, and quiet keeps them off the log : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L220 ]
[test: switching a plugin off ends the sessions its provider was driving : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L314 ]
[test: a Claude plugin switched off takes its routes and its provider away : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L548 ]
[test: an installed plugin whose manifest says autoEnable switches itself on in the open folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L238 ]
[test: an agent plugin that arrives while the window is open offers the pane : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L60 ]

## S2: A session record from before agent plugins still opens @v1 [proposed]

- Given: a session record written before agents were plugins, whose agent
  says "pane", "pi-pane" or "codex-pane"
- When: the app reads it
- Then: the record's agent is read as the provider id — Claude, Pi or Codex;
  the file is not rewritten
- and a session whose provider has no plugin any more still lists and opens,
  with its name, no permission modes, no Stop, and a new session on it refused

[test: a session record's agent from before agent plugins reads as the provider id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentLegacy.test.ts#L10 ]
[test: a record whose plugin is gone renders through a fallback with no modes and no Stop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentLegacy.test.ts#L23 ]

## S3: Agent plugins are installed for the commands found on this machine @v1 [proposed]

- Given: a first start of the app, not hosted and not the demo, with the
  marketplace index on hand
- When: the index names an agent plugin whose command is on PATH and the
  plugin is not installed
- Then: the app installs it the way Settings → Plugins would, once; a plugin
  the user removed is not installed again, and a start that cannot reach
  the marketplace installs nothing and says so in the log

[test: the index's agent field is read when it names an id and a command : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L51 ]
[test: installs the agent plugins whose commands are on PATH, once, and remembers them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L63 ]
[test: a marketplace that cannot be reached, or an install that fails, installs nothing and is reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L97 ]
[test: the offers come from the cached indexes on disk, for agents no plugin registers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L119 ]
[test: a start installs the agent plugin for a command it finds, mounts it, and records it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L137 ]

## S4: The core names no agent @v1 [proposed]

- Given: the app with no agent plugin installed
- When: the user asks for a session, a code gen target or an estimate
- Then: the app refuses with "No coding agent is installed — install one
  from Settings → Plugins", and nothing in the core mentions Claude, Codex or
  Pi by name
- and with plugins installed, a session asked for without an agent runs on
  the first one whose command is on PATH, by rank; the "Run in" choice of the
  session dialog lists the agents the plugins supply, by rank

[test: no plugin, the wrong plugin, or a missing CLI is a clear refusal, and the default is the first by rank : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L249 ]
[test: a session's agent is read off the descriptors, or a fallback when its plugin is gone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L26 ]
[test: only agents whose CLI is there are usable, in rank order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L42 ]
[test: the state's agents list is what the pane reads, and nothing stands in for it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L49 ]
[test: an unchosen dialog opens on the first usable agent by rank : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionDialog.test.tsx#L77 ]
[test: the session target is only ever desktop or a registered agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L162 ]

## S5: The pane offers to install an agent plugin @v1 [proposed]

- Given: the coding agent pane with no agent that can start
- When: the pane opens on its empty state
- Then: with no agent plugin and nothing to offer, it says agents come as
  plugins and opens Settings → Plugins; with a plugin installed whose
  command is missing, it says so in the plugin's words; for a marketplace
  plugin whose command is on PATH, it offers to install the plugin, and for
  one whose command is not, it names the command to install first
- and nothing installs from the pane: the Plugins page has the consent step

[test: with no agent plugin and nothing to install, it says agents come as plugins : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L91 ]
[test: an installed plugin whose CLI is missing says so in the provider's words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L100 ]
[test: a marketplace offer is installable when its CLI is there, and names the CLI first when not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentDescriptors.test.tsx#L142 ]

## S6: The Settings page shows a plugin's section @v1 [proposed]

- Given: a plugin whose browser half registers a Settings section — a title,
  an order among the core's sections, and what it renders
- When: the user opens Settings
- Then: the section sits where its order puts it, with the page's own
  navigation entry, and what it changes goes through the plugin's own
  settings, kept under the plugin's name in the settings file
- and a secret the plugin keeps — a token — lives apart from the settings
  file, in a store of the plugin's own on this device

[test: sit between the core sections their order puts them between, in order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsRegistry.test.tsx#L27 ]
[test: show only where their when allows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsRegistry.test.tsx#L45 ]
[test: a patch merges into the plugin's section, null removes a key, and an emptied section goes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L59 ]
[test: the shape is checked, never the keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L73 ]
[test: the service a host half gets reads, writes and adopts its section of the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L84 ]
[test: PUT /api/state takes a patch per plugin id, GET reports the sections, and the hosted app refuses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L114 ]
[test: the plugin adopts the old top-level keys once, and PUT /api/claude/settings changes its own section : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L510 ]
[test: a store round-trips through an owner-only file under the app's home, and is cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L39 ]
[test: a corrupt or empty file reads as no secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L58 ]
[test: the Claude token keeps its old file, through the store the Claude plugin owns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L70 ]
[test: the backend factory swaps every store, the ones handed out before included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L80 ]
