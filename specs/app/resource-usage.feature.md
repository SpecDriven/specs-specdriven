# Resource usage

SpecDriven, the agents it starts and the folders it fills can take a lot of a computer: an agent CLI holds hundreds of megabytes while it works, agent worktrees with their dependencies pile up, the installer keeps the previous version to go back to, and the desktop window's browser cache grows by itself. The Resources tab shows how much memory and disk the app uses, and frees the space that is safe to free.

- A third tab on the Observability page, beside Logs and Verbosity. Like Observability, it is not in the demo or the hosted app.
- Memory is the resident memory of the app's own process (the desktop app's main process, or the web server) and every process under it, read from the system's process list — `ps` on macOS and Linux, Win32_Process on Windows. In the desktop app, Electron's own report names its processes (a window, the GPU process, the network service) and gives their figure.
- Processes are grouped by what the app started: SpecDriven itself with its windows and helpers; one row per agent CLI (Claude Code, Codex, OpenCode, Pi) with everything that agent started counted in it; MCP servers; anything else (git, a terminal). A row with several processes opens to list them with their pids.
- A total with the computer's memory beside it, a bar split by group, and the figures measured again every 3 seconds while the tab shows and the window is in view. The figures are approximate: memory that processes share is counted in each of them.
- Disk is every folder the app keeps things in: the installed app versions (the running one, and the one kept to go back to); in the desktop app, its browser cache and its other app data; the open folder, split into its files, its `.git` history and (a spec folder) its Automerge edit history; agent worktrees, agent session records and the Claude Code binaries; plugins, plugin builds, backups and marketplaces; the logs; and whatever else is in `~/.specdriven`.
- Folders are measured by what they take on disk, links are never followed and a hard-linked file counts once. Measuring a large folder takes seconds, so disk is measured when the tab opens and again on Refresh, not on a timer. The free space on each disk involved shows beside the total.
- A row that can be cleaned up has a button. Every cleanup asks first, naming what goes, what cannot be undone and about how much it frees, then measures the disk again and says how much was freed.

## S1: See how much memory SpecDriven and its agents use @v1 [proposed]

- Given: the app running with an agent session open
- When: the user opens the Observability page's Resources tab
- Then: the Memory section shows the total, the computer's memory, a bar by group, and the groups — SpecDriven with its windows and helpers, the agent with everything it started, MCP servers, the rest — each with its size
- and the figures change as the agent works, every 3 seconds, while the tab is shown
- Otherwise: the process list cannot be read, and only the app's own processes are counted, with a line saying why

[test: ps output, an executable path with spaces, and the command line beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L34 ]
[test: PowerShell's Win32_Process JSON, a list, one object, or junk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L54 ]
[test: the tree is the root and everything under it, nothing beside it; a loop ends : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L61 ]
[test: the app and its helpers, one branch per agent with its children, MCP servers, and the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L82 ]
[test: Electron's report names its processes and its figure wins over ps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L94 ]
[test: the web server is the Server, without a desktop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L107 ]
[test: this process's tree from the reader, with the machine's memory : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L119 ]
[test: an unreadable process table still counts this process, and says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L128 ]
[test: memory groups with the total, measured again while shown and not while the window is hidden : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L232 ]
[test: a tiny group still shows in the bar; nothing at all is empty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L204 ]
[test: sizes read as people read them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L196 ]
[test: lists the entries, searches through the server and saves verbosity : https://github.com/SpecDriven/specdriven-app/blob/main/tests/observabilityPage.test.tsx#L194 ]
[test: resource routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L259 ]
[test: a refused request shows the server's reason : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L308 ]

## S2: See how much disk each place takes and the free space @v1 [proposed]

- Given: the app installed with two versions, a spec folder open, agent worktrees and logs
- When: the user opens the Resources tab
- Then: the Disk section shows the total, the free space on the disk, and each place grouped — App, the open folder, Agents, Plugins, Logs, Other — with its size and its path
- and the running version is marked "running", the previous one "kept to go back to", and a worktree "in use" or "uncommitted changes"
- and Refresh measures again

[test: files add up; a link is not followed, and a hard-linked file counts once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L160 ]
[test: an unreadable folder counts as itself and does not stop the walk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L177 ]
[test: every place, with the cleanups each offers and the free space : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L258 ]
[test: the desktop app's data folder, the browser cache apart from the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L301 ]
[test: a spec folder, files, git history and the Automerge store, each counted once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L318 ]
[test: disk groups with free space, tags, and a cleanup only after confirming : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L266 ]

## S3: Remove an agent worktree @v1 [proposed]

- Given: agent worktrees on the Resources tab, none of them used by a running session
- When: the user clicks Remove on one and confirms
- Then: the worktree folder is removed with `git worktree remove`, while its branch and every commit on it stay in the repository, and the disk is measured again
- and a worktree with uncommitted changes says in the confirmation that they are lost, and goes only after that confirmation
- Otherwise: a running agent session works in the worktree, and its Remove button is off and says so; the server refuses it too

[test: a clean worktree goes; its branch stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L348 ]
[test: uncommitted changes are lost only when the request says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L357 ]
[test: a worktree a session works in, and anything that is not a worktree, are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L365 ]
[test: the last worktree of a repo takes its folder with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L375 ]
[test: the confirmation names what goes and how much it frees : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L209 ]
[test: a request is checked before anything runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L419 ]

## S4: Clear the browser cache, old logs and the previous version @v1 [proposed]

- Given: the Resources tab in the desktop app
- When: the user clicks Clear on the browser cache, Delete old logs on the logs, or Delete on the version kept to go back to, and confirms
- Then: the browser cache is emptied through Electron; the rotated log files are deleted while the one being written stays; the previous version's folder and its source zip are deleted
- and each says how much it freed
- Otherwise: the version is the running one, and it is never offered or deleted

[test: old logs go; the one being written stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L384 ]
[test: the kept version goes with its zip; the running one is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L391 ]
[test: the browser cache is cleared through the desktop app, and only there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L399 ]
