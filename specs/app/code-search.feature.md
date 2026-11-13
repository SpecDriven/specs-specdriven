# Code search

**Created by:** vydra

With a code repository open (code-repo-mode.feature.md), the developer finds
things the way an IDE lets them: a file by a few characters of its name, and
a string or a pattern anywhere in the repository's text.

There is no index to build or keep. The list of files is read from the
repository every few seconds and the text is read from disk on every search,
so a result is never older than the files — an agent's or the IDE's edit
shows in the next search. Both searches are read-only: the app still runs no
git command of its own in the repository beyond asking git which files there
are.

## S1: Find a file by name @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user opens the command palette (⌘K or `/`) from the Files tab
  and types a few characters
- Then: the palette lists the repository's files whose name or path holds
  those characters in order — `sapi` finds `src/server/api.ts` — best first:
  a match in the file's name before one in its folders
- and every file of the repository can be found, whether or not the tree
  has opened its folder
- and each row shows the file's name and its path, the matched characters
  marked, and opens the file
- and a file made a moment ago is found within a few seconds

[test: a file is found by part of its name, in a folder the tree never listed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L139 ]
[test: the typed characters need only be in order, and a hit in the name comes before one in the folders : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L150 ]
[test: a file made after the list was read is found once the list is read again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L168 ]
[test: the palette asks the server for files by name and marks what matched in the path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L196 ]

## S2: Find text in the repository @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user presses ⇧⌘F (Ctrl+Shift+F), or the search button in the
  Files tab's head, and types
- Then: the sidebar shows the Files tab with a search field under its head,
  and once the typing pauses the results take the tree's place: each file
  with a match — its name, its folder, how many matches — and under it each
  match's line number and line, the match marked
- and the results arrive while the search is still running, in path order,
  with a line under the field that counts them
- and three switches narrow the search: match case, whole words, and
  regular expression (JavaScript's); without them the text is looked for as
  typed, in either case
- and a regular expression that does not parse says why instead of
  searching
- and a search with very many results stops at the first 2,000 matches or
  200 files and says to narrow it; a file lists its first 100 matches
- and a new query drops the search still running
- and Esc in the field, or the search button again, puts the search away
  and shows the tree as it was

[test: the search routes are registered, for reading only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L83 ]
[test: outside code repository mode there is nothing to search : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L90 ]
[test: a search streams each file with its matches in path order, then the totals : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L179 ]
[test: match case, whole word and regular expression narrow the search : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L193 ]
[test: a search that cannot run is refused before anything is streamed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L221 ]
[test: the search stops at its limits and says there may be more : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L231 ]
[test: a search stops when its reader goes away : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L256 ]
[test: a long line is shortened around its match, and Windows line ends are not part of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L274 ]
[test: a search asks the server with its switches and shows the files as they arrive : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L51 ]
[test: typing waits for a pause, and a new query drops the one still running : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L79 ]
[test: the line under the field says no results, the limit, or why the search failed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L102 ]
[test: closing the search drops the query and its results; ⇧⌘F shows the field again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L130 ]
[test: a result lists the file with its count, then each match's line number and marked line : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L150 ]
[test: the results are a tree of their own, filled from the search as it stands : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L167 ]
[test: the field is a search box with three switches that say whether they are on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L180 ]
[test: the tab registers mod+shift+f, which shows the search field : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L221 ]

## S3: A result opens the file at its line @v1 [proposed]

**Created by:** vydra

- Given: results of a search in the Files tab
- When: the user picks a match, by a click or with ↑/↓ and Enter
- Then: the editor pane opens the file with the match selected and its line
  in the middle of the editor
- and the results stay, and keep the keyboard, so the next match is one key
  away
- and picking a file's own row opens it at its first match
- and when the file has changed since the search, so that the line is no
  longer there, the editor goes to the file's last line

[test: a plain click on a result opens it in the app, and a modified click is left to the browser : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L261 ]\n[test: a place asked of the editor is selected, at once and when asked later, a line past the end being the last : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeLanguage.test.tsx#L379 ]

[test: a result asks the editor for its place once, and the request is dropped when another file opens : https://github.com/SpecDriven/specdriven-app/blob/main/tests/filesSearch.test.tsx#L232 ]

## S4: Search leaves out what the repository ignores @v1 [proposed]

**Created by:** vydra

- Given: a code repository open in the app
- When: the user searches for a file or for text
- Then: `.git` and whatever the repository's `.gitignore` files exclude
  (`node_modules`, build output) are neither found nor searched; a tracked
  file is, even under an ignored folder
- and without git on the machine, or in a folder that is not a git
  repository, the `.gitignore` files are read by the app itself, a folder's
  own file adding to the ones above it
- and the text search passes over what the editor would not open as text —
  a binary file, one that is not UTF-8, one too large to edit here — and
  over links

[test: without git the list is every file but .git and what the .gitignore files exclude : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L98 ]
[test: a folder's own .gitignore adds to the ones above it, and can take a file back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L109 ]
[test: in a git repository the list is git's own, tracked files and untracked ones that are not ignored : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L122 ]
[test: an ignored file is never found by name, and nothing typed finds nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L161 ]
[test: ignored files, git's folder, binary files, files too large to open and links are not searched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/codeSearch.test.ts#L206 ]
