# Data viewer

**Assigned:** David

A read-only browse of the Automerge store (datastore/automerge-store.feature.md
"Storage layout"), the way a SQL tool browses a database: the store's
documents flattened into tables on the left, a grid of rows in the middle, and
from a row its document — as its chunk files hold it, no schema check and no
migration — with its change history and the document as it stood at any
change. Its own page in the editor pane at `/data`, opened from the user menu
like Observability (`g d`), for developers and admins: raw field names, docids,
heads, actors and chunk files stay visible. Nothing here writes to the data
repo.

## A document reads as its chunks hold it @v1 [proposed]

- Given: a document whose directory holds a snapshot and change files
- When: the user opens a row's document
- Then: the Document view shows its kind, the data repo file it mirrors (none
  for the test management document), its docid, heads, schema version and
  change, op and actor counts, every chunk file with its kind, size and
  whether it applied, and the document as a folding JSON tree
- and a document still at an old schema reads as written — a pre-v3 feature
  keeps its `scenarios` list — and a document newer than the app opens too,
  while the editor shows its outdated screen
- and a chunk that could not be applied is named, never fatal

[test: loadRawDoc reads a pre-v3 document as written, lists its chunks and skips a corrupt one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeStore.test.ts#L305 ]
[test: readDoc returns what is on disk with its chunk files : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L213 ]
[test: sourceFileOf maps a store directory back to its data repo file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeStore.test.ts#L344 ]
[test: keep answering while the data is newer than the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L313 ]

## A document's history lists its changes newest first @v1 [proposed]

- Given: a document with more than one change
- When: the user opens its History view
- Then: the changes are listed newest first in Automerge's causal order,
  each with its hash, time, actor, sequence number, message and op count,
  fifty at a time with "Load older"

[test: history lists the changes newest first, and a change shows the document then and its patches : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L228 ]
[test: lists the tables, sorts and filters through the server, opens a row, its document and its history : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L219 ]

## A row opens as a tree and leads to its document @v1 [proposed]

- Given: a table's rows
- When: the user clicks a row
- Then: a drawer beside the grid shows the row as a folding JSON tree —
  nested values, which the grid summarises as `[2]` or `{3}`, open there
- and "Open document" switches to the Document view of the document the row
  came from, and the Document and History views stay reachable for it

[test: lists the tables, sorts and filters through the server, opens a row, its document and its history : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L219 ]
[test: opens branches to the given depth and colours scalars : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L185 ]
[test: cells read as text, nested values as a summary : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L163 ]

## Picking a change shows the document then and what it touched @v1 [proposed]

- Given: the History view of a document
- When: the user picks a change
- Then: the document as it stood at that change shows as a tree, with one
  line per patch the change made against its parents — `splice /specs/1/body
  "…"`, `put /specs/0/name = "…"`
- and the first change, having no parents, says so instead of listing every
  value as a patch

[test: history lists the changes newest first, and a change shows the document then and its patches : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L228 ]
[test: changes and patches read as lines : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L174 ]

## Rows are filtered, sorted and paged on the server @v1 [proposed]

- Given: a table with rows
- When: the user types in the filter, clicks a column header, or pages
- Then: the server answers with the rows whose cells contain the text, case
  aside — nested values by their JSON — sorted by the column (numbers as
  numbers, empty cells last either way; a second click flips the direction),
  a hundred at a time, and the toolbar says "1–100 of 1,431"
- and a column the table does not have is refused

[test: queryRows filters over scalar and nested cells, sorts, and pages : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L184 ]
[test: readTable serves a page and refuses a column it does not have : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L202 ]
[test: serve tables, rows, a document, its history and a change : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L267 ]
[test: urls leave defaults out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataPage.test.tsx#L152 ]

## The store's documents are listed as tables @v1 [proposed]

- Given: a data folder with features, pages, event models and the test
  management document
- When: the user opens Data
- Then: the left list names documents, features, specs, pages, test cases,
  test plans, test executions and event models, each with its row count,
  and how many documents the store holds
- and documents is one row per document directory, whatever wrote it — a
  plugin's folder the app never heard of is listed too; specs is every
  entry of every feature, with its feature's path, its ids, status, rank
  and published body; the test management document's maps and an event
  model's top-level keys become rows and columns of their own
- and a save made here is seen by the next request; "Rescan" reads the
  store again regardless

[test: every table lists its rows, from documents loaded as they are on disk : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L122 ]
[test: specs are flattened with their feature, publishedBody read as text, nested values wrapped : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L139 ]
[test: the documents table carries identity, heads, chunks and sizes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L155 ]
[test: the testing document's maps and a plugin's document become rows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L170 ]
[test: Data has its own url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L340 ]
[test: listDocDirs finds every document directory and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeStore.test.ts#L296 ]
[test: a local save is seen by the next request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L253 ]

## The viewer never writes and never leaves the store @v1 [proposed]

- Given: the viewer's routes
- When: a request names a document path outside the store — `../..`, an
  absolute path, a dot-led segment — or a change hash that is not one
- Then: it is refused, and no request of the viewer saves, migrates or
  compacts a document

[test: loadRawDoc stays inside the store : https://github.com/SpecDriven/specdriven-app/blob/main/tests/automerge/automergeStore.test.ts#L335 ]
[test: refuse what is not a table, a column, a store path or a change hash : https://github.com/SpecDriven/specdriven-app/blob/main/tests/dataViewer.test.ts#L296 ]
