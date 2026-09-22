# Plugin architecture

**Assigned:** David

A proposal for turning SpecDriven into a small core plus plugins, in the
style of DeepSeek Harness. It refines
[Modularity, Plugins, Customization](modularity.md): that page asks for "a
core with extension points", versioned interfaces and room for "vibe coded
plug-ins"; this one says what the extension points are, how a plugin is
packaged, loaded and switched off, and in what order the existing modules
become plugins. The behaviour is specified in
[specs/app/plugins.feature.md](../specs/app/plugins.feature.md).

## Why

Every optional module — Roadmap, Tasks, Notebook, Models, Tests, Issues —
is hand-wired in about seven places of the app: the sidebar tab union and
the tab rail in `App.tsx`, the main-pane chain of conditions, the module
flags on the client and in `state.ts`, the shortcut and command-palette
arrays, the flat route table in `api.ts`, the shell paths in `index.ts` and
the sync tick in `reload.ts`. Adding a module means touching all of them,
and nothing outside the repository can add one at all. The first plugins
are therefore those tabs themselves: once they load through the same
contract a third party would use, the contract is proven.

## What we borrow from DeepSeek Harness

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) is an
agent runtime whose slogan is *everything is a plugin*. It is built on the
[Cordis](https://github.com/cordiverse/cordis) framework, and five of its
ideas carry over directly:

- **Everything is a plugin.** The model adapter, the tool registry, the
  session log and the web UI are all plugins mounted beside each other;
  there is no privileged core to patch. For us: each module tab is a
  plugin, and the core keeps only the kernel, the shell (sidebar frame,
  main-pane slot, Settings dialog frame, command-palette frame) and the
  shared services — data repo, git, Automerge store, settings, sync tick,
  agent sessions. Built-in plugins use exactly the API third-party ones do.
- **A plugin is `apply(ctx, config)` plus `inject`.** It names the services
  it needs and waits until they exist; it never imports another plugin's
  module directly.
- **Every registration is a reversible effect.** A service, a route, a
  listener or a tab is installed through `ctx.effect` or `ctx.on` and
  returns a disposer, so unloading a plugin unwinds everything it did.
  Turning a module switch off *is* disposing its scope: its routes answer
  404 as they do today, its tab, shortcuts and palette entries vanish, its
  sync hook stops. Turning it on mounts it again. No restart.
- **Dual-half packages.** A plugin has a host half and a browser half; the
  browser half is exported as `./client`, declared in `package.json`, and
  served to the browser by the host from a bundle route.
- **A profile is an ordered list; installation is one verb.** Harness
  composes a running system from ordered bundles and installs with
  `dsh plugin --profile <p> add <local path | git | npm>`. For us the
  profile is the ordered `plugins.enabled` list in settings, demo mode is
  the profile that enables everything, and the marketplace is a git
  repository holding an index.

What we deliberately do **not** borrow yet: Harness's session log, agent
loop and capability seams. Coding-agent adapters (Claude, Pi, Codex) are a
natural later plugin family, but the tabs come first.

## The kernel

`src/plugins/kernel.ts`, about 250 lines, shared unchanged by the Bun
server, the Electron main process (built for node) and the browser. No
dependency: the `cordis` package fits semantically but brings its own
loader, schema library and fork semantics, and the subset we need — one
context, effects, inject ordering, four event modes — is small enough to
own. The surface keeps Cordis's vocabulary so a later swap is mechanical.

```ts
export type Disposer = () => void | Promise<void>;

export interface Plugin<C extends Context = Context, Cfg = void> {
  name: string;                                   // "tests", "@acme/jira-tab"
  inject?: readonly (keyof Services & string)[];  // services that must exist before apply
  Config?: { parse(raw: unknown): Cfg };          // validate + defaults; a throw refuses the load
  apply(ctx: C, config: Cfg): void | Promise<void>;
}
export interface Services {}   // declaration-merged by whoever provides a service
export interface Events {}     // declaration-merged; the listener type drives emit/serial/bail

export interface Scope {
  readonly name: string; readonly parent: Scope | null;
  readonly status: "pending" | "active" | "disposed"; readonly error?: Error;
  dispose(): Promise<void>;    // children first, effects last-in-first-out, idempotent
}

export class Context {
  readonly scope: Scope; readonly root: Context;
  provide<K extends keyof Services>(key: K, value: Services[K]): Disposer;  // throws on duplicate
  get<K extends keyof Services>(key: K): Services[K] | undefined;
  need<K extends keyof Services>(key: K): Services[K];                      // throws when absent
  effect(run: () => Disposer): Disposer;                                    // runs now, unwound on dispose
  on<K extends keyof Events>(event: K, listener: Events[K]): Disposer;
  emit(event, ...args): void;                 // fire and forget; errors logged, never thrown
  serial(event, ...args): Promise<void>;      // awaited, in registration order
  parallel(event, ...args): Promise<void>;    // awaited, concurrent
  bail(event, ...args): Promise<unknown>;     // first non-undefined result wins
  plugin<Cfg>(plugin: Plugin<this, Cfg>, config?: unknown): Scope;  // fork a child scope
  ready(fn: () => void | Promise<void>): Disposer;                   // after start() and injects satisfied
  start(): Promise<void>; stop(): Promise<void>;                     // stop disposes every scope, root last
}
```

Rules:

- an unmet `inject` leaves the scope *pending* until the service appears;
  a service whose provider is disposed returns its dependents to pending
  with their effects unwound, and re-applies them when it comes back
- disposal runs disposers in reverse order; a throwing disposer is logged
  and never stops the unwind
- a plugin that throws in `apply` is marked disposed with its error, its
  partial effects are unwound and the error shows in Settings → Plugins;
  its siblings live and the app never crashes

## Extension points

Each service replaces one hand-wired place. Every call returns a disposer.

| Half | Service | What it replaces today |
| --- | --- | --- |
| host | `ctx.routes.add(method, pattern, handler)` | the flat route table in `api.ts`; `matchApi` walks the registry, core first, then plugins in order; a plugin's routes are gone while it is off, which the `withTesting` and `withIssues` wrappers do by hand today |
| host | `ctx.shell.add(pathPrefix)` | the per-module `"/tests": index` rows in `index.ts`; a plugin path serves the app shell, on the desktop too |
| host | `ctx.settings.field({ key, default, demo? })` | the per-module keys in `state.ts` and the field list `GET /api/state` returns; the module switch itself is not a plugin field, the kernel owns `plugins.enabled` |
| host | `ctx.sync.drift(async (dir) => …)` | the hand-wired branches of `syncDataDir` in `reload.ts`: `ensurePageFolder`, `importPageDrift` and `importModelDrift` move into their plugins; the specs drift pass stays in the core |
| host | `ctx.docs.register(kind, schema)` | the per-module document types in `automerge/schema.ts`; the store stays generic |
| host | `ctx.capabilities.set(id, flags)` | the capability flags the client reads from `GET /api/state` (`issuesRepo`, `canRunCodex`…) |
| client | `ctx.tabs.add({ id, title, icon, order, when?, Sidebar, Pane })` | the sidebar tab union, `PAGE_TABS`, the rail JSX and the main-pane chain in `App.tsx` |
| client | `ctx.shortcuts.add(shortcut)` | the inline `go-tests`, `go-issues`, `go-models` entries, feeding the existing `Shortcut` registry in `shortcuts.tsx` |
| client | `ctx.palette.add(group, () => items)` | the per-module spreads that build the command palette |
| client | `ctx.settingsUI.add({ section, Component })` | the per-module drafts in the Settings dialog |
| client | `ctx.router.add({ prefix, match, path })` | the hand-written route parsing in `route.ts`; deep links per [navigation](../specs/ui/navigation.feature.md), `specdriven://` included, keep working |
| both | `ctx.settings.get(key)` and the `settings/changed` event | flag hydration from `/api/state` |

`ctx.tabs.add` is the one contract a tab plugin needs; everything else is
opt-in. The per-module client stores (`testing.ts`, `issues.ts`,
`eventModel/models.ts`, all `useSyncExternalStore` singletons) keep their
shape and move into their plugins, gaining a `dispose()` so the singleton
resets when the scope unwinds.

Core wiring changes: the literal route array becomes the core plugin
calling `ctx.routes.add` per row, `matchApi` delegates to the registry and
`handleApi` keeps its signature, so `index.ts` and the Electron main process
need no change for routing. `syncDataDir` calls the `sync/drift` event
after the specs drift pass and its per-module options go away. On the
client a `<PluginBoundary>` at the root of `App` mounts each enabled
browser half and disposes the ones that disappear; the rail, the pane, the
shortcut and palette arrays and `parseRoute` each consult the registries
first, and each migrated module deletes its hardcoded branch.

## Package shape

```
<plugin>/
  package.json        # name, version, specdriven: { id, api, host, client, … }
  src/host.ts         # export default { name, inject, apply } as Plugin
  src/client.tsx      # export default { name, inject, apply } as Plugin (React)
  dist/host.js        # prebuilt ESM, loadable by node (the Electron main process)
  dist/client.js      # prebuilt ESM, React and the client API left external
```

```json
{
  "name": "@specdriven/plugin-tests",
  "version": "0.1.0",
  "type": "module",
  "exports": { ".": "./dist/host.js", "./client": "./dist/client.js" },
  "specdriven": {
    "id": "tests",
    "api": "1",
    "displayName": "Test management",
    "description": "Test cases, plans and runs on their own tab",
    "host": "./dist/host.js",
    "client": "./dist/client.js",
    "src": { "host": "./src/host.ts", "client": "./src/client.tsx" },
    "minAppVersion": "0.2.0",
    "defaultEnabled": false,
    "demoEnabled": true,
    "permissions": ["data:read", "data:write"]
  },
  "peerDependencies": { "@specdriven/plugin": "^1", "react": "^19" }
}
```

- `api` is the kernel contract version; the app refuses a plugin whose
  major it does not speak — the versioned interface `modularity.md` asks
  for. `permissions` is declarative in v1 (plugins are trusted and run in
  process) and is shown before an install. `src` present means the host
  rebuilds the plugin on load and on change wherever it can build.
  `@specdriven/plugin` is a types-only package so authors type against
  `Services` and `Events`.
- Built-in plugins have the same shape under
  `src/plugins/builtin/<id>/{host.ts,client.tsx}` and are imported
  statically, so they ride in the compiled binary and the desktop bundle.
- **One React.** `index.tsx` publishes `window.__specdriven = { React,
  ReactDOM, ReactDOMClient, jsxRuntime, lucide, client, version }` before
  the plugin boundary mounts. Plugin client halves are built with
  `Bun.build` and a resolver plugin that points `react`, `react-dom`,
  `react/jsx-runtime`, `lucide-react` and `@specdriven/client` at shim
  modules re-exporting from that global. The output has no bare
  specifiers, so a plain `import(url)` works in the web build and in
  Electron's sandboxed renderer without an import map. The design tokens
  in `index.css` are global CSS custom properties and apply unchanged. A
  bundle carrying its own React is refused at load.
- **Serving.** `GET /plugins/<id>/client.js?rev=<sha256>` is registered on
  the route registry by the loader; the id is checked against the installed
  set, never joined into a path from the request. A matching `rev` is
  served immutable with an ETag, anything else `no-cache`, an unknown id
  404. The desktop app needs nothing extra: its `app://` handler already
  runs `handleApi` before static files and the shell. `GET /api/state`
  gains `plugins: [{ id, rev, enabled, client, caps, error? }]`, and the
  browser imports each enabled bundle in that order. Host halves are
  imported with `?rev=` so a rebuild defeats the module cache.
- **Building.** A local TypeScript plugin is built by the app itself with
  `Bun.build` at load time when the server runs on Bun — the dev server and
  the compiled binary. The Electron main process runs on node and cannot
  build, so it loads `dist/` only and shows "build this plugin with
  `specdriven plugin build`" when it is missing. Marketplace plugins ship
  `dist/`.

## Discovery, profile, installation

Sources, in load order:

1. built-in — in the tree, always present, cannot be uninstalled, can be
   disabled
2. `~/.specdriven/plugins/<id>/` — installed from a marketplace, or linked
   by hand
3. `<data repo>/.specdriven/plugins/<id>/` — plugins that travel with a
   team's repo; a trust prompt on first load, since the repo is pulled
   from GitHub
4. explicit paths in settings — the vibe-coded plugin under development

The profile lives in the device-local `.specdriven/settings.json`, beside
today's switches, so the promise that a module switch does not follow the
user to another device still holds:

```json
{
  "plugins": {
    "enabled": ["notebook", "tests", "@acme/jira-tab"],
    "sources": [{ "path": "/Users/me/dev/jira-tab" }],
    "marketplaces": [{ "name": "specdriven", "source": "https://github.com/SpecDriven/plugins" }]
  }
}
```

`enabled` is ordered and is the tab order. The existing `roadmap`,
`tasks`, `notebook`, `models`, `testing` and `issues` keys migrate into it
on first read and stay as aliases for one release. A plugin is *installed*
when a source provides it and *on* when it is listed; Settings → Plugins
shows both. Demo mode enables every built-in, replacing the hand-written
`DEMO_SETTINGS` module flags. An unknown id is reported, not fatal.

A marketplace is a git repository with `marketplace.json` at its root,
cached under `~/.specdriven/marketplaces/<name>/` through the app's own git
engine so it works with `gh`, the app's sign-in or the bundled JavaScript
git alike:

```json
{
  "name": "specdriven",
  "owner": { "name": "SpecDriven", "url": "https://github.com/SpecDriven" },
  "plugins": [
    { "name": "jira-tab", "source": "https://github.com/SpecDriven/plugin-jira-tab", "ref": "v0.3.1",
      "description": "Jira issues as a tab", "version": "0.3.1", "category": "tabs", "api": "1" },
    { "name": "linear-tab", "source": "./plugins/linear-tab", "description": "Linear issues as a tab" }
  ]
}
```

Sources in v1 are a git url with a `ref`, or a subdirectory of the
marketplace clone; an npm source is a later addition. Install clones (or
copies) pinned to the ref into `~/.specdriven/plugins/<name>/`, writes an
install record (marketplace, version, commit, date), reads the manifest,
checks `api` and `minAppVersion`, shows the permissions, and only after the
user confirms appends the id to `enabled`. Update fetches the new ref, one
install at a time per id; Remove disables, disposes and deletes the
folder. Symlinks escaping the plugin folder are refused. Routes:
`GET /api/plugins`, `GET|POST /api/plugins/marketplaces`,
`POST /api/plugins/install`, `POST /api/plugins/:id/enable|disable|update`,
`DELETE /api/plugins/:id`, none of them in demo. Enable and disable are
also just `PUT /api/state` with a changed list: the loader listens to
`state/changed` and diffs the mounted scopes.

## Developer loop

`specdriven plugin new <id>` scaffolds both halves; `plugin build` runs
`Bun.build` for both; `plugin link <path>` appends a source. With a path
source and the server on Bun, a file watcher rebuilds and the kernel
disposes and re-mounts that one plugin; the browser re-imports the new
`rev`. That is the reversible-effects payoff.

## The Tests tab as a plugin

```ts
// src/plugins/builtin/tests/host.ts
export default {
  name: "tests",
  inject: ["routes", "docs", "shell", "sync"],
  apply(ctx) {
    ctx.docs.register("testing", TestingDocSchema);
    ctx.shell.add("/tests");
    ctx.routes.add("GET", "/api/tests", getTesting);
    ctx.routes.add("POST", "/api/tests/cases", postTestCase);
    // … the other handlers, unchanged
    ctx.sync.drift((dir) => importTestingDrift(dir));
  },
} satisfies Plugin;

// src/plugins/builtin/tests/client.tsx
export default {
  name: "tests",
  inject: ["tabs", "shortcuts", "palette", "router"],
  apply(ctx) {
    ctx.router.add(testsRoute);                       // /tests, /tests/cases/:id, …
    ctx.tabs.add({ id: "tests", title: "Tests", icon: FlaskConical, order: 60,
      Sidebar: TestsSidebar, Pane: TestsPane });
    ctx.shortcuts.add({ id: "go-tests", keys: "g e", run: () => ctx.tabs.open("tests") });
    ctx.palette.add("Tests", () => searchTesting(testing.snapshot()));
    ctx.effect(() => () => disposeTestingStore());
  },
} satisfies Plugin;
```

No `withTesting` wrapper any more: the routes exist only while the plugin
is mounted, so "the server refuses the module's requests while the switch
is off" ([tests](../specs/app/tests.feature.md)) falls out of the kernel.

Boot, for the record:

```
host:   new Context() → plugin(core: routes/settings/sync/docs/shell) → plugin(loader)
        → discover sources → for id in plugins.enabled: validate manifest → ctx.plugin(hostHalf, config)
        → loader adds GET /plugins/:id/client.js → start()
client: new Context() → plugin(clientCore: tabs/panes/shortcuts/palette/router/settingsUI)
        → <PluginBoundary> reads /api/state.plugins → for each enabled: import(url) → ctx.plugin(mod.default, caps)
        → App renders the rail, the pane, the shortcuts and the palette from the registries
```

## Phases

**Phase 1 — kernel and built-in tab plugins, in the tree only.** Create
`src/plugins/kernel.ts`, the host services over the existing `api.ts`,
`reload.ts` and `state.ts` helpers, the client services and the
`<PluginBoundary>`, and `src/plugins/builtin/{tests,issues,models,pages}`.
Migration order: **Tests** (own store, own document, clearest gate), then
**Issues** (its tab depends on a GitHub repo, which exercises `when()`),
then **Models** (already lazy and in its own directory), then Roadmap,
Tasks and Notebook as three instances of one `pages` plugin parameterized
by the `PageFolder` descriptor in `pages.ts`, with the notebook's extras.
**Specs stays core.** One tab per pull request; the old branch stays alive
behind the registry until the last tab moves. Risk: the `App.tsx` state
that cuts across modules — the open page shared by pages and features, the
reload key, "go home when the tab closes".

**Phase 2 — local filesystem plugins.** The loader for the three
filesystem sources, manifest validation, `Bun.build` on load, the bundle
route with revisions, the shared-React shims, `specdriven plugin
new|build|link`, watch-and-remount in dev, and a Settings → Plugins list
with enable, disable and errors. The desktop build is unchanged; plugins
live outside the app bundle. Risk: a plugin bundling its own React, caught
by the shims and the load-time check.

**Phase 3 — marketplace.** The marketplace cache through the git engine,
`marketplace.json` parsing, the install, update and remove routes, a
browse dialog (category, version, api check, consent step),
`plugins.marketplaces` in settings and a default `SpecDriven/plugins`
index. Risk: supply chain — pinned refs, explicit consent, a visible
"third-party, runs with the app's rights" note, and no automatic updates.

## Testing

With `bun test`: the kernel (effects unwind last-in-first-out and children
first; a removed service re-pends its dependents and re-applies them on
return; `serial` keeps order, `bail` stops at the first answer, `emit`
isolates errors; dispose is idempotent; a throwing `apply` fails only its
scope), the route registry (the existing `matchApi` cases ported, a
disposed plugin's route answers 404, core routes match first), settings
(a field appears on `GET /api/state`, `PUT` validates, demo wins in demo,
the old flags migrate, a flip makes a route appear and disappear), sync
(`sync/drift` runs after the specs pass on a temporary git repo), a fixture
plugin copied to a temporary folder — discovered, built, served immutable
on a matching `rev` and `no-cache` otherwise, rebuilt with a new `rev`
after an edit, disposed when the folder goes — and a marketplace index
installed into a temporary home against a fake git remote. In the browser:
a tab appears and disappears without a reload, and a `/tests` deep link
with the plugin off goes home. On the desktop: the crash test after
Phase 1.

## References

- DeepSeek Harness: <https://github.com/deepseek-ai/deepseek-harness> —
  `docs/architecture.md`, `docs/cordis-primer.md`,
  `docs/subsystems/client-modules.md` (the `dsh.client` declaration, boot
  graph and bundle route) and `docs/user/develop/basic/publish.md`
  (`dsh plugin --profile <p> add <path | git | npm>`)
- Cordis: <https://github.com/cordiverse/cordis>
- Overviews: <https://www.datacamp.com/blog/what-is-deepseek-harness>,
  <https://thenewstack.io/deepseek-harness-open-source-plugins/>
