# Plugins and Marketplace

**Assigned:** David

**Created by:** vydra

**Updated by:** vydra

SpecDriven is a small core plus plugins, in the style of DeepSeek Harness:
everything a module adds — a sidebar tab, a pane, routes, shortcuts,
palette commands, a settings section, a sync hook, a document type — is
registered by a plugin through a kernel, and every registration is
reversible, so switching a module off unwinds it without a restart. The
design is on the roadmap (roadmap/plugin-architecture.md); this feature
says what a user sees.

A plugin is a package with two halves: a host half that runs inside the
server (or the desktop app's main process) and a browser half that the
server serves to the app. Plugins are trusted code running in process,
which is what lets them share the app's React, design tokens, command
palette and shortcuts. The first plugins are the app's own modules —
Roadmap, Tasks, Notebook, Models, Tests and Issues — built in and loaded
through the same contract a third party's plugin would use. Further
plugins load from folders on this machine or from a marketplace, a git
repository holding an index. Which plugins are on, and in what order, is
the profile. Each data folder has its own, kept in
`.specdriven/settings.yaml` on this machine like the module switches it
replaces.

## Marketpalce

Plugin marketplace will be hosted in 'marketplace' folder on https://github.com/SpecDriven/specdriven-app-release.
Pligins will be store as zip files.

## Plgin deployment

Plugins can be developed in any repository and then deployed to https://github.com/SpecDriven/specdriven-app-release/marketplace

## References

- DeepSeek Harness: <https://github.com/deepseek-ai/deepseek-harness> —
  `docs/architecture.md`, `docs/cordis-primer.md`,
  `docs/subsystems/client-modules.md` (the `dsh.client` declaration, boot
  graph and bundle route) and `docs/user/develop/basic/publish.md`
  (`dsh plugin --profile <p> add <path | git | npm>`)
- Cordis: <https://github.com/cordiverse/cordis>
- Overviews: <https://www.datacamp.com/blog/what-is-deepseek-harness>,
  <https://thenewstack.io/deepseek-harness-open-source-plugins/>

## S1: A built-in module is a plugin like any other @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the app starts
- When: the user opens the Plugins page
- Then: Roadmap, Tasks, Notebook, Models, Tests and Issues are listed as
  built-in plugins, each with its switch, its version and a line saying
  what it adds
- and the tab rail, the shortcuts, the palette groups and the `/api/<module>`
  routes come from the list of mounted plugins — nothing in the core names
  a module, so a built-in and a third-party plugin are told apart only by
  where they were loaded from
- and a built-in plugin can be switched off but never uninstalled

[test: the browser loader registers the tab while the server says it is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L189 ]
[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L31 ]

## S2: A plugin from the marketplace is installed with consent @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a marketplace is configured on the Plugins page
- When: the user picks a plugin from the marketplace list and presses
  Install
- Then: the app shows the plugin's name, version, description, the
  permissions its manifest declares and a note that it runs with the app's
  own rights, and installs nothing until the user confirms
- and on confirmation the plugin is cloned pinned to the ref the index
  names, into `~/.specdriven/plugins/<id>`, its manifest is read, its
  `api` and `minAppVersion` are checked against this app, and it is added
  to the profile; its tab appears at once, without a restart
- and a plugin whose `api` major the app does not speak, or that needs a
  newer app, is refused with the reason and nothing is left behind
- and Update fetches the ref the index now names and remounts the plugin;
  Remove switches it off, unwinds it and deletes its folder
- and Reinstall — or Install of a plugin whose id is installed already,
  from this marketplace or from one that was removed and added back under
  another name — shows the same consent, naming what it replaces, and on
  confirmation swaps a fresh copy at the ref the index names in for the
  installed folder, the plugin on as before
- and in the hosted demo none of this is offered: it installs the Event
  Models from the SpecDriven marketplace itself when it starts, and a
  visitor browses the marketplace, the Event Models marked installed, with
  no Install, Update, Remove or Refresh

[test: a subdirectory plugin is staged and described first, and installed only on confirmation : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L309 ]
[test: a plugin is reinstalled from a marketplace that was removed and added back under another name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L360 ]
[test: a git-url plugin is installed pinned to the ref the index names, updated to the ref it names now, and removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L435 ]
[test: a plugin the app cannot run is refused with the reason and nothing is left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L394 ]
[test: the hosted demo browses the marketplaces, never refreshed, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L517 ]
[test: the hosted demo installs the Event Models from the marketplace at start and lists them on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L544 ]

## S3: A plugin is scaffolded, built and linked from the command line @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the `specdriven` command (the binary, or `bun src/index.ts` in the
  checkout)
- When: the developer runs `specdriven plugin new <id> [dir]`
- Then: a folder with a package.json whose `specdriven` section names the
  id, the `api` version, both halves' sources and the url prefix, a host
  half that registers a route and a browser half that registers a tab, is
  written — and nothing needs installing to build it
- and `specdriven plugin build [dir]` bundles `src/` into `dist/host.js` and
  `dist/client.js`, the browser half over the app's own React, so the
  desktop app can load it
- and `specdriven plugin link <dir>` adds the folder to this machine's
  profile sources, from where the app finds it like any other folder
- and a bad id, a folder already in use or an unknown command are refused
  with the reason

[test: plugin new, build and link take a plugin from nothing to a mounted tab : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L51 ]
[test: plugin new refuses a bad id or a folder in use, and an unknown command shows the usage : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L93 ]

## S4: A plugin loads from a local folder @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a plugin folder exists — under `~/.specdriven/plugins/<id>`, under
  the data repo's `.specdriven/plugins/<id>`, or at a path named in the
  profile's sources
- When: the app starts, or the folder is added while it runs
- Then: the plugin's host half is mounted and its browser half is served
  to the app, and its tab, shortcuts and commands appear
- and a plugin under the data repo asks for trust once before its first
  load, since the repo is pulled from GitHub
- and a plugin written in TypeScript is built by the app itself when the
  server runs on Bun; the desktop app, whose main process runs on node,
  loads a prebuilt `dist/` only and shows "build this plugin with
  `specdriven plugin build`" when there is none
- and while a path source is being edited the app rebuilds and remounts
  that one plugin, and the browser picks up the new build without a reload
- and a plugin that fails to load fails alone: the others keep running and
  the Plugins page shows the error beside its name

[test: a plugin folder under ~/.specdriven/plugins is found, built, mounted and served : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L114 ]
[test: an edit rebuilds the plugin under a new revision and remounts it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L176 ]
[test: a plugin whose folder goes away is unwound and reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L259 ]
[test: a manifest the app cannot accept is refused with the reason, and loads nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L274 ]
[test: a plugin under the data repo asks for trust before its first load : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L317 ]
[test: a plugin that fails to load fails alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L347 ]
[test: a runtime that cannot build loads dist/ only, and refuses a bundle with its own React : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L360 ]
[test: the browser loader imports a served bundle and remounts it at a new revision : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L398 ]

## S5: Each data folder keeps its own plugins @v1 [proposed]

- Given: two data folders on this machine, one showing Roadmap and Tests
  and the other Specs alone
- When: the user switches from one to the other
- Then: the plugins, their tabs and their routes switch with the folder,
  and Settings → Plugins shows the open folder's switches
- and each folder's list is kept under `plugins.repos[<folder>].enabled` in
  `.specdriven/settings.yaml` on this machine, so it stays device-local: it
  does not follow the data repo to another machine or another clone
- and a folder opened for the first time shows Specs alone, and nothing is
  written for it until its plugins change
- and a settings file from before per-folder lists — one device-wide
  `plugins.enabled`, or the old module switches — has that list copied to
  every folder the file already knows, the open one and the recent ones,
  which keep showing what they showed; a folder picked later starts with
  Specs alone
- and an older file that names a folder but holds no list and none of the
  old switches — someone who never changed a plugin — gives the folders it
  knows the default plugins, the Notebook, as that user saw before; a file
  this app has recorded a folder in is never taken for such an older one
- and a plugin found for the first time goes on in the open folder only;
  the other folders list it, switched off
- and uninstalling a plugin takes it off every folder's list

[test: each data folder keeps its own plugins, and a new one shows Specs alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L105 ]
[test: a fresh install shows Specs alone without a settings write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L89 ]
[test: an older device-wide list carries over to the folder that is open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L140 ]
[test: each data folder keeps its own profile; a new one shows Specs alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L252 ]
[test: an older device-wide list is copied to the folders the file knows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L277 ]
[test: an older file with a folder but no plugin list keeps the Notebook on in that folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L154 ]
[test: a folder picked in this app still shows Specs alone after a restart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L167 ]
[test: an older file with folders but no list gives them the defaults; one with repos does not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L313 ]

## S6: Plugin urls serve the app shell @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a plugin is on and has registered a url prefix, `/tests` say
- When: the user opens `/tests/…` from the address bar, a link, or as
  `specdriven://tests/…` from another app
- Then: the app shell is served and the plugin's view opens with the
  sidebar on its tab, the way `/notebook/<entry>` opens an entry
  (engineering-notebook.feature.md "Deep links")
- and with the plugin off the url goes home, and a url no plugin claims
  shows the not-found page

[test: the plugin's urls serve the app shell whether it is on or off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L169 ]

## S7: The marketplace is a git repository with an index @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the Plugins page names one or more marketplaces, each a git url
- When: the user opens the marketplace list, or presses Refresh
- Then: each marketplace is cloned by the first way that works, or pulled
  through the app's own git engine, into `~/.specdriven/marketplaces/<name>`,
  so it works with `gh`, the app's sign-in or the bundled JavaScript git
  alike, and its `marketplace.json` is read
- and the index names the marketplace and its owner and lists plugins,
  each with a name, a description, a version, a category and a source —
  a git url with a ref, or a subdirectory of the marketplace itself
- and the list shows every plugin with those fields, marks the ones
  already installed and offers Install, Update or Remove accordingly
- and an index that does not parse is reported beside the marketplace's
  name and does not hide the other marketplaces

[test: each marketplace is cloned through the git engine and its index listed; a broken one is reported on its own row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L220 ]
[test: marketplace.json is read as an index, and refused when it is not one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L170 ]
[test: Browse reads the cache, and Refresh afterwards pulls : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L124 ]
[test: the settings carry the marketplaces, and GET /api/state lists them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L502 ]

## S8: The profile lists enabled plugins in tab order @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: `.specdriven/settings.yaml` on this machine holds an ordered
  `enabled` list for the open data folder, under `plugins.repos[<folder>]`
  ("Each data folder keeps its own plugins")
- When: the app starts
- Then: exactly the listed plugins are mounted, in that order, and the
  sidebar shows their tabs in that order after Specs
- and a settings file from before plugins, with the `roadmap`, `tasks`,
  `notebook`, `models`, `testing` and `issues` switches, is read as the
  matching list once and works as it did
- and an id in the list that no source provides is reported on the
  Plugins page and otherwise ignored
- and the hosted demo starts with Specs alone: the built-in plugins, and
  the Event Models it installs from the marketplace, are listed on the
  Plugins page switched off, for a visitor to turn on
- and the list is device-local, like the switches it replaces: it does not
  follow the user to another machine

[test: a settings file from before plugins is read as the matching profile : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L89 ]
[test: the profile's order is the tab order, and an id nothing provides is reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L136 ]
[test: a settings file with the old switches is rewritten as the profile on start, and works as it did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L63 ]
[test: PUT /api/state takes the profile whole, or a legacy switch as an alias : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L179 ]
[test: Specs alone, and every other optional setting on except IntelliJ IDEA : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L143 ]
[test: a visitor turns the roadmap on and it lists the dataset's pages : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L128 ]

## S9: Turning a plugin off unwinds everything it registered @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a plugin is on and one of its views has the pane
- When: the user switches it off on the Plugins page
- Then: its tab, its shortcuts, its palette commands and its settings
  section disappear, its routes answer 404 — the server refuses the
  module's requests, not just the UI (tests.feature.md) — and its sync hook
  no longer runs on the tick
- and the open view leaves the pane and the app goes home
- and the plugin's data in the data repo is left alone
- and switching it on again mounts it afresh, with its tab, shortcuts and
  routes back, without a restart

[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L31 ]
[test: switching a folder plugin off unwinds its routes, and on mounts it again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L217 ]
[test: effects unwind last-in-first-out, and only once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/kernel.test.ts#L21 ]
[test: a plugin waits for its injects and unmounts when one goes away : https://github.com/SpecDriven/specdriven-app/blob/main/tests/kernel.test.ts#L43 ]

## S10: The marketplaces are refreshed on start when external plugins are installed @v1 [proposed]

- Given: the Plugins page names the default SpecDriven marketplace
  (`https://github.com/SpecDriven/plugins`), under whatever name
- When: the app starts
- Then: the default marketplace is pulled once in the background, even
  with only the built-in plugins, so the agent plugins and the update
  check below read its index as it is now
- and when a plugin from outside the app is installed — from a
  marketplace, linked from a folder, or in the data folder's own plugins
  folder — every other marketplace the Plugins page names is pulled too,
  so the marketplace list offers Update for a plugin whose index row moved
  on since the last start, without pressing Refresh
- and the start does not wait for it: the app opens at once, and the agent
  plugins installed for the commands found on this machine
  (agents/agent-plugins.feature.md) read the index once the pull is over
- and a marketplace that cannot be reached keeps the copy it had, is
  logged, and does not hold up the others
- and neither the hosted app, the hosted demo nor the read-only site pulls
  at all

[test: a start pulls the marketplaces when an external plugin is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L624 ]
[test: a start leaves the marketplaces alone with the built-ins only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L626 ]
[test: the start's refresh pulls the default marketplace always and the others only when an external plugin is installed, and logs one it cannot reach : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L206 ]
[test: the default marketplace is known by its url, whatever its name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L27 ]

## S11: Refresh clones a marketplace afresh when its copy can no longer be pulled @v1 [proposed]

- Given: a marketplace's copy in `~/.specdriven/marketplaces/<name>` can no
  longer be pulled — its upstream history was rewritten ("refusing to merge
  unrelated histories"), or a checkout was left half-done
- When: the user presses Refresh on the Plugins page — offered whether or
  not the marketplace list has been browsed yet
- Then: the copy is thrown away and the marketplace is cloned again by the
  first way that works, its index read and listed, and Refresh pulls again
  from then on
- and there is no separate Re-clone action: Refresh does it when needed
- and the fresh clone takes the old copy's place only once it is whole: a
  remote that cannot be reached is reported beside the marketplace and the
  old copy stays
- and installed plugins are left as they are
- and the hosted demo neither pulls nor clones again; the hosted app pulls
  but never throws a copy away to clone it afresh

[test: Refresh clones a marketplace afresh when its history was rewritten upstream, and one that cannot be reached keeps its copy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L270 ]
[test: offers Refresh before the list is browsed, and no Re-clone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L106 ]
[test: Refresh before browsing opens the list with a single pulling read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L114 ]
[test: the hosted demo browses the marketplaces, never refreshed, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L517 ]

## S12: A tab's sidebar uses the section head @v1 [proposed]

- Given: a plugin's tab in the sidebar — Roadmap, Tasks, Notebook, Issues,
  Tests, Fix this app
- When: the user looks at its list
- Then: it opens with the same section head the Specs tree has: the tab's
  name in small capitals (a link to the tab's overview where it has one —
  Tests, Issues), a Filter chip where the list has facets (issue state,
  tasks open or done), and the tab's "new" actions as quiet icon buttons
  with tooltips at the head's right edge — never a row of buttons of its
  own
- And: `c` on the tab creates the tab's own thing — a page, a task, an
  entry, a case, an issue, a fix — and the tab's tooltip in the tab bar
  carries its `g` chord
- And: the list is a keyboard tree like the Specs tree: ↑ ↓ Home End walk
  its rows, Shift+F10 opens the focused row's ⋯ menu, and a row opened from
  elsewhere — a deep link, the command palette — is scrolled into view
- And: clicking the tab only turns the sidebar to the list; the head's
  link and the rows open the pane

## S13: A tab's rows use the sidebar row @v1 [proposed]

- Given: a plugin's tab in the sidebar
- When: the user reads its list
- Then: every row has the shape the Specs tree's rows have — a leading
  glyph, the name, a trailing mark, the ⋯ menu — on the same padding, the
  open row filled as the open feature's is, and what the row carries takes
  a slot rather than a shape of its own: a roadmap page's assignee as an
  avatar at the end, a task's done mark at the start with the name muted
  rather than struck through, an entry's kind as a glyph over a line with
  its author and day, an issue's number at the start, a fix's status as a
  glyph
- And: a plan's progress is the same ring a feature's publish rollup is,
  never a count in text
- And: the tab bar is one row however many tabs there are — the open tab
  shows its name, the others their icon with a tooltip, except in the
  drawer on a small screen, where every tab keeps its name

## S14: Plugins with an update are offered in a dialog at start @v1 [proposed]

- Given: a plugin installed from a marketplace whose index row, after the
  start's refresh, names a newer version than the one installed — or pins
  it to another ref
- When: the app starts
- Then: once the refresh is over a "Plugin updates" dialog lists each such
  plugin with the version installed and the version offered, every one
  ticked
- and Update fetches the ticked plugins one after another the way the
  marketplace list's Update does — their tabs and routes are remounted
  without a restart — showing each one's progress and, beside it, why one
  failed while the others still update
- and Not now closes the dialog and changes nothing; it is offered again at
  the next start, and not again in this run of the app, a reload included
- and with nothing to update, the marketplace not reachable, a plugin
  linked from a folder or shipped with the data folder, or in the hosted
  app, the hosted demo or the read-only site, no dialog appears

[test: a plugin whose index row is newer, or pinned to another ref, is offered; nothing else is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L52 ]
[test: versions compare part by part, a pre-release before its release : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L18 ]
[test: the start's check reads the installed plugins against the indexes it is handed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L78 ]
[test: a start pulls the marketplaces when an external plugin is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L624 ]
[test: lists each plugin ticked, with the version installed and the one offered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdatesDialog.test.tsx#L84 ]
[test: Update runs the ticked plugins in order and closes once all are updated : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdatesDialog.test.tsx#L95 ]
[test: a failure is shown beside its row, the others still update, and Close answers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdatesDialog.test.tsx#L107 ]
[test: Not now updates nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdatesDialog.test.tsx#L125 ]
[test: the offer is empty when the server refuses or answers none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdatesDialog.test.tsx#L133 ]

## S15: A plugin repository is cloned by the first way that works @v1 [proposed]

- Given: a marketplace, or a plugin the index names by a git url, is not
  on this computer yet — or Refresh is cloning a marketplace afresh
- When: the app clones it
- Then: it tries the git installed on the computer first, then the git
  built into the app, then the GitHub CLI, moving on to the next when one
  fails, whatever Git access Settings names
- and a way this computer does not have is skipped: no git installed
  leaves the built-in git alone, and the hosted app never uses the GitHub
  CLI
- and what a failed try left behind is removed before the next one starts
- and when every way fails the error names each one with its reason

[test: the system git is tried first, then the built-in git, then the GitHub CLI : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginRepoClone.test.ts#L47 ]
[test: a way that fails hands over to the next, which starts from an empty folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginRepoClone.test.ts#L71 ]
[test: when every way fails the error names each with its reason, and nothing is left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginRepoClone.test.ts#L90 ]
[test: a plugin repository is cloned whichever engine Settings names, and with no git installed at all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginRepoClone.test.ts#L106 ]
