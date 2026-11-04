# Test Traceability

**Created by:** vydra

**Updated by:** vydra

Trace tests to indivitual specs (scenarios / rules).

In '.md' file store tests as [test:<url to test>]

Ex: [test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]

Internal TS interface RenderTestInEditor will format the test for user display.

Display test as QuoteHomeownersPolicyTest.

[test: <test name> : link in GitHub]

## S1: a coding agent's edit to a test re-anchors the links to it @v1 [proposed]

- When: Claude Code edits or writes a test file under `tests/` in the app repo
- Then: the project hook (`scripts/test-links.ts`, registered in `.claude/settings.json`) checks every `[test: <name> : <url>#L<n> ]` link in the spec repo that points at that file
- And: a link is right only when its anchor is the line declaring the named test — a multi-line `test.each([…])("title")` and a template title count, and links inside code do not
- And: a test that moved in its file is re-anchored, one that moved to another file is re-pointed, and a bare url is given the name at its anchor
- And: a link whose test is gone is reported back to the agent; an edit outside `tests/` is let through without a word

[test: only links into this repo's tests count, and none inside code : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L35 ]
[test: a multi-line test.each is named by its closing title, template titles by their pattern or head : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L51 ]
[test: a link whose anchor names its test is right, a test.each head included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L66 ]
[test: a test that moved in its file is re-anchored, one that moved file is re-pointed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L72 ]
[test: a bare url is given the name at its anchor : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L80 ]
[test: a name no test carries any more is broken : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L84 ]
[test: a rewrite touches the token it names and leaves code alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L89 ]
[test: after an edit to a test file the links into it are re-anchored, and Claude is told : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L143 ]
[test: an edit outside tests/ is let through without a word : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L150 ]

## S2: a coding agent's turn ends with its tests linked @v1 [proposed]

- When: a Claude Code turn ends with test files changed on the branch
- Then: the links into those files are checked and re-anchored as after an edit
- And: a link that no longer names a test, or a test added on the branch that no scenario links to, keeps the agent working once with the list of them
- And: if the agent stops again with them still open, they are shown as a warning and the turn ends; a turn that leaves every link right ends quietly
- And: `bun scripts/test-links.ts` runs the same check by hand — `--fix` re-anchors, `--changed` limits it to the branch

[test: at the end of a turn a broken link or an unlinked new test blocks once, then only warns : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L155 ]
[test: a turn that leaves every link right ends quietly : https://github.com/SpecDriven/specdriven-app/blob/main/tests/testLinks.test.ts#L166 ]

## S3: a reference inside code is code @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: `[test:<url>]` sits inside backticks or a fenced code block
- Then: it is shown as code, not as a test link

## S4: a reference names the test function @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a scenario or rule carries `[test: quotesAHomeownersPolicy : https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32 ]`
- Then: the scenario card, the editor preview and the Automated tests field show a link reading `quotesAHomeownersPolicy` — the name alone, never the file name
- And: the link opens the full url, line anchor included, and the tooltip shows the full url
- And: a reference without a name, `[test: <url> ]`, is still read and still shown as the file name

## S5: a test name the token cannot carry is refused @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a name typed into the Name field contains `:` or `]`
- Then: the row cannot be added or saved, with a message saying so

## S6: a Tests heading over the author's own notes stays in the body @v1 [proposed]

- When: a scenario body carries a `### Tests` heading with prose or steps under it, a url to somewhere other than GitHub, or more text after the links
- Then: the heading and everything under it are the scenario's own text — kept on save, shown in the body, never read as the older table
- And: only a trailing `### Tests` section holding nothing but GitHub links and the table's own framing is read as the older table

Read as the table, every line under the heading that held no url used to be dropped the next time the scenario was saved, with no error — once, a manual checklist and the steps after it.

[test: a Tests heading over notes or steps stays in the body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/traceability.test.ts#L363 ]
[test: a Tests heading followed by another scenario-like section keeps both : https://github.com/SpecDriven/specdriven-app/blob/main/tests/traceability.test.ts#L376 ]
[test: a Tests section whose only link points off GitHub is content : https://github.com/SpecDriven/specdriven-app/blob/main/tests/traceability.test.ts#L387 ]
[test: a Tests heading with notes under it survives the save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/features.test.ts#L961 ]

## S7: a Tests table in an older file is still read @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a feature file still carries a trailing `### Tests` table
- Then: its urls are the scenario's test links all the same
- And: the next time the app writes the file they come out as `[test: <url> ]` lines

## S8: a url without a line anchor is turned away @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the url in the Automated tests field has no `#L<n>` anchor
- Then: Add is disabled and a Save with it pending is blocked with "Needs a line anchor"
- And: a url from an older file that has no anchor is still read and shown

## S9: an unsafe or malformed reference is left as text @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the url in `[test:<url>]` is not an http or https url, or the token is followed by `(` like an ordinary markdown link
- Then: it is shown as it was typed — an ordinary markdown link keeps working, and `[test:javascript:…]` never becomes a link

## S10: no name can be read @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the file cannot be fetched (a private repository without a sign-in, no such file), the language is not one the app reads, or no test is declared near the line
- Then: the link is still added, with an empty name, and a hint under the field says why nothing was found
- And: typing a name by hand fills it in

## S11: the reference stays in the feature file @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a scenario with a `[test: <url> ]` reference is saved, proposed or published
- Then: a reference on a line of its own is the scenario's test link — shown in the Automated tests table, editable in the pane — and is written back as a `[test: <url> ]` line at the end of the scenario, never as a `### Tests` table
- And: a reference inside a rule's line stays in the body exactly where it stood
- And: GitHub renders either as plain text

## S12: the test name is read from the code on GitHub @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a GitHub url with a `#L<n>` line anchor is typed into the Automated tests field, or an existing row's url is changed
- Then: the app fetches the file through the GitHub API with the app's sign-in token and fills the Name field with the test function declared at that line
- And: an anchor on a `@Test` annotation, decorator or comment names the declaration right under it; an anchor inside a test's body names the test it sits in
- And: Java and Kotlin methods, `test(…)` / `it(…)` titles and functions in TypeScript and JavaScript, Python `def`s and Go `func`s are recognised
- And: the name can be changed by hand before or after it is filled in, and is written to the file as `[test: <name> : <url> ]`

## S13: the test name is the file name @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a `[test:<url>]` url ends in a file with an extension, a `#L<n>` line anchor or a query string
- Then: the display name is the file name with the extension, anchor and query dropped
- And: a url with no path is displayed as the url itself

## S14: trace a test to a rule @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a rule inside a scenario body carries `[test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]`
- Then: the scenario card and the editor preview show that rule with a link reading `QuoteHomeownersPolicyTest`
- And: the link opens the full url, line anchor included, in a new tab, and the tooltip shows the full url
