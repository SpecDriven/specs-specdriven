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

[test: ps output, an executable path with spaces, and the command line beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L37 ]
[test: PowerShell's Win32_Process JSON, a list, one object, or junk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L57 ]
[test: the tree is the root and everything under it, nothing beside it; a loop ends : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L64 ]
[test: the app and its helpers, one branch per agent with its children, MCP servers, and the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L85 ]
[test: Electron's report names its processes and its figure wins over ps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L97 ]
[test: the web server is the Server, without a desktop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L110 ]
[test: this process's tree from the reader, with the machine's memory : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L122 ]
[test: an unreadable process table still counts this process, and says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L152 ]
[test: the commands reading the process table are left out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L131 ]
[test: a ps the app started, or the reading command itself, counts as measuring : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L144 ]
[test: memory answers with this process in its groups : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceRoutes.test.ts#L48 ]
[test: memory is measured every 3 seconds by default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L206 ]
[test: memory groups with the total, measured again while shown and not while the window is hidden : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L246 ]
[test: a tiny group still shows in the bar; nothing at all is empty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L218 ]
[test: sizes read as people read them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L210 ]
[test: lists the entries, searches through the server and saves verbosity : https://github.com/SpecDriven/specdriven-app/blob/main/tests/observabilityPage.test.tsx#L194 ]
[test: resource routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L259 ]

## S2: See how much disk each place takes and the free space @v1 [proposed]

- Given: the app installed with two versions, a spec folder open, agent worktrees and logs
- When: the user opens the Resources tab
- Then: the Disk section shows the total, the free space on the disk, and each place grouped — App, the open folder, Agents, Plugins, Logs, Other — with its size and its path
- and the running version is marked "running", the previous one "kept to go back to", and a worktree "in use" or "uncommitted changes"
- and Refresh measures again

[test: files add up; a link is not followed, and a hard-linked file counts once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L184 ]
[test: an unreadable folder counts as itself and does not stop the walk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L201 ]
[test: every place, with the cleanups each offers and the free space : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L282 ]
[test: the desktop app's data folder, the browser cache apart from the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L325 ]
[test: a spec folder, files, git history and the Automerge store, each counted once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L342 ]
[test: disk groups with free space, tags, and a cleanup only after confirming : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L281 ]
[test: disk is measured once, again on Refresh, and again after a cleanup : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L367 ]
[test: the places measured are the SpecDriven home and the settings beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceRoutes.test.ts#L57 ]

## S3: Remove an agent worktree @v1 [proposed]

- Given: agent worktrees on the Resources tab, none of them used by a running session
- When: the user clicks Remove on one and confirms
- Then: the worktree folder is removed with `git worktree remove`, while its branch and every commit on it stay in the repository, and the disk is measured again
- and a worktree with uncommitted changes says in the confirmation that they are lost, and goes only after that confirmation
- Otherwise: a running agent session works in the worktree, and its Remove button is off and says so; the server refuses it too

[test: a clean worktree goes; its branch stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L401 ]
[test: uncommitted changes are lost only when the request says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L410 ]
[test: a worktree a session works in, and anything that is not a worktree, are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L418 ]
[test: the last worktree of a repo takes its folder with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L428 ]
[test: the confirmation names what goes and how much it frees : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L223 ]
[test: a request is checked before anything runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L472 ]
[test: a cleanup request that is not one is refused before anything runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceRoutes.test.ts#L38 ]

## S4: Clear the browser cache, old logs and the previous version @v1 [proposed]

- Given: the Resources tab in the desktop app
- When: the user clicks Clear on the browser cache, Delete old logs on the logs, or Delete on the version kept to go back to, and confirms
- Then: the browser cache is emptied through Electron; the rotated log files are deleted while the one being written stays; the previous version's folder and its source zip are deleted
- and each says how much it freed
- Otherwise: the version is the running one, and it is never offered or deleted

[test: old logs go; the one being written stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L437 ]
[test: the kept version goes with its zip; the running one is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L444 ]
[test: the browser cache is cleared through the desktop app, and only there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceUsage.test.ts#L452 ]

## S5: Resources stay out of the demo and the hosted app @v1 [proposed]

- Given: the public demo, or a hosted worker whose container other people's workers share
- When: the Resources tab, or its routes, are asked for memory, disk or a cleanup
- Then: nothing is measured or deleted, and the answer says Resources are not shown in the demo, or not available in the hosted app
- and the tab shows that reason in place of the figures

[test: the demo and the hosted app refuse every resource route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceRoutes.test.ts#L22 ]
[test: a refused request shows the server's reason : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourcesPanel.test.tsx#L323 ]
