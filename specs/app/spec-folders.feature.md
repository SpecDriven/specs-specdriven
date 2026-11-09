# Spec folders

**Created by:** vydra

**Updated by:** vydra

Support opening and switching between multiple specifications folders. (Currently called data directory)

<img src="/api/images/image-c9aa1fcb.png" alt="image" width="516">

## S1: Manage specification folders @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Open new folders from the local file system ("Open another folder…"). Manage folders — in the
recent folders menu and the command palette — is the only way to change the open folder; there
is no separate "Change data directory" button.

Ability to delete specification folders.

A folder cloned from a GitHub url (with a personal access token or otherwise) is listed by its url,
with the directory it is cloned to on this computer shown beneath it.

Manage folders lists every folder the app knows on one list: the specification folders and,
where the app can open them, the code repositories (code-repo-mode.feature.md), most recent first,
the open one first and badged. Each entry carries an icon for its kind — a book for a
specification folder, a code folder for a code repository — named in its tooltip. The same path
opened both ways is two entries, one per kind. The recent folders menu beside the folder name
lists the 10 newest entries of that list, with the same icons, the open one ticked.

[test: lists every folder by name with its path, the open one badged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L83 ]
[test: every folder but the open one can be opened or removed from the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L94 ]
[test: a GitHub url shows the folder it is cloned to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L106 ]
[test: offers to open another folder from the file system, and to close : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L124 ]
[test: spec folders and code repositories share one list, in the given order, each marked by its kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L163 ]
[test: the same path as a spec folder and as a code repository is two entries, only the open kind badged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L185 ]
[test: Open and Remove hand back the entry with its kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L210 ]
[test: spec folders and code repositories make one list, the open one first, then newest first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L151 ]
[test: the stored list is cleaned, each path once per kind, at most 10 of each kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L185 ]
[test: an older file's two lists and open times become the one list, in the order it showed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L206 ]
[test: forgetting a folder takes only that kind's entry off the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L230 ]
[test: GET /api/state lists spec folders and code repositories together, newest first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoMode.test.ts#L330 ]
[test: keeps well-formed entries in order and drops the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L252 ]
[test: GET /api/state says where each GitHub url on the list is cloned : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L175 ]
[test: PUT /api/state forgetDataRepo removes a folder from the list and persists it, the folder itself untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L262 ]
[test: the open folder cannot be removed from the list — switch away first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L295 ]

## S2: One entry per repository @v1 [proposed]

A spec data repo is on the folder list once. Its GitHub url and a folder on this computer for the
same repository — the url's own clone opened by its path, or another checkout whose origin is that
url — are one entry: the one opened last stays, the other leaves the list. Removing the entry takes
every name of that repository off the list.

The coding agent pane offers the data repo's folder once too: for a GitHub url, the folder it is
cloned to.

[test: a url and its own clone opened by path are one entry, the open one kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L397 ]
[test: a checkout whose origin is the url is the same repository, in any remote form : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L411 ]
[test: other repositories stay apart, even urls that share a clone folder name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L425 ]
[test: picking the clone folder by path replaces the url on the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L433 ]
[test: removing a repository from the list takes its other names with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L446 ]

## S3: Switch spec folder @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

When switching spec folders, the app will bring up a modal dialog indicating the progress of loading the new dataset.
All other operations are blocked.

A path that names no folder is not opened silently: the app says there is no folder there and asks
whether to create it as a new, empty spec folder. Declining keeps the current folder open.

[test: nothing on it can be pressed — no buttons, and it says the rest waits : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L57 ]
[test: a missing folder is refused with a 404, nothing made, the open folder kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L335 ]
[test: createDataRepo makes the missing folder and opens it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L350 ]
[test: a path to a file is not a folder to open, even with createDataRepo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L360 ]
[test: a modal dialog names the folder being opened and the step it is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L23 ]
[test: the steps read done → current → pending as the switch advances : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L35 ]
