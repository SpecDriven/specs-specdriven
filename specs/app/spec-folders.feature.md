# Spec folders

Support opening and switching between multiple specifications folders. (Currently called data directory)

<img src="/api/images/image-c9aa1fcb.png" alt="image" width="516">

## S1: Manage specification folders @v1 [proposed]

Open new folders from the local file system. "Change data directory"

Ability to delete specification folders.

A folder cloned from a GitHub url (with a personal access token or otherwise) is listed by its url,
with the directory it is cloned to on this computer shown beneath it.

[test: lists every folder by name with its path, the open one badged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L72 ]
[test: every folder but the open one can be opened or removed from the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L83 ]
[test: a GitHub url shows the folder it is cloned to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L95 ]
[test: offers to open another folder from the file system, and to close : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L113 ]
[test: GET /api/state says where each GitHub url on the list is cloned : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L168 ]
[test: PUT /api/state forgetDataRepo removes a folder from the list and persists it, the folder itself untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L255 ]
[test: the open folder cannot be removed from the list — switch away first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L288 ]

## S2: One entry per repository @v1 [proposed]

A spec data repo is on the folder list once. Its GitHub url and a folder on this computer for the
same repository — the url's own clone opened by its path, or another checkout whose origin is that
url — are one entry: the one opened last stays, the other leaves the list. Removing the entry takes
every name of that repository off the list.

The coding agent pane offers the data repo's folder once too: for a GitHub url, the folder it is
cloned to.

[test: a url and its own clone opened by path are one entry, the open one kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L390 ]
[test: a checkout whose origin is the url is the same repository, in any remote form : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L404 ]
[test: other repositories stay apart, even urls that share a clone folder name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L418 ]
[test: picking the clone folder by path replaces the url on the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L426 ]
[test: removing a repository from the list takes its other names with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L439 ]

## S3: Switch spec folder @v1 [proposed]

When switching spec folders, the app will bring up a modal dialog indicating the progress of loading the new dataset.
All other operations are blocked.

A path that names no folder is not opened silently: the app says there is no folder there and asks
whether to create it as a new, empty spec folder. Declining keeps the current folder open.

[test: nothing on it can be pressed — no buttons, and it says the rest waits : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L48 ]
[test: a missing folder is refused with a 404, nothing made, the open folder kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L328 ]
[test: createDataRepo makes the missing folder and opens it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L343 ]
[test: a path to a file is not a folder to open, even with createDataRepo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L353 ]
[test: a modal dialog names the folder being opened and the step it is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L14 ]
[test: the steps read done → current → pending as the switch advances : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specFolders.test.tsx#L26 ]
