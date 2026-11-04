# code-repo-submodule

- add the spec repo to a code repo as a git submodule, from Settings

The app works on its own standalone clone of the spec repo and keeps doing so.
This adds the other direction: the code repo pins the spec repo as a git
submodule, so cloning the code with `--recurse-submodules` brings the specs
that go with it. The action is **Add to a code repo…** in Settings → Git &
GitHub → Code repo submodule.

The submodule points at the spec repo's `origin`, the remote the app pushes
to. The app commits it in the code repo but never pushes the code repo: that
repository is the user's, and sharing it is their call.

`git submodule` needs the git binary: the built-in engine
(git-operations.md) does not implement it, whatever engine Settings has
chosen.

## S1: A submodule that cannot be added is refused, saying why @v1 [proposed]

- Given: the "Add to a code repo" dialog
- When: the user confirms a folder the specs cannot go into
- Then: Settings shows why, and nothing is committed in the code repo
- and that holds when no folder was chosen and project-settings.json names
  no `code-repo`
- and when the folder does not exist, or is not the top folder of a git
  repository
- and when the code repo and the spec repo are the same folder, or one is
  inside the other
- and when the folder name is not a plain relative path, or already exists in
  the code repo
- and when the code repo already has this spec repo as a submodule, under any
  name
- and when the spec repo has no `origin` to point the submodule at — push it
  to GitHub first
- and when `git submodule add` itself failed, with the sign-in hint when it
  was an authentication failure

[test: the refusals say what is wrong : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L151 ]
[test: adding the same spec repo twice is refused, under any name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L175 ]
[test: a spec repo with no origin has nothing to point the submodule at : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L182 ]
[test: a failed git submodule add is reported, with nothing committed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L199 ]

## S2: Add the spec repo to a code repo as a submodule @v1 [proposed]

- Given: the app has a spec repo open that has an `origin` remote, and the
  machine has git installed
- When: the user presses "Add to a code repo…" in Settings and confirms a
  code repo's folder
- Then: the code repo gains the spec repo as a submodule — `.gitmodules`
  names the spec repo's origin, and a gitlink pins the commit the app's clone
  is on
- and the code repo has a new commit, "Add <name> specs as a submodule",
  holding `.gitmodules` and the submodule's folder and nothing else; anything
  the user had staged there stays staged
- and the code repo is not pushed, and the app's own clone is left as it was
- and a fresh `git clone --recurse-submodules` of the code repo has the specs
- and a toast says where the specs were added, and that the code repo still
  needs a push

[test: the route exists : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L94 ]
[test: the spec repo is added to the code repo as a submodule on its branch, and committed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L98 ]
[test: a clone of the code repo with --recurse-submodules brings the specs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L122 ]
[test: only the submodule is committed, whatever else is staged in the code repo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L129 ]

## S3: Choosing the code repo and the submodule's folder @v1 [proposed]

- Given: the "Add to a code repo" dialog
- When: it opens
- Then: the path field starts with `code-repo` from the data repo's
  project-settings.json, when one is set
- and on runtimes with a native folder chooser, "Browse…" opens one titled
  "Choose the code repository"; the request names only what the folder is
  for, never the title's text
- and a relative path resolves against the spec repo, as `code-repo` does
- and the submodule's folder is named after the spec repo (`specs-insurance`
  for SpecDriven/specs-insurance), unless a name is given

[test: the folder name can be chosen, and code-repo in project-settings.json is the default code repo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L137 ]
[test: a relative code repo resolves against the spec repo, as code-repo does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L145 ]
[test: the default folder name is the spec repo's name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L225 ]
[test: the dialog title follows the purpose, never free text from the request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L160 ]
[test: the Linux dialog shows the title it is given : https://github.com/SpecDriven/specdriven-app/blob/main/tests/folderChooser.test.ts#L184 ]

## S4: Not offered where it cannot run @v1 [proposed]

- Given: the app runs hosted (app/hosted-app.feature.md), as the public demo
  (app/public-demo.feature.md), or on a machine without git
- When: a submodule is asked for
- Then: hosted and in the demo it is refused (403); the Git & GitHub section
  is not shown there either
- and without git it is refused (501), and the Settings button is disabled
  with "Needs git installed"

[test: refused when hosted, in the demo, and without system git : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L208 ]

## S5: The submodule follows the branch the app works on @v1 [proposed]

- Given: the app's clone of the spec repo is on a branch, e.g. `main`
- When: the spec repo is added to a code repo as a submodule
- Then: `.gitmodules` records that branch, so `git submodule update --remote`
  in the code repo brings the newest specs from it
- and a clone on a detached HEAD records no branch

[test: the spec repo is added to the code repo as a submodule on its branch, and committed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L98 ]
[test: a detached spec repo records no branch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeRepoSubmodule.test.ts#L189 ]
