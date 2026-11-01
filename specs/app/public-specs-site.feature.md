# Public specs site

The public specs site shows one spec repository to anyone, with nothing to
sign in to and nothing to change: https://specs.specdriven.app shows this
repository. It is the app with `SPECDRIVEN_READONLY=1`, deployed as a
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

[test: a change pushed to the repo shows up after the next refresh : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L174 ]

## S2: A visitor browses the specs and the roadmap without signing in @v1 [proposed]

- Given: the public specs site
- When: anyone opens it
- Then: they see the feature tree, every feature and its scenarios rendered,
  each scenario's own page, the Roadmap pages and the README, and can search
  them
- and the site names the spec repo, no one is signed in, and the only
  plugin is the Roadmap

[test: the state says read-only, names the repo and no one signed in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L123 ]
[test: features and the roadmap are served : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L135 ]
[test: on the hosted image the site listens publicly, keeps HOME off /root and fetches no plugins : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L50 ]

## S3: Nothing a visitor does changes the spec repo @v1 [proposed]

- Given: the public specs site
- When: a request would write anything, or read what the site does not
  show (logs, the data viewer, tasks, settings, tokens)
- Then: the server refuses it with "This site is read-only" before it runs
- and the app serves a copy of the repo with no `.git`, so nothing it does
  can commit or push

[test: only the reads the site shows are allowed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L36 ]
[test: every write and every other read is refused before it runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L145 ]
[test: the app serves a copy with no .git, so nothing it does can commit or push : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L166 ]

## S4: The site shows SpecDriven's own spec repo @v1 [proposed]

- Given: the site is deployed with no repo named
- When: it starts
- Then: it shows SpecDriven/specs-specdriven's main branch, cloned with no
  credentials, so the repo must be public
- and a repo that is not a GitHub https url, a branch name git would read as
  an option, or a bad refresh interval stops the start

[test: the site shows SpecDriven's spec repo unless told otherwise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L17 ]
[test: a repo that is no GitHub url, a branch git reads as an option and a bad cadence stop the start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/readOnly.test.ts#L28 ]
