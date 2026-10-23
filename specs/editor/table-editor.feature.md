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

## edit the table as a grid @v1 [proposed]

- Given: the caret is in a markdown table in the body, or the user hovers
  a table in the preview
- When: the user opens the table dialog, edits cells, adds, removes or
  reorders rows and columns, and saves
- Then: the markdown table in the body is rewritten to match
- Otherwise: the user cancels, and the body is unchanged

## insert a table at the caret @v1 [proposed]

- Given: the caret is in ordinary text
- When: the user opens the table dialog, fills in the grid, and inserts
- Then: a markdown table lands at the caret on a paragraph of its own

## variable creates a column @v1 [proposed]

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
