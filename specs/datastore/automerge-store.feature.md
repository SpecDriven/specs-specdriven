# Automerge system of record

The data repo's system of record is a set of Automerge (CRDT) documents, one
per feature. The `.feature.md` files that render on GitHub are deterministic
projections of those documents — same doc, same bytes, on every peer.

Each scenario in a document carries a `status` field: `draft`, `proposed`, or
`published` (crud-feature.feature.md "Scenario status"). Only proposed and
published scenarios are projected into the `.feature.md`; a draft lives in
the document alone.

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

## A feature file deleted outside the app @v1 [published]

When a git pull removes a feature file, the next sync tick removes its document directory too — the markdown is the interchange format, and its deletion carries the intent.

## a repo that predates documents @v1 [proposed]

Feature files with no document behind them behave as before until touched:
the first save, publish, or sync tick imports them (the document is minted
from the parsed markdown). Nothing breaks for collaborators on older app
versions — their plain-markdown pushes keep working and are absorbed as
drift. If two peers import the same feature concurrently, the merge picks
one document identity deterministically and folds both sides' content in.

## Documents from before the mirrored layout @v1 [proposed]

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

## Documents from before the status field @v1 [published]

Documents written by earlier app versions carried `published`/`proposed`
booleans per scenario instead of `status`. Loading such a document migrates
it in place — booleans fold into the status field, unmarked scenarios become
drafts — and the migration persists with the document's next save.

## draft scenarios live in the document only @v1 [proposed]

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

## live sync keeps a disk edit the sync tick has not absorbed @v1 [proposed]

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

[test: a remote absorb keeps a local disk edit the sync tick has not absorbed : https://github.com/SpecDriven/specdriven_app/blob/main/tests/automerge/automergeNetsync.test.ts#L157 ]

## markdown edited outside the app @v1 [proposed]

A feature file edited where no document ops exist — an older app version, a
hand edit merged through a GitHub PR — drifts from its document's
projection. The next sync tick absorbs the drift into the document (a
scenario-level three-way merge against the git merge base when the pull
conflicted) and rewrites the file to the canonical projection. The GitHub
PR workflow stays a first-class way to edit specs.

## optional live sync over websocket @v1 [proposed]

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

[test: document ids derive deterministically from the docid uuid : https://github.com/SpecDriven/specdriven_app/blob/main/tests/automerge/automergeNetsync.test.ts#L91 ]
[test: a local save reaches the other peer over websocket alone : https://github.com/SpecDriven/specdriven_app/blob/main/tests/automerge/automergeNetsync.test.ts#L100 ]
[test: concurrent edits on both peers merge through the sync server : https://github.com/SpecDriven/specdriven_app/blob/main/tests/automerge/automergeNetsync.test.ts#L136 ]
[test: a peer that starts after the edit still catches up : https://github.com/SpecDriven/specdriven_app/blob/main/tests/automerge/automergeNetsync.test.ts#L188 ]

## published scenarios stay immutable @v1 [proposed]

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

## two peers edit the same scenario concurrently @v1 [proposed]

Peer A and peer B both edit the body of the same scenario, even the same
line. Each save becomes CRDT operations in the feature's document. When
their pushes meet in a git pull, the markdown conflicts — but the sync tick
resolves it from the merged document, character-level, with no conflict
dialog. Both edits survive on both peers, byte-identical.
