# Agent plugins

**Updated by:** jmeowmeow

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
[test: a plugin's provider drives a session from start to end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L88 ]
[test: the pane's permission card answers the provider, and a withdrawn or ended question is denied : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L120 ]
[test: Stop and the mode go to the provider, and are refused where it declares neither : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L165 ]
[test: a turn the provider refuses is noted and the composer freed; a provider that dies ends the session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L303 ]
[test: the provider's facts reach the pane through ready and update, and quiet keeps them off the log : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L320 ]
[test: switching a plugin off ends the sessions its provider was driving : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L414 ]
[test: a Claude plugin switched off takes its routes and its provider away : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L868 ]
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

[test: no plugin, the wrong plugin, or a missing CLI is a clear refusal, and the default is the first by rank : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L349 ]
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
[test: show only where their when allows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsRegistry.test.tsx#L48 ]
[test: a patch merges into the plugin's section, null removes a key, and an emptied section goes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L59 ]
[test: the shape is checked, never the keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L73 ]
[test: the service a host half gets reads, writes and adopts its section of the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L84 ]
[test: PUT /api/state takes a patch per plugin id, GET reports the sections, and the hosted app refuses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSettings.test.ts#L114 ]
[test: the plugin adopts the old top-level keys once, and PUT /api/claude/settings changes its own section : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/auth.test.ts#L811 ]
[test: a store round-trips through an owner-only file under the app's home, and is cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L39 ]
[test: a corrupt or empty file reads as no secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L58 ]
[test: the Claude token keeps its old file, through the store the Claude plugin owns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L70 ]
[test: the backend factory swaps every store, the ones handed out before included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/secrets.test.ts#L80 ]

## S7: Each agent offers the models the data repo lists @v1 [proposed]

- Given: the coding agent plugins installed — Claude, Codex, Pi
- When: the user opens Settings
- Then: each agent's own section — Settings → Claude — has a "<agent>
  models" list, and an agent whose plugin adds no section (Codex) gets a
  section the app titles after it holding its list; there is no separate
  section for the lists; each is one model name per line, as that agent's
  CLI names them — `opus` or `claude-sonnet-4-5[1m]` for Claude,
  `gpt-5-codex` for Codex — saved when the field is left
- and each agent's list is kept in a file of its own in the data repo's
  `.specdriven/` — `claude-models.json`, `codex-models.json`, a JSON list of
  names — so the team shares them; a name with spaces is refused with a
  message, a teammate's list for an agent this app lacks is kept, and a data
  repo that still has the old shared `agent-models.json` has it split into
  those files
- and a listed model should be one the agent's API reports: one it does not
  report is still kept and offered, and the app's log gets a warning naming
  the agent, the model and its file, once until the list or the report
  changes; an agent whose API cannot say (signed out, an API key) is not
  checked
- and the list narrows what the agent offers: each agent plugin lists every
  model its sign-in offers — the signed-in account's, a proxy's or the API's
  `GET /v1/models`, else its CLI's aliases — and where the data repo lists
  names for the agent, every model menu offers those alone, in the list's
  order, each under the name the agent gives it; a listed name the agent
  does not offer is kept as typed; an empty list narrows nothing
- and every model field for an agent is a dropdown of what is left — the
  coding agent pane's model, the "Import from Jira/Wiki" dialog's Model,
  Settings → Claude's default model — the agent's own default chosen until
  another is, else the first; a model a running session or the import
  dialog chose before the menu lost it stays chosen, but a default model
  the list does not hold is not used: the list's first is; with nothing
  offered and nothing listed the field says "No models listed" and offers
  nothing

[test: each agent keeps its own list in the data repo's .specdriven/<agent>-models.json : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L58 ]
[test: a name no agent would take, a bad agent id or too many names is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L67 ]
[test: a teammate's list for an agent this app lacks survives a save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L80 ]
[test: a broken file reads as no list and is not overwritten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L100 ]
[test: the old shared agent-models.json is split into a file per agent and removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L111 ]
[test: an old shared file that is not a JSON object is left to fix by hand : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L125 ]
[test: reading keeps names only, valid, once each : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L97 ]
[test: a listed model the agent's API does not report is logged as a warning, once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L147 ]
[test: the lists come with the state and Settings writes one agent's at a time : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L119 ]
[test: saving a list checks it against the models the agent's API reports : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModels.test.ts#L204 ]
[test: the host reads Claude's list from the data repo's claude-models.json, in order : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/models.test.ts#L12 ]
[test: the models the API reports are every value and id the account offers, or none to check against : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/models.test.ts#L32 ]
[test: a list per agent, a model per line, saved on blur to that agent's key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelsSettings.test.tsx#L105 ]
[test: a refused list says why and is kept to fix : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelsSettings.test.tsx#L140 ]
[test: an agent whose plugin adds no Settings section gets one of its own for its list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelsSettings.test.tsx#L158 ]
[test: the lists come from the state payload; anything else is none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelsSettings.test.tsx#L174 ]
[test: a Model field of the models the agent offers, its default chosen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiDialog.test.tsx#L102 ]
[test: the names Settings → Claude lists narrow the offered models, in the list's order, the first chosen until another is : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L111 ]
[test: the pane's choices are the models the sign-in offers, else Claude Code's aliases, narrowed to the data repo's list : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1167 ]
[test: the default model is a dropdown of only the models the app lists for Claude, the first until another is chosen : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L44 ]
[test: an app that keeps no model lists leaves the default model a free field : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L61 ]
[test: Claude's list comes from the app's state payload, keyed by agent : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L67 ]
[test: the host reads Claude's list from the data repo's agent-models.json, in order : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/defaultModel.test.tsx#L74 ]

## S8: The core hands a provider the MCP servers a session should have @v1 [proposed]

- Given: a plugin's coding agent (S1)
- When: the core starts a session on it
- Then: the start options carry `mcpServers` — each a name, a command, its
  arguments and the environment it needs: a server over stdio — when the app
  has any to offer: the Confluence server while a wiki token is saved
  (external-links/confluence-wiki.feature.md); none, and the field absent,
  otherwise
- and the provider hands them to its agent the way that agent takes MCP
  servers — Claude as the Agent SDK's stdio server configs under the
  server's name, Codex as `-c mcp_servers.<name>=…` overrides on
  `codex app-server`, the user's own config untouched — and a provider whose
  agent takes none ignores the field
- and no token travels in the options: the environment names the file the
  server reads

[test: a saved wiki token hands the provider the Confluence MCP server; none, nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L525 ]
[test: the MCP servers the app asks for reach the SDK as stdio servers, by name : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1283 ]
[test: mcpServerConfigs keys the SDK's stdio configs by server name : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1310 ]
[test: the MCP servers the app asks for reach `codex app-server` as -c overrides, never the user's config : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L986 ]
[test: strings are quoted with their quotes, backslashes and control characters escaped : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L1008 ]
[test: a server is one inline table under its name, and an odd name is made a bare key : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L1014 ]

## S9: A coding session should check that source is up to date @v1 [proposed]

**Created by:** jmeowmeow

**Updated by:** jmeowmeow

- Given: a coding session is about to start in a folder that is the code
  repository under development (not the SpecDriven data repo, which the app
  already pulls and pushes on its own)
- When: the user starts the session and the branch there is not already clean
  and level with its remote of record
- Then: before the agent runs, the pane reads that branch's state — its name,
  how far it is ahead of and behind its upstream after a bounded live fetch,
  and whether the working tree is dirty — and shows it so new work branches
  from the right place
- and online, with a clean tree, it offers to bring the branch up to date
  before the session — a merge (`git pull --no-rebase`) or a rebase (`git pull
  --rebase`) — or to start as it is; the chosen refresh runs first and the
  session starts only once it finishes
- and offline, with no upstream, or with a dirty tree, it shows the local state
  (the last the repo saw of its origin, if any) and offers no refresh, only to
  start as it is or cancel, saying why a refresh is off
- and a refresh that leaves conflicts, or fails, is reported and the session is
  not started until it is resolved; a branch already clean and up to date, or a
  folder that is not a code repository, starts with no dialog

[test: a clean branch level with its upstream is online, up to date, not dirty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L78 ]
[test: a branch whose origin advanced is reported behind, after a live fetch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L95 ]
[test: a local commit the upstream lacks is reported ahead : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L105 ]
[test: an uncommitted change marks the tree dirty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L115 ]
[test: a branch with no upstream is local only, not an error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L122 ]
[test: a detached HEAD is reported as such, with no branch to refresh : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L131 ]
[test: a folder that is not a git repository is reported, not thrown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L141 ]
[test: no git on PATH degrades to a reason, not a crash : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L150 ]
[test: an unreachable remote reports offline but still returns the last-known state : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L160 ]
[test: refreshSource with merge advances a behind branch to its upstream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L179 ]
[test: refreshSource with rebase replays local work on top of the upstream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L194 ]
[test: refreshSource refuses a dirty working tree rather than half-applying : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L214 ]
[test: refreshSource reports a conflict instead of leaving a silent half-merge : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L227 ]
[test: GET /api/agent/source returns the branch's status for an absolute path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L241 ]
[test: GET /api/agent/source rejects a relative path and a missing folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L249 ]
[test: POST /api/agent/source/refresh runs the merge and reports success : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L254 ]
[test: POST /api/agent/source/refresh validates the path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSource.test.ts#L264 ]
[test: names how far behind and ahead the branch is of its upstream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L36 ]
[test: flags an offline (last-known) comparison and uncommitted changes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L41 ]
[test: says local-only and detached in words, with no upstream talk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L45 ]
[test: is on only with a remote, a clean tree, online, and a branch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L52 ]
[test: explains why a refresh is off when there is a remote but it cannot run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L62 ]
[test: a behind branch offers Rebase, Update, Start anyway, and Cancel : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L71 ]
[test: offline and local-only drop the refresh buttons and keep confirm-or-cancel : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L81 ]
[test: the why-off hint shows when a refresh is blocked, not when it is available : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L88 ]
[test: busy disables every action so a refresh cannot be double-run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sourceCheckDialog.test.tsx#L97 ]
