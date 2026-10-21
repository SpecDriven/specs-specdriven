# AGENTS.md in the spec folder

An AI agent asked to write or change specs in a data folder needs to know
the `.feature.md` grammar the app holds the files to — the heading shape,
the status chips, the metadata lines, the canonical form — or its scenarios
silently turn into plain text. The app installs an `AGENTS.md` at the
folder's root that teaches the grammar by example, with a section for the
project's own conventions that the team fills in.

## S1: The sync tick installs AGENTS.md into a folder that has none @v1 [proposed]

- Given: a data folder with no `AGENTS.md` at its root
- When: the app's sync tick runs on it
- Then: the app's template is written as `AGENTS.md`, committed on its own
  as "Add AGENTS.md" and pushed, so the auto-save commit of the same tick
  still names what the user changed
- and a folder that has one, under any spelling of the name (`agents.md`,
  `Agents.md`), is left as it is: the template is a starting point the team
  edits, and the app never writes over it
- and a later tick says nothing about it

[test: the sync tick installs AGENTS.md and pushes it; a later tick leaves it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentsFile.test.ts#L104 ]
[test: the sync tick keeps a teammate's AGENTS.md : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentsFile.test.ts#L120 ]
[test: the template is installed into a folder with no AGENTS.md, once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentsFile.test.ts#L53 ]
[test: a folder's own AGENTS.md is never written over, under any spelling : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentsFile.test.ts#L66 ]

## S2: The template teaches the grammar by example @v1 [proposed]

- Given: the installed `AGENTS.md`
- Then: it describes the folder layout, the file naming, the scenario
  heading `## S<n>: <name> @v<N> [proposed]` and what silently breaks it,
  the metadata lines (ticket and Confluence links, `[negative]`,
  `**Assigned:**`, flags, attributes), the body conventions, the canonical
  form and a checklist to run before finishing
- and its example feature file is canonical: the app parses it and writes it
  back unchanged
- and it ends with a "Conventions in this repo" section for the project's
  own vocabulary, roles and folders

[test: the template's example is a canonical feature file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentsFile.test.ts#L83 ]
