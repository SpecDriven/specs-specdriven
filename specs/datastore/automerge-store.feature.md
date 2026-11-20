# Automerge system of record

**Created by:** vydra

**Updated by:** vydra

The data repo's system of record is a set of Automerge (CRDT) documents, one
per feature. The `.feature.md` files that render on GitHub are deterministic
projections of those documents — same doc, same bytes, on every peer.

Each scenario in a document carries a `status` field: `draft`, `proposed`, or
`published` (crud-feature.feature.md "Scenario status"). Only proposed and
published scenarios are projected into the `.feature.md`; a draft lives in
the document alone.

Each scenario also carries a `rank`: its position among the feature's
scenarios (ui.feature.md "Number scenarios and move them up or down"). The
projection orders by it — then by name and version, which is the whole order
for a document from before the ranks and the tie-break when two peers moved
scenarios at once — and writes the `S<n>:` numbers in that order. A rank is
a scalar set whole, so the last move wins on every peer, and the document's
list of scenarios is never spliced to reorder: each entry keeps its object
and every concurrent edit to it.

## Storage layout

Each document lives inside the data repo under `.specdriven/automerge/`, in
a tree that mirrors the repo's own top-level folders: a document's directory
is its source file's path, with `models/` reading as `event-models/`, plus
one fixed directory for the test management module's document, which has no
markdown side.

```
specs/policy/auto-policy.feature.md                    <- projection
.specdriven/automerge/
    specs/policy/auto-policy.feature.md/               <- the feature's document
        docid                                          <- shared identity
        snapshot-<hash>.bin                            <- full document
        changes-<hash>.bin                             <- incremental changes
    roadmap/<name>.md/                                 <- roadmap pages
    tasks/<name>.md/                                   <- tasks
    notebook/<name>.md/                                <- notebook entries
    event-models/<path>.model.json/                    <- Event Models (models/)
    test-management/project/                           <- test management
```

Chunk files are content-addressed and write-once, so two peers pushing
concurrently never collide on a path: a git merge of their pushes is a plain
union of files, and loading merges every chunk into one document. Once
enough change files pile up, a save folds them into a fresh snapshot and
deletes them from the working tree (compaction).

## Decisions

Git stays the required transport and GitHub access stays the whole
authorization model (ARCHITECTURE.md): the documents ride the same clone,
pull and push as the markdown. No sync server is required; the optional
websocket sync below accelerates delivery without changing the source of
record.

## S1: A feature file deleted outside the app @v1 [published]

**Created by:** vydra

**Updated by:** vydra

When a git pull removes a feature file, the next sync tick removes its document directory too — the markdown is the interchange format, and its deletion carries the intent.

## S2: a repo that predates documents @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Feature files with no document behind them behave as before until touched:
the first save, publish, or sync tick imports them (the document is minted
from the parsed markdown). Nothing breaks for collaborators on older app
versions — their plain-markdown pushes keep working and are absorbed as
drift. If two peers import the same feature concurrently, the merge picks
one document identity deterministically and folds both sides' content in.

## S3: Documents from before the mirrored layout @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Earlier app versions filed documents under a directory per kind —
`features/`, `docs/`, `models/`, `testing/` — with the source file's whole
path beneath it, so every folder name appeared twice
(`features/specs/...`, `models/models/...`). On the first sync tick after
the upgrade, every document found under those roots is moved to its place
in the mirrored layout, identity file included, and the moves land in one
commit of renames, so nothing about a document changes but where it sits.
A document that already has a directory at the new place — a teammate
migrated first, or pushed from an older version since — is merged into it
chunk by chunk: chunk names are content hashes, so a name already there is
the same bytes, and loading still merges every chunk into one document. The
old roots disappear once nothing is left in them.

## S4: Documents from before the status field @v1 [published]

**Created by:** vydra

**Updated by:** vydra

Documents written by earlier app versions carried `published`/`proposed`
booleans per scenario instead of `status`. Loading such a document migrates
it in place — booleans fold into the status field, unmarked scenarios become
drafts — and the migration persists with the document's next save.

## S5: draft scenarios live in the document only @v1 [proposed]

**Created by:** vydra

**Updated by:** jmeowmeow

A draft scenario is internal: it is stored in the feature's document but
never written to the `.feature.md`, so the published spec on GitHub shows
only proposed and published scenarios. The editor exchanges drafts with the
server as an explicit `[draft]` chip on the heading — a wire format the
projection never writes to a file — and reading a feature folds the
document's drafts back into what the editor shows. A chipless heading
hand-added to the markdown is absorbed as a draft — into the document, out
of the file — on the next sync tick. Because markdown cannot see drafts, markdown-driven
reconciliation never deletes them: a draft leaves the document only when
something that can see it (the document itself) removes or promotes it. A
`[proposed]` chip on the draft's name and version promotes the existing
entry in place, keeping its identity and edit history. Deleting a draft is
the other way out: the app removes the entry from the document by name and
version — the deletion markdown cannot express — and the reload cron commits
and pushes the document as it does any other saved change. The operation is
for drafts only; a proposed scenario is deleted by saving the feature
without it. Concurrent status
changes merge as a one-way latch: the side further along
draft → proposed → published wins.

## S6: live sync keeps a disk edit the sync tick has not absorbed @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: live sync is on, and a feature file on peer B was edited on disk —
  by hand or by an agent — after the last sync tick, so the edit is not in
  B's document yet
- When: an edit to the same feature arrives from peer A over the sync server
- Then: B's file holds both A's edit and the disk edit, and B's document
  holds the disk edit too, so the next sync tick finds no drift left
- and B hands the absorbed disk edit to the sync server, so it reaches A
  over the network, not only through git
- and stopping live sync waits for an absorb already running to finish, so
  nothing is written into the data folder after the bridge has stopped

[test: a remote absorb keeps a local disk edit the sync tick has not absorbed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L173 ]
[test: stopping waits for an absorb in flight, and absorbs nothing after : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L219 ]

## S7: markdown edited outside the app @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

A feature file edited where no document ops exist — an older app version, a
hand edit merged through a GitHub PR — drifts from its document's
projection. The next sync tick absorbs the drift into the document (a
scenario-level three-way merge against the git merge base when the pull
conflicted) and rewrites the file to the canonical projection. The GitHub
PR workflow stays a first-class way to edit specs.

## S8: optional live sync over websocket @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Settings can name a sync server (e.g. wss://sync.automerge.org). While one
is named, every feature document is announced to it under an identity all
peers derive from the docid file, local saves stream out as CRDT operations
in near-real time, and arriving operations merge into the chunk store and
rewrite the markdown projection — the next sync tick commits them, so git
remains the source of record. Double delivery over both transports is
harmless: merging the same operations twice is a no-op. The setting is off
by default and the Settings page warns that on a public server anyone
holding a document's id can read that document; peers with private data
point it at a self-hosted sync server instead.

[test: document ids derive deterministically from the docid uuid : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L107 ]
[test: a local save reaches the other peer over websocket alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L116 ]
[test: concurrent edits on both peers merge through the sync server : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L152 ]
[test: a peer that starts after the edit still catches up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L204 ]

## S9: published scenarios stay immutable @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Publishing records the scenario's body in the document, set whole. While the
scenario stays published that recorded text is what every peer renders and
what every file is held to. Drift that changes it is refused: the file is
repaired from the document and the repair is committed, so tampering with
published history — accidental or otherwise — does not stick. In-app writes
are refused at save time, as before. A merge artifact in the collaborative
text — two peers absorbing the same markdown edit, which the merge keeps
twice — is never mistaken for tampering: the sync tick heals the text back
to the recorded body rather than writing the artifact into the file. An
explicit human conflict resolution is the one door that may override the
document's published state; it re-records the body it chooses.

## S10: two peers edit the same scenario concurrently @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Peer A and peer B both edit the body of the same scenario, even the same
line. Each save becomes CRDT operations in the feature's document. When
their pushes meet in a git pull, the markdown conflicts — but the sync tick
resolves it from the merged document, character-level, with no conflict
dialog. Both edits survive on both peers, byte-identical.

## S11: a peer's edit shows on screen as it arrives @v1 [proposed]

An edit from a peer reaches this machine over the sync server within
moments. The screen shows it then, not at the next timed refresh up to a
minute later.

- **Given** live sync is on, and peer B has the app open on a feature
- **When** an edit to that feature arrives from peer A over the sync server
  and changes something
- **Then** B's screen shows the edit as soon as it is in B's file — the
  server tells every open page which feature changed, on the change stream
  the pages already hold open, with no second connection and no polling
- **And** each page re-reads its view by the rules of the timed refresh: the
  feature list, the open feature, the README and the plugins' views, leaving
  alone whatever holds an unsaved edit
- **And** edits that arrive together — a peer catching up after time
  offline — share one re-read rather than one each
- **But** what arrived holding nothing new, and the page's own saves, tell
  the pages nothing
- **And** a page that never hears of an edit (the stream dropped, live sync
  is off) still shows it at its next timed refresh, as before

[test: a peer's edit is announced once it has landed, and only an edit that changed something : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L292 ]
[test: an edit that comes over a real websocket server is announced the same way : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeNetsync.test.ts#L323 ]
[test: the change stream tells every open page which feature a peer changed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/documentsEvent.test.ts#L16 ]
[test: a page hears of a peer's edit apart from the plugin news on the same stream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/documentsEvent.test.ts#L66 ]
[test: a page that asks for plugin news alone listens for nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/documentsEvent.test.ts#L89 ]
[test: a documents event that names no feature still reads as one, with no name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/documentsEvent.test.ts#L96 ]
[test: calls that come together share one run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/coalesce.test.ts#L8 ]
[test: a call during a run earns one more run after it, never two at a time : https://github.com/SpecDriven/specdriven-app/blob/main/tests/coalesce.test.ts#L19 ]
[test: a run that fails does not stop the next call from running : https://github.com/SpecDriven/specdriven-app/blob/main/tests/coalesce.test.ts#L38 ]
[test: cancel drops a run that is waiting : https://github.com/SpecDriven/specdriven-app/blob/main/tests/coalesce.test.ts#L50 ]

## S12: a sync server of its own for each data repository @v1 [proposed]

One sync server address can serve many data repositories without mixing
them: each repository gets a server of its own, with its own documents and
its own storage (cloudflare/sync-server — a Cloudflare Worker with one
Durable Object per repository).

- **Given** Settings names a sync server address with `{repo}` in it, such
  as wss://sync.example.com/{repo}, and the data repository's origin is
  acme/specs on GitHub
- **When** live sync starts
- **Then** the app syncs with that repository's own server,
  wss://sync.example.com/acme/specs, and edits reach the other peers of that
  repository through it
- **And** another data repository behind the same address has a different
  server, which shares no document and no storage with this one
- **And** an address without `{repo}` is dialled as it is, as before
- **But** a data folder whose origin is no GitHub repository has no server to
  go to, and stays on git alone

[test: an address with {repo} becomes the data repository's own server, opened with a pass : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L78 ]
[test: a plain sync server address is dialled as it is, and no GitHub access goes anywhere : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L68 ]
[test: a data folder with no GitHub repository, or no sign-in, has no sync server to go to : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L111 ]
[test: a writer gets a pass, and the pass opens that repository's own sync server : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L134 ]
[test: a pass for one repository does not open another's sync server : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L152 ]
[test: an edit on one peer reaches another through the repository's sync server : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L320 ]

## S13: only people who can push to a repository may use its sync server @v1 [proposed]

- **Given** a sync server that keeps a server per data repository
- **When** the app asks it for access with the user's GitHub sign-in
- **Then** the server asks GitHub whether that sign-in can push to the
  repository, and hands the app a pass for that repository when it can
- **And** the pass lasts an hour, and the app renews it before it runs out
  with nothing for the user to do
- **But** someone who can only read the repository, or whom GitHub does not
  know, gets no pass: nothing of theirs is synced and nothing is sent to them
- **And** without a current pass for that repository — none, a made-up one,
  another repository's, one that ran out — the connection is refused
- **And** someone whose write access was taken away is disconnected when
  their pass runs out, and gets no new one
- **And** a server that cannot be reached is asked again a minute later,
  while one that said no is not asked again until Settings or the sign-in
  change

[test: only a token GitHub says can push to the repository is let in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L72 ]
[test: a token GitHub rejects, a repository it cannot see and a GitHub that is down are all refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L95 ]
[test: someone who can only read the repository gets no pass : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L170 ]
[test: no pass, no connection : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L161 ]
[test: a pass opens the repository it was signed for until it runs out, and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L35 ]
[test: a connection is closed when its pass runs out, and the next check is set for the next one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L478 ]
[test: a pass that was altered or made up is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L44 ]
[test: a server with no signing secret, a plain request and a name that is no repository are turned away : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L189 ]
[test: a no from the sync server is final and in its own words, an unreachable one is tried again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L125 ]
[test: a sync server that could not be reached is asked again in a minute, one that said no is not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L145 ]
[test: a pass is renewed well before it runs out, and never in a hurry : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L160 ]
[test: the pass the app is given opens the sync server's socket for that repository alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L168 ]

## S14: the user's GitHub access goes only where the user pointed it @v1 [proposed]

Proving write access means showing the sync server the user's GitHub
sign-in, so the app is careful where that goes.

- **Given** Settings names a sync server address
- **When** live sync starts
- **Then** the GitHub sign-in is sent only to an address with `{repo}` in
  it — never to a plain sync server such as wss://sync.automerge.org
- **And** only over an encrypted connection (wss://), or to a server on this
  machine
- **And** only after the server said which GitHub it checks against, and
  that is the GitHub the data repository is on
- **And** neither the sign-in nor the pass is written to the log

[test: the GitHub token is not sent to a server that checks another GitHub : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L92 ]
[test: the GitHub token is not sent over an unencrypted connection, except to this machine : https://github.com/SpecDriven/specdriven-app/blob/main/tests/repoSyncServer.test.ts#L100 ]
[test: the server says which GitHub it checks against, so a peer can tell before sending a token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L182 ]

## S15: a repository's sync server keeps its documents while it sleeps @v1 [proposed]

- **Given** every peer of a data repository has disconnected, and its sync
  server is no longer running
- **When** a peer that was away connects later
- **Then** the server starts again with the documents it had, from its own
  durable storage, and the peer receives the edits made while it was away
  with no other peer online

[test: the sync server still has a repository's edits after it slept, for a peer that was away : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L331 ]
[test: what the sync server stores is read back whole, however large : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L218 ]
[test: a range is the keys under a prefix, and no key that merely starts the same : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L238 ]

## S16: a repository's sync server sleeps while nobody is editing @v1 [proposed]

An app stays connected for as long as it is open, edited in or not. So that
an open, idle app costs nothing, the server does not stay running for it.

- **Given** peers connected to a repository's sync server, none of which has
  saved an edit for a while, so that the server has gone out of memory —
  the peers still connected, and none of them told
- **When** one of them saves an edit
- **Then** the server wakes, learns again who is connected from what it
  noted on each connection, and passes the edit on to the others within
  moments — nobody reconnects
- **And** a peer that has sent nothing since the server woke still receives
  it: every peer of a repository is offered every document of that
  repository
- **And** a connection whose pass runs out while the server sleeps is still
  closed on time

[test: the sync server sleeps while its peers stay connected, and passes on the next edit when it wakes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/syncServer.test.ts#L443 ]
