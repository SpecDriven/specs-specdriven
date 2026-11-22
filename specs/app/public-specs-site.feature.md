# Public specs site

The public specs site shows one spec repository, or several to pick from,
to anyone, with nothing to sign in to and nothing to change:
https://specs.specdriven.app shows this repository. It is the app with `SPECDRIVEN_READONLY=1`, deployed as a
Railway service built from the hosted image (app/hosted-app.feature.md),
like the public demo (app/public-demo.feature.md). A visitor sees the Specs
tab, the Roadmap tab, the README and search; every way to create, edit,
move, publish or delete is gone, and so are Sync, Settings, Plugins, the
logs and the coding agents.

## S1: A change pushed to the spec repo shows up on the site @v1 [proposed]

- Given: the site is showing the spec repo's branch
- When: someone pushes a change to that branch
- Then: within the refresh interval (a minute by default) the site fetches
  it and shows it, and an open page re-reads on the same cadence
- and a fetch that fails leaves the site showing what it had

[test: a change pushed to the repo shows up after the next refresh : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L267 ]
[test: shows a pushed change to its scenario after the next refresh : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyOpenPage.test.tsx#L163 ]
[test: is left as it is while nothing changed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyOpenPage.test.tsx#L178 ]
[test: each project is refreshed on its own, a push to the second shows in it alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L277 ]

## S2: A visitor browses the specs and the roadmap without signing in @v1 [proposed]

- Given: the public specs site
- When: anyone opens it
- Then: they see the feature tree, every feature and its scenarios rendered,
  each scenario's own page, the Roadmap pages and the README, and can search
  them
- and the site names the spec repo, no one is signed in, and the only
  plugin is the Roadmap

[test: the state says read-only, names the repo and no one signed in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L174 ]
[test: features and the roadmap are served : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L202 ]
[test: on the hosted image the site listens publicly, keeps HOME off /root and fetches no plugins : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L96 ]

## S3: Nothing a visitor does changes the spec repo @v1 [proposed]

- Given: the public specs site
- When: a request would write anything, or read what the site does not
  show (logs, the data viewer, tasks, settings, tokens)
- Then: the server refuses it with "This site is read-only" before it runs
- and the app serves a copy of the repo with no `.git`, so nothing it does
  can commit or push

[test: only the reads the site shows are allowed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L82 ]
[test: every write and every other read is refused before it runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L238 ]
[test: the app serves a copy with no .git, so nothing it does can commit or push : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L259 ]

## S4: The site shows SpecDriven's own spec repo @v1 [proposed]

- Given: the site is deployed with no repo named
- When: it starts
- Then: it shows SpecDriven/specs-specdriven's main branch, cloned with no
  credentials, so the repo must be public
- and a repo that is not a GitHub https url, a branch name git would read as
  an option, or a bad refresh interval stops the start

[test: the site shows SpecDriven's spec repo unless told otherwise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L22 ]
[test: a repo that is no GitHub url, a branch git reads as an option and a bad cadence stop the start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L52 ]

## S5: A visitor copies a deep link to a feature or a scenario @v1 [proposed]

- Given: the public specs site
- When: a visitor presses "Copy link" on a feature's page, on a scenario's
  card (shown on hover, focus or selection) or on a scenario's own page, or
  presses mod+shift+c
- Then: the absolute url of that feature or scenario is on the clipboard and
  the control flips to "Copied" for 2 sec, so it can be handed to a customer
- and a scenario's link names no version, so it keeps opening the latest
  version after the scenario gets a new one

[test: a versionless scenario link names no version and round-trips to its route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/deepLink.test.ts#L72 ]

## S6: The site shows several projects named in one setting @v1 [proposed]

- Given: the site is deployed with `SPECDRIVEN_READONLY_PROJECTS` listing
  several spec repos, apart by commas, spaces or new lines, each written
  `[<id>=]<repo>[#<branch>]`
- When: it starts
- Then: it clones every repo and shows each as a project of its own, named
  by its repo unless the entry names it, on its own branch or the site's
- and a new visitor sees the first project on the list
- and an entry that is no GitHub https url, two projects of one name, or a
  single `SPECDRIVEN_READONLY_REPO` set beside the list stops the start

[test: several projects are listed in one variable, each named by its repo unless the entry names it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L36 ]
[test: a project list with a bad entry, two projects of one name, or a single repo beside it stops the start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L60 ]

## S7: A visitor picks one of the site's projects @v1 [proposed]

- Given: the public specs site showing several projects
- When: a visitor opens the ▾ next to the folder name
- Then: it lists the projects by name, the repo as each one's tooltip, the
  one shown ticked
- and picking another opens that project's README, and from then on every
  page, feature, search and refresh is that project's, after a reload too
- and a browser tab brought back to the front shows its own project again,
  though another tab picked a different one meanwhile
- and a site showing one project has no ▾

[test: lists them under the folder name's ▾, the visitor's ticked, and links name the project : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyOpenPage.test.tsx#L195 ]
[test: with one project there is no ▾ and links name none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyOpenPage.test.tsx#L218 ]
[test: a visitor's cookie picks the project every request is answered for; any other cookie, the first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L217 ]
[test: a visitor's project is read from their cookie, and only a well-formed id counts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L75 ]
[test: a tab coming back to the front puts its own project back in the cookie : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyProject.test.ts#L54 ]
[test: only well-formed project rows are read from the state : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyProject.test.ts#L63 ]

## S8: A copied link opens the project it was copied in @v1 [proposed]

- Given: the public specs site showing several projects
- When: a visitor presses "Copy link" on a feature or a scenario
- Then: the url names the project (`?project=<id>`)
- and whoever opens it sees that project, the parameter gone from the
  address bar
- and a link naming no project the site knows opens the project the
  visitor had, or the first

[test: a ?project= link picks the project and leaves the address bar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyProject.test.ts#L40 ]
[test: a malformed project id is dropped from the url and picks nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnlyProject.test.ts#L47 ]
