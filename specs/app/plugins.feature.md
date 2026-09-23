# Plugins

**Assigned:** David

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
the profile, kept in `.specdriven/settings.json` on this machine like the
module switches it replaces.

## A built-in module is a plugin like any other @v1 [proposed]

- Given: the app starts
- When: the user opens Settings → Plugins
- Then: Roadmap, Tasks, Notebook, Models, Tests and Issues are listed as
  built-in plugins, each with its switch, its version and a line saying
  what it adds
- and the tab rail, the shortcuts, the palette groups and the `/api/<module>`
  routes come from the list of mounted plugins — nothing in the core names
  a module, so a built-in and a third-party plugin are told apart only by
  where they were loaded from
- and a built-in plugin can be switched off but never uninstalled

[test: the browser loader registers the tab while the server says it is on : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L178 ]
[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L21 ]

## A plugin from the marketplace is installed with consent @v1 [proposed]

- Given: a marketplace is configured in Settings → Plugins
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
- and none of this is offered in the hosted demo

## A plugin loads from a local folder @v1 [proposed]

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
  Settings → Plugins shows the error beside its name

## Plugin urls serve the app shell @v1 [proposed]

- Given: a plugin is on and has registered a url prefix, `/tests` say
- When: the user opens `/tests/…` from the address bar, a link, or as
  `specdriven://tests/…` from another app
- Then: the app shell is served and the plugin's view opens with the
  sidebar on its tab, the way `/notebook/<entry>` opens an entry
  (engineering-notebook.feature.md "Deep links")
- and with the plugin off the url goes home, and a url no plugin claims
  shows the not-found page

[test: the plugin's urls serve the app shell whether it is on or off : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L158 ]

## The marketplace is a git repository with an index @v1 [proposed]

- Given: Settings → Plugins names one or more marketplaces, each a git url
- When: the user opens the marketplace list, or presses Refresh
- Then: each marketplace is cloned or pulled through the app's own git
  engine into `~/.specdriven/marketplaces/<name>`, so it works with `gh`,
  the app's sign-in or the bundled JavaScript git alike, and its
  `marketplace.json` is read
- and the index names the marketplace and its owner and lists plugins,
  each with a name, a description, a version, a category and a source —
  a git url with a ref, or a subdirectory of the marketplace itself
- and the list shows every plugin with those fields, marks the ones
  already installed and offers Install, Update or Remove accordingly
- and an index that does not parse is reported beside the marketplace's
  name and does not hide the other marketplaces

## The profile lists enabled plugins in tab order @v1 [proposed]

- Given: `.specdriven/settings.json` on this machine holds a `plugins`
  section with an ordered `enabled` list
- When: the app starts
- Then: exactly the listed plugins are mounted, in that order, and the
  sidebar shows their tabs in that order after Specs
- and a settings file from before plugins, with the `roadmap`, `tasks`,
  `notebook`, `models`, `testing` and `issues` switches, is read as the
  matching list once and works as it did
- and an id in the list that no source provides is reported in Settings →
  Plugins and otherwise ignored
- and the hosted demo enables every built-in plugin, so a visitor sees the
  whole app
- and the list is device-local, like the switches it replaces: it does not
  follow the user to another machine

[test: a settings file from before plugins is read as the matching profile : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L77 ]
[test: the profile's order is the tab order, and an id nothing provides is reported : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L125 ]
[test: a settings file with the old switches is rewritten as the profile on start, and works as it did : https://github.com/SpecDriven/specdriven_app/blob/main/tests/pluginProfile.test.ts#L62 ]
[test: PUT /api/state takes the profile whole, or a legacy switch as an alias : https://github.com/SpecDriven/specdriven_app/blob/main/tests/pluginProfile.test.ts#L101 ]

## Turning a plugin off unwinds everything it registered @v1 [proposed]

- Given: a plugin is on and one of its views has the pane
- When: the user switches it off in Settings → Plugins
- Then: its tab, its shortcuts, its palette commands and its settings
  section disappear, its routes answer 404 — the server refuses the
  module's requests, not just the UI (tests.feature.md) — and its sync hook
  no longer runs on the tick
- and the open view leaves the pane and the app goes home
- and the plugin's data in the data repo is left alone
- and switching it on again mounts it afresh, with its tab, shortcuts and
  routes back, without a restart

[test: the Tests switch mounts the plugin's routes and unmounts them again : https://github.com/SpecDriven/specdriven_app/blob/main/tests/plugins.test.ts#L21 ]
[test: effects unwind last-in-first-out, and only once : https://github.com/SpecDriven/specdriven_app/blob/main/tests/kernel.test.ts#L21 ]
[test: a plugin waits for its injects and unmounts when one goes away : https://github.com/SpecDriven/specdriven_app/blob/main/tests/kernel.test.ts#L43 ]
