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
Specs, Roadmap, Tasks, Notebook, Models, Tests and Issues — built in and
loaded through the same contract a third party's plugin would use. Further
plugins load from folders on this machine or from a marketplace, a git
repository holding an index. Which plugins are on, and in what order, is
the profile. Each data folder has its own, kept in
`.specdriven/settings.yaml` on this machine like the module switches it
replaces.

## Releases

A plugin in the marketplace is released in versions. Beside its source in
the marketplace repository, a `release` folder holds one zip per version
and `releases.json`, which lists them with their checksums and what each
needs of the app. The app installs from those zips, so any version in the
folder can be installed and an older one put back. A release is cut with
`specdriven plugin release` and published by merging it into the
marketplace's `main`. A plugin with no `release` folder is installed as
its folder is.

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

[test: the browser loader registers the tab while the server says it is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L223 ]
[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L34 ]

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
- and a plugin the marketplace releases in versions is unpacked from the
  zip of one version instead of cloned or copied: the newest this app can
  run, or the one picked
- and Reinstall — or Install of a plugin whose id is installed already,
  from this marketplace or from one that was removed and added back under
  another name — shows the same consent, naming what it replaces, and on
  confirmation swaps a fresh copy at the ref the index names in for the
  installed folder, the plugin on as before
- and in the hosted demo none of this is offered: it installs the Event
  Models from the SpecDriven marketplace itself when it starts, and a
  visitor browses the marketplace, the Event Models marked installed, with
  no Install, Update, Remove or Refresh

[test: a subdirectory plugin is staged and described first, and installed only on confirmation : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L342 ]
[test: a plugin is reinstalled from a marketplace that was removed and added back under another name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L393 ]
[test: a git-url plugin is installed pinned to the ref the index names, updated to the ref it names now, and removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L468 ]
[test: a plugin the app cannot run is refused with the reason and nothing is left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L427 ]
[test: the hosted demo browses the marketplaces, never refreshed, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L679 ]
[test: the hosted demo installs the Event Models from the marketplace at start and lists them on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L707 ]

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

[test: plugin new, build and link take a plugin from nothing to a mounted tab : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L56 ]
[test: plugin new refuses a bad id or a folder in use, and an unknown command shows the usage : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L98 ]

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
[test: a plugin whose folder goes away is unwound and reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L266 ]
[test: a manifest the app cannot accept is refused with the reason, and loads nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L281 ]
[test: a plugin under the data repo asks for trust before its first load : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L324 ]
[test: a plugin that fails to load fails alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L354 ]
[test: a runtime that cannot build loads dist/ only, and refuses a bundle with its own React : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L367 ]
[test: the browser loader imports a served bundle and remounts it at a new revision : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginSources.test.ts#L405 ]

## S5: Each data folder keeps its own plugins @v1 [proposed]

- Given: two data folders on this machine, one showing Roadmap and Tests
  and the other Specs alone
- When: the user switches from one to the other
- Then: the plugins, their tabs and their routes switch with the folder,
  and Settings → Plugins shows the open folder's switches
- and each folder's list is kept under `plugins.repos[<folder>].enabled` in
  `.specdriven/settings.yaml` on this machine, so it stays device-local: it
  does not follow the data repo to another machine or another clone
- and a folder opened for the first time gets its list from the data it
  holds ("A folder's plugins are picked from its data the first time it
  opens"); one with nothing in it shows Specs alone
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

[test: each data folder keeps its own plugins, and a new one shows Specs alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L114 ]
[test: a fresh install shows Specs alone without a settings write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L97 ]
[test: an older device-wide list carries over to the folder that is open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L161 ]
[test: each data folder keeps its own profile; a new one shows Specs alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L286 ]
[test: an older device-wide list is copied to the folders the file knows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L312 ]
[test: an older file with a folder but no plugin list keeps the Notebook on in that folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L182 ]
[test: a folder picked in this app still shows Specs alone after a restart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L198 ]
[test: an older file with folders but no list gives them the defaults; one with repos does not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L357 ]

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

[test: the plugin's urls serve the app shell whether it is on or off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L203 ]

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
- and a plugin kept in a subdirectory with a `release` folder is listed
  with the versions that folder holds, on a version chip beside its
  actions
- and an index that does not parse is reported beside the marketplace's
  name and does not hide the other marketplaces
- and the SpecDriven marketplace is always among them, whatever the
  settings name; any other can be added and removed

[test: each marketplace is cloned through the git engine and its index listed; a broken one is reported on its own row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L253 ]
[test: marketplace.json is read as an index, and refused when it is not one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L185 ]
[test: Browse reads the cache, and Refresh afterwards pulls : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L125 ]
[test: the settings carry the marketplaces, and GET /api/state lists them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L535 ]

## S8: The profile lists enabled plugins in tab order @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: `.specdriven/settings.yaml` on this machine holds an ordered
  `enabled` list for the open data folder, under `plugins.repos[<folder>]`
  ("Each data folder keeps its own plugins")
- When: the app starts
- Then: exactly the listed plugins are mounted, in that order, and the
  sidebar shows their tabs in that order, Specs among them ("Specs is a
  built-in plugin")
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

[test: a settings file from before plugins is read as the matching profile : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L95 ]
[test: the profile's order is the tab order, and an id nothing provides is reported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L157 ]
[test: a settings file with the old switches is rewritten as the profile on start, and works as it did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L70 ]
[test: PUT /api/state takes the profile whole, or a legacy switch as an alias : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L211 ]
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

[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L34 ]
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

[test: a start pulls the marketplaces when an external plugin is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L787 ]
[test: a start leaves the marketplaces alone with the built-ins only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L789 ]
[test: the start's refresh pulls the default marketplace always and the others only when an external plugin is installed, and logs one it cannot reach : https://github.com/SpecDriven/specdriven-app/blob/main/tests/autoInstall.test.ts#L206 ]
[test: the default marketplace is known by its url, whatever its name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L39 ]

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

[test: Refresh clones a marketplace afresh when its history was rewritten upstream, and one that cannot be reached keeps its copy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L303 ]
[test: offers Refresh before the list is browsed, and no Re-clone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L107 ]
[test: Refresh before browsing opens the list with a single pulling read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L115 ]
[test: the hosted demo browses the marketplaces, never refreshed, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L679 ]

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

[test: `c` makes a feature from home and a scenario on a feature page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L363 ]

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
- and a plugin released in versions is offered the newest version this
  app can run, and is not listed while the user holds it at an older one
- and with nothing to update, the marketplace not reachable, a plugin
  linked from a folder or shipped with the data folder, or in the hosted
  app, the hosted demo or the read-only site, no dialog appears

[test: a plugin whose index row is newer, or pinned to another ref, is offered; nothing else is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L64 ]
[test: versions compare part by part, a pre-release before its release : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L30 ]
[test: the start's check reads the installed plugins against the indexes it is handed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L91 ]
[test: a start pulls the marketplaces when an external plugin is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L787 ]
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

## S16: A plugin is released as a versioned zip in its release folder @v1 [proposed]

- Given: a plugin's folder in a marketplace repository, its `package.json`
  naming version 1.19.0
- When: its author runs `specdriven plugin release` in the folder
- Then: the plugin is built and `release/<id>-1.19.0.zip` is written beside
  its source, holding what runs — `package.json`, the built halves under
  `dist/` without their source maps, the README and the licence — and none
  of its sources
- and `release/releases.json` lists the version on top with the zip's
  name, its SHA-256, its size, the day, the plugin api it speaks, the
  oldest app it runs in and the note given with `--notes`
- and the same files always give the same zip
- and a version that is in the folder already is refused: a release is
  never written again
- and the newest ten versions are kept unless `--keep` says otherwise; an
  older one's zip and row are dropped
- and committing the folder to the marketplace is what publishes the
  version: the marketplace list then offers every version in it, newest
  first

[test: plugin release adds package.json's version to the release folder as a zip, and never writes one again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L115 ]
[test: plugin release keeps the newest ten versions when --keep does not say otherwise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginCli.test.ts#L175 ]
[test: a zip unpacks to the files it was written from : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L50 ]
[test: the same files give the same bytes, whatever order they are handed in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L61 ]
[test: the system's unzip reads a zip this app wrote : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L146 ]
[test: this app reads a zip the system's zip wrote, folders and all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L155 ]
[test: a zip is unpacked into a folder, its subfolders made : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L67 ]
[test: the checksum is the one every zip tool computes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L74 ]
[test: releases.json is read newest first, and refused when it is not a list of zips with checksums : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L111 ]
[test: a release is named after its plugin and version, a scope folded into the name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L149 ]
[test: the marketplace lists a plugin's released versions, newest first, with why this app cannot run one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L212 ]

## S17: Any released version of a plugin can be installed @v1 [proposed]

- Given: a marketplace plugin with several versions in its release folder
- When: the user opens the version chip on its row in the marketplace list
- Then: the menu lists every version, newest first, each with its date and
  note, and the installed one is ticked
- and a version this app cannot run — it needs a newer SpecDriven, or
  another plugin api — is greyed with the reason, and a withdrawn version
  is not listed unless it is the one installed
- and before an install the version picked is the one Install names in its
  consent and fetches; untouched, that is the newest this app can run
- and on an installed plugin, picking another version switches to it at
  once: its zip is unpacked in place of the installed folder, and the
  plugin's tab and routes are remounted without a restart
- and Reinstall unpacks the installed version afresh, not the newest
- and a plugin with no release folder has no chip and installs from its
  folder as before

[test: the chip names the newest version this app can run, and its menu the others with why one cannot be picked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L210 ]
[test: the version picked is the one Install stages and confirms : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L233 ]
[test: an installed plugin switches to the version picked at once; held, it says so and Update names where it goes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L259 ]
[test: the menu's rows say what each version is; the newest offered passes over what this app cannot run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L303 ]
[test: a switch asks the server for the version by the plugin's id, a scope encoded : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L328 ]
[test: any released version can be installed, and an older one is held with no update offered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L282 ]

## S18: A plugin rolled back to an older version stays there until the user updates it @v1 [proposed]

- Given: a plugin installed at 1.19.0, with 1.18.0 in its release folder
- When: the user picks 1.18.0 from its version chip
- Then: the plugin runs 1.18.0, its row says it is held, and it remembers
  the version it replaced
- and the start's "Plugin updates" dialog does not offer 1.19.0 while the
  plugin is held
- and the row's Update names where it goes — "Update to v1.19.0" — and
  pressing it moves the plugin there and lets go of the hold
- and the older version is unpacked from the marketplace's copy on this
  computer, so it is put back with no network
- and a plugin on a version that was withdrawn since is offered the newest
  good one at the next start, held or not, and its row offers to move
  there

[test: a plugin rolled back is held at that version and remembers the one it replaced; Update moves it on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L308 ]
[test: a version this machine already has the zip of is put back without the network : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L333 ]
[test: a plugin on a version that was withdrawn is offered the newest good one at start, held or not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L350 ]
[test: a released plugin is offered the newest version this app runs — unless held — and moved off a withdrawn one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L201 ]
[test: an install record keeps the checksum, the version it replaced and the hold : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L188 ]
[test: on the newest version this app runs there is no Update; on a withdrawn one, Move names the way off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L289 ]

## S19: A plugin that fails after an update offers its previous version @v1 [proposed]

- Given: a plugin that was moved from 1.18.0 to 1.19.0 and now fails to
  load
- When: the user opens the Plugins page
- Then: under the plugin's error a "Roll back to v1.18.0" link puts the
  version it replaced back, held there like any version picked by hand
- and the version put back loads at once, without a restart, its tab and
  routes back; so does an update past the version that failed
- and a roll back the app cannot do says why under the plugin
- and a plugin that loads, one that was never on another version, and the
  hosted demo offer no such link
- and a held plugin's row says so beside its version

[test: a version that fails to load is rolled back to the one it replaced, which loads without a restart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L405 ]
[test: a plugin that fails after a version change offers the version it replaced, and nothing else does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginsPageRollback.test.tsx#L125 ]
[test: a roll back the server refuses says why under the plugin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginsPageRollback.test.tsx#L150 ]
[test: a held plugin says so beside its version, and the hosted demo offers no roll back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginsPageRollback.test.tsx#L162 ]

## S20: A release whose checksum does not match is refused @v1 [proposed]

- Given: a release zip in the marketplace's copy that is not the file
  `releases.json` lists — its SHA-256 differs, it holds another version
  than its row says, or it is not there
- When: the user installs that version, or switches to it
- Then: the app refuses with the reason and nothing is installed: no
  folder is left under `~/.specdriven/plugins`, and a plugin installed
  already keeps running the version it had
- and a zip that carries anything but plain files under its own folder — a
  symlink, an encrypted entry, a path that leaves the folder, more bytes
  than it declares — is refused the same way, before a file is written
- and a `releases.json` the app cannot read is reported on that plugin's
  row, where nothing of it installs, and does not hide the other plugins

[test: a zip that is not what releases.json says is refused and nothing is left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L371 ]
[test: a zip with an entry that leaves the folder is refused and nothing is written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L88 ]
[test: a symlink, an encrypted entry, a zip64 zip and an unknown compression are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L101 ]
[test: an entry whose bytes are not what it declares is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L122 ]
[test: a zip past the limits, a truncated one and a file that is no zip are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L137 ]
[test: a zip with more entries than a release could hold is refused by default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L165 ]
[test: a path that would leave the folder is not an entry : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginZip.test.ts#L79 ]

## S21: An app too old for the newest release installs the newest one it can run @v1 [proposed]

- Given: a plugin whose newest release needs a newer SpecDriven than this
  one, with an older release this app can run
- When: the user presses Install or Update, or the app installs an agent's
  plugin on its own
- Then: the newest version this app can run is installed, not the newest
  one listed, and the consent names that version
- and the start's "Plugin updates" dialog offers that version and no newer
  one
- and a pre-release version is never chosen this way: it is installed only
  when picked
- and when no version runs in this app the install is refused with what
  the newest one needs

[test: Install unpacks the newest version this app can run from its zip : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginReleases.test.ts#L257 ]
[test: a release this app cannot run says why; the newest it can run is what Install means : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginUpdates.test.ts#L167 ]

## S22: An empty repository is initialized as a marketplace @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

A company keeps its own plugins in a marketplace of its own — a private
repository, say — set up from the app rather than by copying the SpecDriven
one by hand.

- Given: a repository the user has created on its git host, empty or
  holding only what the host put there, and no `marketplace.json`
- When: the user chooses Initialize marketplace… in the Marketplaces section
  of the Plugins page and gives its git url
- Then: the app clones it through its git engine, writes a
  `marketplace.json` named after the repository with no plugins, a README
  saying how to add and release plugins and how users add the marketplace,
  a `plugins` folder, and the CI workflow, scripts and `.gitignore` of the
  SpecDriven marketplace as the app's copy of it has them
- and commits them and pushes `main`, which then tracks the remote's
- and only once the push has gone through is the marketplace added to the
  list and the list opened, showing it with no plugins
- and a README or `.gitignore` already in the repository is kept, as are
  its commits
- and a repository that already has a `marketplace.json`, one already on
  the list, one that cannot be cloned and one that refuses the push are
  each refused with the reason, nothing is added and no working copy is
  left behind
- and the hosted demo and a hosted worker offer no such thing

[test: an empty repository is initialized as a marketplace: index, README and CI pushed to main : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L577 ]

[test: a repository with commits keeps them, and its own README : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L635 ]
[test: a repository that is a marketplace already, or cannot be pushed to, is refused with nothing left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L652 ]
[test: a marketplace's owner is read from its url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L672 ]
[test: the cli engine publishes the first branch of an empty repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L373 ]
[test: a repository that already has a branch gets the commit on that branch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L397 ]
[test: Initialize posts the url and adds the marketplace once the server has pushed it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L387 ]
[test: a refused initialization is shown and nothing is added : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L399 ]
[test: the hosted demo browses the marketplaces, never refreshed, and changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L679 ]

## S23: The SpecDriven marketplace cannot be removed @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the Marketplaces section of the Plugins page
- When: the user looks for a way to remove the SpecDriven marketplace
  (`https://github.com/SpecDriven/plugins`)
- Then: its row is marked Default and has no Remove; every other
  marketplace's row has one
- and settings that leave it out — an empty list saved by an earlier
  version, or a settings file edited by hand — get it back, first on the
  list; one that lists it under another name keeps that name and place
- and a marketplace that took its name for another url is listed under
  that name with a number after it

[test: the settings name the marketplaces, and the SpecDriven index is always among them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L221 ]
[test: the SpecDriven marketplace stays on the list whatever the settings say; the others can go : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplace.test.ts#L546 ]
[test: the SpecDriven marketplace has no Remove; another one does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginMarketplacePage.test.tsx#L380 ]

## S24: Specs is a built-in plugin @v1 [proposed]

- Given: a data folder is open
- When: the user opens the Plugins page
- Then: Specs is listed among the plugins as built in, with a switch like
  Roadmap or the Notebook, and while it is on it is first in the order: it
  has no arrows, and nothing moves above it
- and switching it off takes away the Specs tab, its icon on the rail, the
  `g s` shortcut, the features and scenarios the command palette finds or
  lists under Recent, and the imports into a spec folder; the sidebar
  turns to the first tab still on, and a link to a feature opens home
- and while a feature is open the rail lights Specs, whichever tab the
  sidebar shows, as it lights any tab whose page has the pane
- and the feature files, their routes and the sync are untouched:
  switching Specs off hides the tab, the folder's specs are still synced
- and the tabs stand in the list's order, Specs first while it is on —
  the leftmost tab, even when it is switched on after the others or a
  stored list puts it later; with no tab remembered the first one is shown
- and home is the folder's README.md whether Specs is on or off
- and a list stored before Specs was a plugin, which never names it, is
  read with Specs on and first, as it was then; the first write puts Specs
  into every stored list and marks the file (`plugins.format`), so from
  then on a list without Specs has it switched off
- and with every plugin switched off the sidebar says so and links to the
  Plugins page

[test: Specs is listed first as a built-in plugin, and switching it off holds across a restart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L255 ]
[test: a list from before Specs was a plugin reads with Specs first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L283 ]
[test: switched on after the notebook, Specs is still the first tab, and its tree is the sidebar when picked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L190 ]
[test: Specs, while it is on, is always first in the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/plugins.test.ts#L145 ]
[test: Specs has no arrows and nothing moves above it; the others move : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginsPageRollback.test.tsx#L172 ]
[test: the rail carries the tabs in the list's order, Specs only while it is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L214 ]
[test: with Specs switched off a feature's link opens home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L228 ]
[test: with every plugin switched off the sidebar says where to switch one on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L239 ]
[test: `g s` on a notebook entry turns the sidebar to the tree and opens the README : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L319 ]
[test: the rail's Specs icon on a notebook entry does the same as `g s` : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L331 ]
[test: the palette finds a feature while Specs is on, and none while it is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L343 ]
[test: switching Specs off with a feature open goes home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L379 ]
[test: the palette's Recent list offers the feature last opened, and not while Specs is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L449 ]
[test: a Recent entry kept from before Specs was a tab still opens its feature : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L470 ]
[test: the palette offers the Aha! import while its switch and Specs are on, and not with Specs off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L484 ]
[test: a url that is not a feature's is not the Specs tab's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsRoute.test.ts#L25 ]
[test: a tab with no home view always has its door, and the door is home : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsRoute.test.ts#L29 ]
[test: switched off with a feature open and on again, Specs has let the feature go : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L393 ]
[test: a reload tick shows the open feature as the folder now has it, and nothing of it with Specs off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L518 ]

## S25: A folder's plugins are picked from its data the first time it opens @v1 [proposed]

- Given: a data folder this app has not looked at yet — a new one, or one
  in use since before this rule
- When: the folder is opened
- Then: each built-in plugin with data in the folder is switched on: Specs
  for a feature file under `specs/`, Roadmap, Tasks and the Notebook for a
  page in their folders, Tests for a test case or a plan
- and Specs is switched off when the folder holds no feature and a plugin
  that is on has data, so a folder of notebook entries opens on the
  Notebook alone, with the folder's README.md in the pane
- and a folder with no data keeps its list — Specs alone, for a new one
- and the plugins that were on stay on, in their order; the ones switched
  on follow them
- and the folder is recorded as looked at
  (`plugins.repos[<folder>].checked`), so this happens once: what is
  switched on or off in Settings → Plugins afterwards holds, whatever
  data arrives later
- and the hosted demo and the read-only site are not looked at: the demo
  starts with Specs alone, the site shows Specs and the Roadmap
- and plugins from the marketplace take no part: only the built-in ones
  are picked by their data

[test: the list is made from the data once, what has data goes on and Specs goes off where another has some : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L295 ]
[test: a folder of notebook entries opens on the Notebook alone, and one with specs too shows both : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L340 ]
[test: a folder used before the check is looked at once, its tasks go on and its empty Specs off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L375 ]
[test: a folder of notebook entries opens on the Notebook alone, with the README in the pane : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L177 ]
[test: a folder with features and notebook entries shows both, Specs first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsPlugin.test.tsx#L205 ]
[test: a folder has Specs data once a feature file lies under its specs folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pluginProfile.test.ts#L325 ]

## S26: Sync waits for the tabs that refresh on the tick @v1 [proposed]

- Given: a data folder is open with several plugins on, and a feature is
  open
- When: the reload tick runs — on its interval, or from the Sync button
- Then: each plugin's tab refreshes what it shows, and the Specs tab reads
  the open feature again unless something on its page is being edited
- and Sync's progress lasts until every tab that asked to be waited for
  has finished; a tab that asks for nothing is not waited for
- and a tab whose refresh fails is logged, and stops neither the other
  tabs nor Sync

[test: settle waits for every listener, in order, and logs the one that fails : https://github.com/SpecDriven/specdriven-app/blob/main/tests/kernel.test.ts#L130 ]
