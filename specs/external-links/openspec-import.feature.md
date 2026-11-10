# OpenSpec import

Teams that already use OpenSpec (openspec.dev) keep their specs in the code
repository: one `openspec/specs/<capability>/spec.md` per capability, each a
`## Purpose` and `### Requirement: <name>` sections holding a SHALL
statement and `#### Scenario: <name>` sections of GIVEN/WHEN/THEN bullets.
Import from OpenSpec reads those files off disk and writes one feature per
capability, so the team can carry on in SpecDriven without retyping them.

Only the specs are imported — the current behaviour. `openspec/changes`
(proposals, delta specs, tasks and the archive) is work in flight and is
never read.

The files are already structured markdown, so no agent and no network are
involved, and nothing needs a key: the option is always offered, except in
the demo and the hosted app, which have no code repository on disk. Nothing
is written to the code repository.

Not in this version: importing a capability again into the feature it made
(to pick up later OpenSpec edits), and importing changes.

## S1: Capabilities become features @v1 [proposed]

- Given: a code repository with `openspec/specs/auth/spec.md` and
  `openspec/specs/payments/spec.md`
- When: the user picks "Import from OpenSpec…" — the folder's ⋯ menu ("Import
  from OpenSpec here…") or the palette — leaves both capabilities ticked and
  presses Import
- Then: each capability becomes a new feature file in that folder, named
  after the capability folder — `auth` → "Auth", `user-auth` → "User Auth" —
  so the same capability always lands on the same file
- and its description is the spec's `## Purpose`, after a first line naming
  the file it came from ("Imported from OpenSpec openspec/specs/auth/spec.md")
- and a capability nested in folders (`openspec/specs/billing/invoices`)
  lands in the same folders under the chosen one
- and the files are committed, the first one opened, and a toast says how
  many features and scenarios were imported

[test: each chosen capability becomes a feature in the folder; a second import skips them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L300 ]
[test: names — the capability folder title-cased, nested capabilities keep their folders, repeats told apart : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L177 ]
[test: reads the title, purpose, requirements with their statements, and scenarios : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L75 ]
[test: what was written, what was there already, and what failed, in one line : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecDialog.test.tsx#L64 ]

## S2: Changes in flight are never imported @v1 [proposed]

- Given: the code repository also holds `openspec/changes/add-2fa/` with a
  proposal and a delta spec under `specs/auth/spec.md`
- When: the user imports from OpenSpec
- Then: only `openspec/specs` is listed and read; nothing from
  `openspec/changes` is offered or written
- and the dialog says so: "Changes in openspec/changes are not imported."

[test: lists each capability under openspec/specs and nothing from openspec/changes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L242 ]
[test: a modal dialog that says where the features land and that changes are not imported : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecDialog.test.tsx#L30 ]

## S3: Pick the code repository to read @v1 [proposed]

- Given: the data repo's project settings name one or more code repositories
- When: the user opens "Import from OpenSpec…"
- Then: the default code repository is read first, and the others can be
  picked from "Read from"
- and "Another folder…" takes any folder on this machine, since the specs
  only usually live in a code repository
- and each capability found is listed with its requirement and scenario
  counts, ticked
- and with no code repository in the project settings, or no
  `openspec/specs` in the one picked, the dialog says so instead of listing

[test: the default code repo's capabilities, with what each becomes and whether it is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L254 ]
[test: another folder can be typed; one without specs, or no code repo at all, says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L281 ]

## S4: Scenarios keep their requirement @v1 [proposed]

- Given: an OpenSpec requirement "User Authentication" with the scenarios
  "Valid credentials" and "Invalid credentials"
- When: the capability is imported
- Then: each OpenSpec scenario becomes one proposed scenario, in OpenSpec's
  order, named "User Authentication - Valid credentials"
- and its body is the requirement's statement as a quote, then the
  scenario's GIVEN/WHEN/THEN bullets as written
- and a requirement without scenarios becomes one scenario holding its
  statement
- and nothing in the text is read as the spec's own syntax: headings become
  bold text, and lines that look like test links, flags, attributes, tickets
  or authorship are escaped
- and two scenarios that would share a name are told apart with " (2)"

[test: one proposed scenario per OpenSpec scenario, the requirement quoted on top, in OpenSpec's order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L125 ]
[test: lines the feature parser would lift out stay in the body; headings become bold : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L149 ]
[test: a heading inside a code fence is text, and other headings stay in what they sit in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L94 ]
[test: lenient — Requirement headings without a Requirements section, CRLF, a bare ### under Requirements : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L115 ]

## S5: Specs already imported are skipped @v1 [proposed]

- Given: "auth" was imported into this folder before
- When: the user opens "Import from OpenSpec…" for the same folder
- Then: "auth" is listed greyed out, "already imported as
  specs/…/auth.feature.md", and cannot be ticked
- and importing it anyway (another window, a stale list) leaves the feature
  as it is and reports it as already imported — edits made in SpecDriven
  are never overwritten

[test: each capability with its counts; one already imported, or unreadable, cannot be ticked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecDialog.test.tsx#L45 ]
[test: each chosen capability becomes a feature in the folder; a second import skips them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L300 ]

## S6: What cannot be imported says why @v1 [proposed]

- Given: a spec that cannot be read (binary, too large), or a capability the
  folder no longer holds
- When: the user imports
- Then: the others are imported, and the toast names each that was not,
  with the reason
- and when nothing at all was imported, the dialog stays open with the reason
- and the import is refused in the hosted app

[test: a capability the folder does not hold is not imported, and says why; nothing chosen is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L335 ]
[test: refused in the hosted app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/openspecImport.test.ts#L343 ]
