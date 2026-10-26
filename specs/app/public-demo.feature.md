# public-demo

The public demo lets anyone try SpecDriven in a browser with nothing to
install and nothing to sign in to. It is the app in demo mode
(`DEMO_MODE=1`): it serves a fresh copy of the checked-in demo dataset, a
snapshot of the specs-insurance repo, and every page load starts it afresh,
so a visitor can edit, publish and run a coding session without changing
anything for the next visitor. Nothing it does reaches GitHub or the
visitor's machine.

It is deployed as a container of its own (the `Dockerfile` in the code
repo), or as a Railway service built from the hosted image
(app/hosted-app.feature.md). Being a network service, it links to the source
of the running version (app/open-source-license.feature.md). Its plugins
start switched off (app/plugins.feature.md "The profile lists enabled
plugins in tab order"). What a demo could show next is on the roadmap
(roadmap/demo.md).

## S1: A coding session is simulated @v1 [proposed]

- Given: the demo, with one or more scenarios picked
- When: the visitor launches a coding session
- Then: it runs in the agent pane, the only target on offer, and plays a
  scripted Claude Code turn instead of starting an agent: it names the
  picked scenarios, shows Read and Grep rows, streams its text, and stops
  at a permission card for a Write
- and allowing the Write finishes the turn; denying it closes the turn and
  says it was denied
- and a follow-up message gets a scripted reply, Stop returns the session
  to idle without ending it, and a chat started with no scenarios gets a
  short reply rather than the scenario script
- and the pane marks the session as a simulation, and the whole session
  writes nothing into the demo's files

[test: the coding agent pane is on and is the only target on offer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L123 ]
[test: a demo session starts and plays a scripted Claude Code turn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L191 ]
[test: readBrief pulls the name and the specs out of the generated prompt : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L140 ]
[test: the script streams its text before settling it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L163 ]
[test: allowing the write finishes the turn and returns the session to idle : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L223 ]
[test: denying the write still closes the turn, and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L240 ]
[test: a follow-up message gets a scripted reply, and the session stays up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L257 ]
[test: a chat started with no specs gets the short reply, not the spec script : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L273 ]
[test: Stop mid-turn returns the session to idle without ending it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L289 ]
[test: the whole session writes no file into the demo dataset : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L297 ]

## S2: A page load starts the demo afresh @v1 [proposed]

- Given: the public demo
- When: a visitor loads or reloads the page
- Then: the demo dataset is copied to a new working folder, and the app
  opens on it
- and editing works in full — the README, features, scenarios, roadmap
  pages — and publishing a feature succeeds, all inside that copy
- and the checked-in dataset itself is never written, so the next page load
  shows it as it was
- and page loads at the same moment each end up on a complete copy

[test: a page load reseeds — the edit is gone, the folder is a new generation : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L225 ]
[test: edits land in the working copy; the pristine snapshot is untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L208 ]
[test: publishing a feature succeeds locally and stays inside the working copy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L247 ]
[test: a roadmap edit lands in the working copy; the snapshot is untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L191 ]
[test: concurrent page loads serialize into consistent generations : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L239 ]

## S3: Nothing in the demo reaches GitHub or the visitor's machine @v1 [proposed]

- Given: the public demo
- When: anything asks it to sign in to GitHub, open another data folder,
  change the git engine, point live sync at a server, add a plugin folder,
  trust one or add a marketplace
- Then: it answers 403, and the app offers none of these
- and there is no native folder chooser: asking for one answers 501
- and the working copy has no git history, so saves stay local and pull and
  push are skipped
- and the Issues tab reads the public SpecDriven/specdriven-app-release
  repository anonymously (app/github-issues.feature.md), and every write to
  it is refused
- and switching plugins on and off, and every cosmetic setting, still saves
  as on the desktop

[test: GitHub sign-in routes refuse in demo mode : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L320 ]
[test: the data directory, git engine and sync server are pinned (403) : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L263 ]
[test: plugin folders, trust and marketplaces are pinned (403); switching plugins still saves : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L285 ]
[test: no native folder chooser — /api/choose-folder is 501 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L362 ]
[test: the demo tracks the release repo's issues and refuses every write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L338 ]

## S4: The demo opens on its own dataset @v1 [proposed]

- Given: a visitor opening the public demo
- When: the page loads
- Then: the app opens straight on the demo's working copy, with no
  first-run folder form, the bundled JavaScript git engine and no sync
  timer
- and the user is "Test User", not the server's operating-system account;
  the app version shows beside it, and the user menu links to the source
  of that version
- and every optional setting is on except "Open test links in IntelliJ
  IDEA", which would run `idea` on the visitor's machine; the plugins start
  switched off, and the dataset has content for each one a visitor turns
  on

[test: boot state names the demo — ephemeral folder, no chooser, js git, no cron : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L110 ]
[test: the demo user is Test User, not the OS account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L379 ]
[test: state carries the app version — the demo shows it beside the user : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L367 ]
[test: state carries the source url of the running version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L374 ]
[test: Specs alone, and every other optional setting on except IntelliJ IDEA : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L150 ]
[test: a visitor turns the roadmap on and it lists the dataset's pages : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L135 ]

## S5: The hosted image runs the demo when DEMO_MODE is set @v1 [proposed]

- Given: a Railway service built from the app's repository, which builds
  every service with the hosted image
- When: the service sets `DEMO_MODE=1`
- Then: the image's entry point runs the public demo instead of the sign-in
  gateway: no login page, none of the hosted app's variables, and no volume
- and the demo runs as the image's unprivileged `bun` user even though the
  hosted image starts as root; an image without that user or `setpriv`
  runs it as root and logs a warning
- and it gets the settings the demo's own container gives it — its working
  folder under `/tmp/specdriven-demo`, listening on every interface, any
  host name accepted, no log file — unless the service sets them
- and it answers the platform's health check at `/healthz`

[test: DEMO_MODE on the gateway's entry point serves the demo, with no hosted variables : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDemo.test.ts#L37 ]
[test: as root the demo drops to the bun user when the image has one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDemo.test.ts#L20 ]
[test: the demo gets Dockerfile's settings unless they are set already : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDemo.test.ts#L12 ]
