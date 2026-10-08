# Spec folders

Support opening and switching between multiple specifications folders. (Currently called data directory)

<img src="/api/images/image-c9aa1fcb.png" alt="image" width="516">

## Manage specification folders @v1 [proposed]

Open new folders from the local file system. "Change data directory"

Ability to delete specification folders.

[test: lists every folder by name with its path, the open one badged : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L72 ]
[test: every folder but the open one can be opened or removed from the list : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L83 ]
[test: offers to open another folder from the file system, and to close : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L95 ]
[test: PUT /api/state forgetDataRepo removes a folder from the list and persists it, the folder itself untouched : https://github.com/SpecDriven/specdriven_app/blob/main/tests/recentFolders.test.ts#L220 ]
[test: the open folder cannot be removed from the list — switch away first : https://github.com/SpecDriven/specdriven_app/blob/main/tests/recentFolders.test.ts#L253 ]

## Switch spec folder @v1 [proposed]

When switching spec folders, the app will bring up a modal dialog indicating the progress of loading the new dataset.
All other operations are blocked.

A path that names no folder is not opened silently: the app says there is no folder there and asks
whether to create it as a new, empty spec folder. Declining keeps the current folder open.

[test: nothing on it can be pressed: no buttons, and it says the rest waits : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L48 ]

[test: a missing folder is refused with a 404, nothing made, the open folder kept : https://github.com/SpecDriven/specdriven_app/blob/main/tests/recentFolders.test.ts#L293 ]
[test: createDataRepo makes the missing folder and opens it : https://github.com/SpecDriven/specdriven_app/blob/main/tests/recentFolders.test.ts#L308 ]
[test: a path to a file is not a folder to open, even with createDataRepo : https://github.com/SpecDriven/specdriven_app/blob/main/tests/recentFolders.test.ts#L318 ]
[test: a modal dialog names the folder being opened and the step it is on : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L14 ]
[test: the steps read done → current → pending as the switch advances : https://github.com/SpecDriven/specdriven_app/blob/main/tests/specFolders.test.tsx#L26 ]
