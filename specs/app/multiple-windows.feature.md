# Multiple windows

The desktop app can have several folders open at once, each in a window of
its own — two specifications folders side by side, or a specifications
folder beside a code repository. The web version shows one folder, as
before: several browser tabs there all show the same one.

## Windows

- A window shows one folder: a specifications folder, or a code repository
  opened directly (code-repo-mode.feature.md). A folder is open in at most
  one window; opening a folder that a window already shows brings that
  window to the front instead.
- Everything a window reads and writes is its own folder's: the feature
  list, the pages, search, the project settings, the coding agent sessions
  it lists. Switching folders in one window leaves every other window on
  its folder.
- The settings are shared by every window (desktop-app.feature.md "Data
  locations"): the GitHub sign-in, the tokens, the list of recent folders
  and every preference. The settings file names the folder of the window
  that changed it last, which is the folder the web version opens.
- The preferences a window keeps for itself in the browser — the theme, the
  sidebar's width, Vim mode — are the same in every window, and a change in
  one shows in the others: every window loads the app from the same
  address.
- Because the address is the same, the app marks each window's requests
  with the window they came from, and answers them for that window's
  folder. While several windows are open a request that names none of them
  is refused rather than answered for a folder its page may not be showing.
- Each window is named after its folder: its title is the folder's name,
  then the page open in it ("specs-insurance — Checkout"), so the Dock
  menu, the Window menu and the task switcher tell the windows apart. The
  scenario open in the page is left out ("specs-insurance — Checkout", not
  "specs-insurance — Checkout · pays by card v2"). A
  title longer than 36 characters is cut there and ends in "…". A
  window with no folder open is titled "SpecDriven". The page's own title —
  what a browser tab and a link preview show — is unchanged.
- All windows belong to one running app: launching it a second time focuses
  the window used last instead of starting another app.
- A development run (`bun run dev:electron`) has the same windows. Their
  pages come from the dev server, so a frontend edit hot-reloads in every
  window; their requests are answered by the desktop app itself, each for
  its window's folder, as in the packaged app. An edit to the server or to
  the shell shows after the next launch.

## Opening a window

- **File → New Window** opens a window with no folder in it, which asks for
  one as the first run does (app-start.feature.md).
- **File → Open Folder…** asks for a folder and opens it in a new window.
- **File → Open Recent** lists the recent folders
  (spec-folders.feature.md); picking one opens it in a new window.
- In the folder dialog, each folder that is not the open one has an **Open
  in new window** action beside the plain Open, which switches the current
  window as before. The recent-folders menu has **New window**, which opens
  a window that asks for a folder. Both are offered in the desktop app
  only.
- The folder dialog marks every folder another window shows **other
  window**. Such a folder has **Show window**, which brings that window to
  the front, in place of both ways of opening it, and it stays on the list
  until that window is closed. The dialog reads the list again each time it
  opens, and each time its window comes back to the front while it is up.
- A window opened for a folder opens it the way a folder picked by hand is
  opened: asked about when the folder is missing or its kind is not clear,
  pulled, and put at the front of the recent list. Meanwhile it shows the
  app loading, not the first-run screen, which comes up only when the folder
  could not be opened.
- A `specdriven://` link (navigation.feature.md "links from outside the
  app") opens in the window that shows the link's folder. When no window
  shows it and the folder is on the recent list, it opens in a new window.
  A link that names no folder this computer knows opens in the window used
  last. The same holds for a link the app is started with.
- A folder being removed from the recent list must not be open in any
  window.

## Closing and restarting

- Each window remembers its own size and position, by folder: a folder
  opened again comes back where its window was left
  (`window-state.json` in the app-data directory, which also lists the
  windows to bring back).
- Quitting the app and starting it again brings back every window that was
  open, each with its folder and its place on screen. A window closed
  before the quit does not come back, and neither does a window that never
  opened a folder.
- A folder picked in the web version while the desktop app was closed
  opens in the window used last (desktop-app.feature.md "desktop and web
  share settings").
- Closing a window while others stay open ends the coding agent sessions
  started in its folder, as switching folders does (agent-pane.feature.md
  "Switching folders closes the other folder's sessions"). When one of them
  is still working, the app asks first; cancelling keeps the window open.
- Closing the last window on macOS leaves the app running, as before, with
  its folder's sessions; clicking its Dock icon opens the window that was
  closed last.
- Signing out (user-info-settings.feature.md) signs every window out: what
  was saved in each open folder is pushed first, the window it was done in
  shows the sign-in screen and the others close.

## Background work

- Sync runs for every open specifications folder: each is pulled when its
  window opens and on every reload tick, and its saved changes are
  committed and pushed to its own repository. A folder a window leaves —
  it switched, or closed — is synced once more. A code repository is never
  synced (code-repo-mode.feature.md).
- A sync server named in Settings (automerge-store.feature.md) is connected
  once per open specifications folder. A peer's edit refreshes the windows
  of the folder it belongs to and no other.
- Each folder keeps its own plugins (plugins.feature.md): a plugin switched
  on for one open folder shows its tabs in that folder's window only, and
  only that folder gets the files it creates. A plugin stays loaded while
  any open folder uses it. A coding agent a plugin provides is offered in
  the windows of the folders that have the plugin on.
- The folder picker opened from one window belongs to that window: another
  window can open its own at the same time, and each gets its own answer.

## S1: a window is answered for its own folder @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** each lists its features
- **Then** the first shows A's features and the second B's, whichever asked last

[test: each window is answered for its own folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L89 ]
[test: a code repository open in one window is that window's alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L169 ]
[test: work that waited for a folder's turn resumes as its own window's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L197 ]
[test: a window's folder is a specifications folder, or a code repository open in its place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L253 ]

## S2: the web version shows the one folder the settings name @v1 [proposed]

- **Given** the settings file names folder A
- **When** a page with no window behind it — the web version — lists its features
- **Then** it shows A's features, as before

[test: a run without windows reads the settings file as it is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L108 ]
[test: detached work reads the settings file, not the window's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L242 ]
[test: with no window registered the start syncs the folder the settings file names : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L147 ]
[test: where there is no shell to open a window, the action is not offered and the route is not there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L392 ]

## S3: switching folders in one window leaves the others alone @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** the second window switches to folder C
- **Then** the first window still shows folder A
- **And** the settings file names folder C, the one changed last

[test: switching folders in one window leaves the other window's folder alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L128 ]
[test: a window that opens a folder is found by its id, and is done with the folder it was opened for : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L272 ]

## S4: a preference saved in one window does not move another window's folder @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** the second window changes a setting
- **Then** the first window reads the new setting and still shows folder A

[test: a window saving a plain setting does not move another window's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L151 ]

## S5: a request is answered for the window it came from @v1 [proposed]

- **Given** two windows, which load the app from the same address
- **When** a request arrives that names one of them, by the header the app sets or by its window's User-Agent
- **Then** it is answered for that window's folder
- **And** with several windows open, a request that names none of them is refused

[test: a request is told apart by the header the shell sets, or by its window's User-Agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L140 ]
[test: with several windows a request that names none of them is nobody's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L150 ]

## S6: a new window asks for a folder @v1 [proposed]

- **Given** the app is open on folder A
- **When** the user chooses File → New Window
- **Then** a second window opens on the first-run screen, with no folder open
- **And** the first window still shows folder A

[test: File holds New Window, Open Folder… and the recent folders, each opening a window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appMenu.test.ts#L45 ]
[test: with no recent folders the submenu says so, and File ends in Quit off macOS : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appMenu.test.ts#L59 ]
[test: without them the standard File menu stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/appMenu.test.ts#L80 ]
[test: a window with no folder open is asked for one, whatever the settings name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L118 ]

## S7: a folder opens in a window of its own from the folder dialog @v1 [proposed]

- **Given** the folder dialog lists folders A, B and C, and A is open
- **When** the user picks "Open in new window" for folder B
- **Then** a second window opens folder B, as a folder picked by hand is opened
- **And** the second window never shows the first-run screen on the way: it
  shows the app loading, then folder B — the first-run screen comes up only
  when folder B could not be opened, with the reason
- **And** the first window still shows folder A

[test: every folder but the open one can be opened in a window of its own, where there are windows to open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L260 ]
[test: outside the desktop app the action is not offered : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L272 ]
[test: the action hands back the entry with its kind, and switches nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L277 ]
[test: a window opening the folder it was opened for does not show the first-run screen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startFolder.test.tsx#L226 ]
[test: Open in new window asks the shell for a window on the folder, or for one that asks for a folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L373 ]
[test: a window opened for a folder tells its page once, as it boots, and opens it like a folder picked by hand : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L398 ]

## S8: opening a folder that is already open focuses its window @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** the user opens folder A from the second window
- **Then** the first window comes to the front
- **And** the second window still shows folder B

[test: picking a folder another window shows brings that window forward and switches nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L416 ]
[test: a GitHub url and the folder it is cloned into are the same folder to the windows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L498 ]
[test: bringing a window forward and closing the others are the shell's to do, and nothing without one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L525 ]

## S9: a folder open in a window cannot be removed from the list @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** the user removes folder B from the recent list in the first window
- **Then** the app refuses, saying the folder is open in another window

[test: a folder open in a window cannot be taken off the recent list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L471 ]

## S10: every open folder is synced @v1 [proposed]

- **Given** two windows on two specifications folders with saved changes in each
- **When** the reload tick runs
- **Then** each folder's changes are committed to its own repository
- **And** a window showing a code repository is not synced

[test: the app's start syncs the folder of every window, and no code repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L135 ]
[test: a window opened later has its folder synced as it opens : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L153 ]
[test: the reload tick syncs every open folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L165 ]
[test: a folder a window leaves is synced once more : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L179 ]
[test: a timer loop started by a window's request works for no window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L227 ]

## S11: a sync server is connected once per open folder @v1 [proposed]

- **Given** a sync server is named in Settings
- **And** two windows on two specifications folders
- **When** one window switches to another folder
- **Then** its connection moves to that folder and the other window's stays

[test: a sync server named in Settings is connected once per open specifications folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L265 ]

## S12: a plugin shows only in the window of the folder that enabled it @v1 [proposed]

- **Given** folder A has the Roadmap plugin switched on and folder B does not
- **And** both are open
- **When** the reload tick runs and each window lists its tabs
- **Then** only A's window has the Roadmap tab
- **And** no `roadmap` folder is created in folder B

[test: a plugin switched on for one open folder shows in that folder's window alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L295 ]
[test: a plugin's files are created in the folder that has it on, and in no other : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L311 ]
[test: a plugin stays loaded while any open folder uses it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L324 ]
[test: a window that closes takes the plugins only its folder used with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L340 ]
[test: a plugin's routes and sync hooks are its own, and answer only where it is let through : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L551 ]
[test: each window is let through to its own folder's plugins, and a tick to its folder's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L575 ]

## S13: each window has its own folder picker @v1 [proposed]

- **Given** two windows
- **When** each opens the folder picker
- **Then** two pickers open, and each window gets its own answer

[test: each window has its own folder picker, and its own answer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L246 ]

## S14: a window lists its own folder's agent sessions @v1 [proposed]

- **Given** two windows, and a coding agent session started in each folder
- **When** each window lists its sessions
- **Then** each shows only the session of its own folder

[test: a window lists its own folder's sessions, not another window's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L190 ]
[test: switching folders in one window ends its old folder's sessions and no other window's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L203 ]

## S15: closing a window ends its folder's agent sessions @v1 [proposed]

- **Given** two windows, and a coding agent session started in each folder
- **When** the user closes the second window
- **Then** the session of its folder ends
- **And** the first window's session goes on

[test: closing a window while another stays ends its folder's sessions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L214 ]
[test: closing the last window leaves its sessions running : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L237 ]
[test: a closed window drops out of the open set; its record keeps its folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L185 ]

## S16: closing a window with a working session asks first @v1 [proposed]

- **Given** two windows, and an agent session in the second that is still working
- **When** the user closes the second window and cancels
- **Then** the window stays open and the session goes on

[test: a window whose folder has a session at work is one to ask about before it closes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L225 ]

## S17: all windows come back after a restart @v1 [proposed]

- **Given** two windows open on folders A and B, each moved and resized
- **When** the user quits the app and starts it again
- **Then** both windows open, each with its folder at its size and position

[test: every window open at quit comes back with its folder and its place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L33 ]
[test: with nothing saved the app starts with one window on the folder the settings name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L48 ]
[test: a file from before there were several windows is that one window, where it was : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L54 ]
[test: a folder saved twice comes back as one window, and a window with no folder does not come back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L61 ]
[test: one of the saved windows is always the one in front : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L109 ]
[test: the session round-trips through its file; a corrupt file is no session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L125 ]
[test: the first run is a first run; every restart must render two windows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L217 ]
[test: the profile is left as an app with two folders open would leave it, the first run's geometry kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/crashTestDesktop.test.ts#L224 ]

## S18: a window closed before the quit stays closed @v1 [proposed]

- **Given** two windows open on folders A and B
- **When** the user closes B's window, quits the app and starts it again
- **Then** one window opens, on folder A
- **And** opening folder B again puts its window where it was left

[test: a window closed before the quit stays closed, though the settings file still names its folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L83 ]
[test: a folder opened again comes back where its window was left : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L93 ]
[test: closing the last window keeps it as the one that comes back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L101 ]
[test: the places remembered are capped, the oldest going first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L115 ]

## S19: a folder picked in the web version opens in the desktop app @v1 [proposed]

- **Given** the desktop app was quit with windows on folders A and B
- **And** the web version, which shares the settings, then switched to folder Z
- **When** the desktop app starts
- **Then** the window used last opens folder Z, and the other window comes back on its folder

[test: a folder picked elsewhere since the last run opens in the window used last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowSession.test.ts#L70 ]
[test: the folder a window last wrote to the settings file is kept for the next start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/workspace.test.ts#L286 ]

## S20: a link opens in the window that shows its folder @v1 [proposed]

- **Given** two windows, one on folder A and one on folder B
- **When** the user follows a `specdriven://` link to a spec in folder B
- **Then** the second window comes to the front on that spec
- **And** the first window is untouched

[test: a link names its folder first, and the window showing that folder takes it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L232 ]
[test: a link that names no folder this machine knows goes to the window used last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L253 ]
[test: a folder named like a page does not take that page's links : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L262 ]
[test: a second launch without a link brings the window used last to the front : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L270 ]
[test: a page never opens a window of the app, even onto its own pages : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L50 ]

## S21: a link to a folder no window shows opens a new window @v1 [proposed]

- **Given** one window on folder A, and folder B on the recent list
- **When** the user follows a `specdriven://` link to a spec in folder B
- **Then** a second window opens on that spec in folder B

[test: a folder opened before that no window shows gets a window of its own, on the linked page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/linkRouting.test.ts#L244 ]

## S22: signing out closes the other windows @v1 [proposed]

- **Given** two windows
- **When** the user signs out in the first
- **Then** what was saved in both folders is pushed first
- **And** the first shows the sign-in screen and the second closes

[test: signing out in one window closes the others : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L481 ]

## S23: the folder dialog marks the folders other windows show @v1 [proposed]

- **Given** three windows, on folders A and B and on code repository C, and
  folder D on the recent list, open in no window
- **When** the user opens the folder dialog in the first window
- **Then** A is badged "open", and B and C are badged "other window"
- **And** B and C each have "Show window", which brings their window to the
  front, and neither Open, "Open in new window" nor a working remove button
- **And** D has Open, "Open in new window" and remove, as before

[test: each window is told which folders on the list the other windows show : https://github.com/SpecDriven/specdriven-app/blob/main/tests/multiWindow.test.ts#L441 ]
[test: a folder another window shows is badged, offers its window, and is neither opened nor removed here : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L328 ]
[test: Show window hands back the entry of the folder whose window comes forward : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L367 ]

## S24: a development run opens several windows @v1 [proposed]

- **Given** `bun run dev` is running, and `bun run dev:electron` beside it
- **When** the user opens folder B in a new window while the first shows
  folder A
- **Then** each window shows its own folder, and both come back after a
  restart of the development run
- **And** an edit to the frontend hot-reloads in both windows
- **And** a page the dev server has no route for still loads the app

[test: a development run loads its windows from the dev server, on the port it listens on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/devFrontend.test.ts#L9 ]
[test: only requests to the dev server's own address pass through the desktop process : https://github.com/SpecDriven/specdriven-app/blob/main/tests/devFrontend.test.ts#L16 ]
[test: a page the dev server has no route for loads the app shell, and a missing file stays missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/devFrontend.test.ts#L31 ]

## S25: each window is named after its folder @v1 [proposed]

- **Given** two windows, one on folder `specs-insurance` with the feature
  Checkout open, and one on the code repository `bookshop`
- **When** the user opens the app's Dock menu or its Window menu
- **Then** the windows are listed as "specs-insurance — Checkout" and
  "bookshop"
- **And** a window that switches to another folder is listed under that
  folder's name from then on
- **And** a window with no folder open is listed as "SpecDriven"
- **And** a window with a scenario open is listed by its feature alone,
  "specs-insurance — Checkout"
- **And** a title longer than 36 characters is cut to 36, ending in "…"

[test: a window is titled with its folder's name, then the page open in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowTitle.test.ts#L9 ]
[test: a window with no folder keeps its page's title, and a title that is not the app's is kept whole : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowTitle.test.ts#L28 ]
[test: a window goes by its code repository, else its specifications folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowTitle.test.ts#L36 ]
[test: a long window title is cut to 36 characters, ending in an ellipsis : https://github.com/SpecDriven/specdriven-app/blob/main/tests/windowTitle.test.ts#L44 ]
