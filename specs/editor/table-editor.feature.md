# Table editor

A table button in the editor toolbar opens a modal dialog for editing markdown
tables. It replaces the grid that used to sit under the scenario body.

The table is stored as a plain markdown table in the text, so it renders
naturally on GitHub, in the preview tab, and on the scenario cards; the
dialog is a structured view over that text. It is available in every
markdown editor — scenario body, feature description, pages, issues, tests.

- The toolbar button is "Edit table" with the caret inside a table, and
  "Insert table" anywhere else. Tooltip names which.
- In the Preview tab, hovering a table shows an "Edit table" button in its
  top-right corner that opens the same dialog on that table.
- The dialog edits cells and column names in place; adds and removes rows
  and columns; moves rows up and down and columns left and right; sets a
  column's alignment (left, center, right, or none), written as `:---:`
  marks in the divider row and honoured by the preview.
- Pasting a block copied from a spreadsheet (tab-separated cells, one row
  per line) into a cell fills the grid from that cell on, adding rows and
  columns as needed. A single value pastes into the cell as text.
- Keyboard: Tab and Shift+Tab move across cells; Enter moves down a row and
  adds one below the last; Escape cancels; ⌘Enter applies.
- Edits stay in the dialog until Insert (a new table) or Save (an existing
  one). Cancel with unsaved changes asks before discarding. A table being
  edited can be deleted from the text with "Delete table", after confirming.
- A new table is inserted at the caret on a paragraph of its own. Removing
  the last column is not possible; delete the table instead.
- Published scenarios are immutable: the button is not offered, and the
  preview shows no edit control (the table itself still renders).

## S1: edit the table as a grid @v1 [proposed]

- Given: the caret is in a markdown table in the body, or the user hovers
  a table in the preview
- When: the user opens the table dialog, edits cells, adds, removes or
  reorders rows and columns, and saves
- Then: the markdown table in the body is rewritten to match
- Otherwise: the user cancels, and the body is unchanged

[test: finds nothing when the body has no table : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L20 ]
[test: reads the divider's alignment marks : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L35 ]
[test: finds every table, in order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L46 ]
[test: is the table on any of its lines, from its first character to its last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L80 ]
[test: is null in ordinary text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L86 ]
[test: picks the right one of several : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L91 ]
[test: writes alignment into the divider : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L105 ]
[test: rewrites the block in place, with the caret at its start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L124 ]
[test: rewrites only the block it was given : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L132 ]
[test: null removes the table block, closing the gap : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L139 ]
[test: a single value is not a block : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L225 ]
[test: tabs and newlines make cells and rows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L230 ]
[test: fills the grid from the cell on, growing it as needed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L237 ]
[test: pasted into the header, the first row names the columns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L249 ]
[test: edits the table the caret is in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L28 ]
[test: is not offered on a read-only body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L35 ]
[test: carries an edit button in the editable preview : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L46 ]
[test: carries none on a published (read-only) body, and renders as before : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L52 ]
[test: is numbered in text order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L59 ]
[test: is aligned as the divider says : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L63 ]
[test: opens on the table at the caret and saves the edited grid into it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L186 ]
[test: cancels without touching the text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L205 ]
[test: Escape with nothing changed closes it outright : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L218 ]
[test: an Escape with focus outside the dialog is still the dialog's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L227 ]
[test: removing a column moves focus to the one before it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L238 ]
[test: Save is disabled until something changes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L245 ]
[test: removes and reorders columns, and aligns them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L254 ]
[test: the last column cannot be removed; Delete table takes the table out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L264 ]
[test: Enter in the last row adds one below : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L292 ]
[test: fills the grid from a pasted block : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L300 ]

## S2: insert a table at the caret @v1 [proposed]

- Given: the caret is in ordinary text
- When: the user opens the table dialog, fills in the grid, and inserts
- Then: a markdown table lands at the caret on a paragraph of its own

[test: lands at the caret on a paragraph of its own, caret on the blank line after it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L156 ]
[test: uses the blank lines already around the caret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L162 ]
[test: becomes the whole text when it is empty, ending in a newline : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L166 ]
[test: replaces the selection : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L170 ]
[test: inserts a table with the caret in ordinary text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L22 ]
[test: inserts a new table at the caret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L277 ]

## S3: variable creates a column @v1 [proposed]

Scenario bodies only: a `{var}` in the body names an example column.
Variable names use letters, digits, `_` and `-`. Variables inside tables
or code fences don't count.

- When: the table dialog opens on a body that has a variable such as
  {amount}
- Then: the grid has an "amount" column, added to the table if it lacked
  one, or as the first column of a new table
- And: that column can be neither renamed nor removed in the dialog — edit
  or remove the `{var}` in the body instead
- Otherwise: a column of that name already exists, and nothing changes

[test: ignores variables inside any table : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L188 ]
[test: a new table has a column per variable and one empty row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L200 ]
[test: a new table with no variables has one blank column : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L208 ]
[test: adds the missing variable columns to an existing table : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L212 ]
[test: leaves a table that already has them alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/mdtable.test.ts#L218 ]
[test: a new table opens with a column per {var}, locked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L317 ]
[test: an existing table gains the missing column : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L327 ]
[test: outside the scenario body, variables mean nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tableEditor.test.tsx#L340 ]
