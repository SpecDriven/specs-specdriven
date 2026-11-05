# project-settings

- every data folder's project-settings.json, edited from the user menu

A data repo describes its project in `project-settings.json` at its root:
the people working on it, the code repositories that implement its specs,
and whether the feature files carry authorship. The file is checked in, so
every teammate's app reads the same project.

The app makes the file for every data folder it opens, and **Project
settings** in the user menu edits it in the editor pane, the way Settings
does: every control saves the moment it changes, and the next sync commits
and pushes the file like any saved change.

```json
{
  "members": [{ "name": "Ada Lovelace", "email": "ada@acme.example" }],
  "code-repos": ["../acme-app", "../acme-mobile"],
  "default-code-repo": "../acme-app",
  "authors-in-markdown": true
}
```

A code repository is optional, and there may be several. One is the
default: where a coding session runs, whose GitHub issues the Issues tab
shows, and where test links resolve. The app assumes a code repo is checked
out beside the data repo, so a repo is stored as a path relative to the data
folder — `../acme-app` — which holds for every teammate with the same layout.

## S1: A code repo's specs symlink is created and removed from the editor @v1 [proposed]

- Given: a code repository listed in Project settings whose folder exists
- Then: its row says what sits at `<code repo>/specs`: no symlink, a symlink
  to this data folder's spec tree, a symlink to somewhere else, or a real
  folder
- When: the user clicks **Create symlink** on a repo with none
- Then: `<code repo>/specs` becomes a relative symlink to the data folder's
  `specs` tree — `../acme-specs/specs` for checkouts side by side — so the
  code sees the specs it implements
- When: the user clicks **Remove symlink** on a repo whose `specs` is a symlink
- Then: the symlink is removed, whatever it pointed at
- and a real `specs` folder is never removed or replaced: the row says it is
  a folder and offers neither
- and a folder that is not there offers neither either

[test: the four states are missing, linked here, linked elsewhere and a real folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsLink.test.ts#L27 ]
[test: the specs link row says what is at <code repo>/specs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L77 ]
[test: create makes a relative link to the data repo's spec tree, once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsLink.test.ts#L44 ]
[test: remove unlinks a symlink, whatever it points at, and refuses a real folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specsLink.test.ts#L59 ]
[test: shows each repo's specs link and saves every change the moment it is made : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L129 ]
[test: a refusal shows as the error line and leaves the page as it was : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L241 ]

## S2: Add and remove code repositories @v1 [proposed]

- Given: the Project settings page
- When: the user adds a repository — by typing a path, or browsing to its
  folder where the app has a native chooser
- Then: it is written to `code-repos`: a bare name is taken as a checkout
  beside the data folder (`acme-app` → `../acme-app`), and a folder picked
  under the data folder's parent is stored relative the same way
- and a row shows the path, the folder it resolves to, and **not found** when
  the folder is not there
- When: the user removes a repository
- Then: the project stops naming it; the folder and its symlink stay as they
  are
- and the first repository added becomes the default
- and the agent pane offers every listed repository as a folder to run in

[test: code repos are stored relative when beside the data repo, the default only when not first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L112 ]
[test: a sibling path, a nested one and a stranger's are stored as the file should hold them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L132 ]
[test: a bare folder name is a sibling of the data folder; paths are kept as typed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L60 ]
[test: the folder menu offers the code repo, the data repo, then folders sessions used : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L359 ]

## S3: Choose the default code repository @v1 [proposed]

- Given: two or more code repositories listed
- When: the user picks **Default** on one of them
- Then: `default-code-repo` names it — written only when it is not the first
  of the list, which is the default otherwise
- and coding sessions, the Issues tab, IntelliJ test links and the entity
  scan all use the default repository

[test: code-repos lists several, the first the default unless default-code-repo says otherwise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L42 ]

## S4: Edit the project members @v1 [proposed]

- Given: the Project settings page
- When: the user adds a member as a name, with the email in angle brackets
  if they like, or removes one
- Then: `members` holds them as `{ "name", "email" }` objects, in order,
  without duplicate names
- and a feature's Assigned dropdown offers the new list

[test: members are written as objects, in order, without duplicates : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L140 ]
[test: a member is a name, with the email in angle brackets : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L70 ]

## S5: Open Project settings from the user menu @v1 [proposed]

- Given: the app with a data folder open
- When: the user picks **Project settings** in the user menu
- Then: the page takes the editor pane at `/project-settings`, with the code
  repositories, the members and the feature-file authorship switch — the
  same switch Settings has, writing the same key
- and in the demo and on the read-only site the page shows the settings
  without the controls

[test: where the settings are read-only, the rows show and the controls do not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L365 ]
[test: project settings routes match GET and PUT, and the specs link its POST : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L26 ]

## S6: The settings file is created for every data folder @v1 [proposed]

- Given: a data folder with no `project-settings.json`
- When: the app opens it — the sync at start, or the tick after switching
  to it
- Then: the file is made with no members and no code repositories, and the
  next sync commits and pushes it like any saved change
- and a file that is there is never touched, even one that does not parse,
  which is someone's to fix by hand
- and the demo's dataset, which ships its own, is left alone

[test: makes the empty settings once and never touches a file that is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L82 ]
[test: the reload tick makes it for a data folder that has none, but not in the demo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L92 ]
[test: a file that does not parse is refused rather than overwritten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L146 ]
[test: project-settings.json is named alone and stays out of a shared title : https://github.com/SpecDriven/specdriven-app/blob/main/tests/saveSummary.test.ts#L151 ]

## S7: The single code-repo key keeps working @v1 [proposed]

- Given: a settings file from before, with `"code-repo": "../acme-app"`
- Then: it reads as a list of that one repository, the default
- When: the page saves the repositories
- Then: the file holds `code-repos` instead, and the older key is dropped

[test: the older single code-repo key reads as a list of one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L67 ]
[test: a bare member list becomes an object, and the authorship switch keeps the other keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsWrite.test.ts#L155 ]

## S8: A code repo that is not on this computer is offered for cloning @v1 [proposed]

- Given: a data folder whose project settings name a code repository whose
  folder is not on this computer — `../acme-app` beside a data repo cloned
  from GitHub, say
- When: the app opens the data folder — at start, or on switching to it
- Then: it asks once per missing repository, the default first, whether to
  clone it, naming the folder it is cloned into
- and the url is filled in from the data repo's own GitHub owner with the
  repository's folder name — `https://github.com/acme/acme-app` beside
  `github.com/acme/acme-specs` — and can be changed before cloning
- When: the user confirms
- Then: the clone runs in the background, through the app's GitHub sign-in
  as the data repo's clone does, and Project settings opens with the row
  saying it is cloning
- and once the folder is there the row shows it found, with its specs
  symlink state, and coding sessions and the Issues tab can use it
- and a clone that fails says why on the row, leaves no half-made folder,
  and offers **Try again…**
- and a row whose folder is missing offers **Clone…** on the Project
  settings page at any time
- and a clone never lands on a folder that is there, and the demo, the
  read-only site and a hosted worker offer none

[test: the suggested url is the data repo's GitHub owner with the code repo's folder name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoClone.test.ts#L51 ]
[test: a GitHub url is cloned by its canonical url, any other url as typed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoClone.test.ts#L63 ]
[test: the clone runs in the background and its job clears once the folder is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoClone.test.ts#L72 ]
[test: a failed clone keeps its error and leaves no half-made folder behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoClone.test.ts#L100 ]
[test: the route starts the clone, and the folder counts as missing until the clone is done : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoClone.test.ts#L122 ]
[test: a missing repo is cloned from the suggested url, shows the clone running, then is found : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L262 ]
[test: a failed clone shows its error with a way to try again; nothing is offered where cloning is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/projectSettingsPage.test.tsx#L326 ]
