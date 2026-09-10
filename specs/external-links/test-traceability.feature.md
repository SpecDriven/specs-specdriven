# Test Traceability

Trace tests to indivitual specs (scenarios / rules).

In '.md' file store tests as [test:<url to test>]

Ex: [test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]

Internal TS interface RenderTestInEditor will format the test for user display.

Display test as QuoteHomeownersPolicyTest.

[test: <test name> : link in GitHub]

## a reference inside code is code @v1 [proposed]

- When: `[test:<url>]` sits inside backticks or a fenced code block
- Then: it is shown as code, not as a test link

## a reference names the test function @v1 [proposed]

- When: a scenario or rule carries `[test: quotesAHomeownersPolicy : https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32 ]`
- Then: the scenario card, the editor preview and the Automated tests field show a link reading `quotesAHomeownersPolicy` — the name alone, never the file name
- And: the link opens the full url, line anchor included, and the tooltip shows the full url
- And: a reference without a name, `[test: <url> ]`, is still read and still shown as the file name

## a test name the token cannot carry is refused @v1 [proposed]

- When: a name typed into the Name field contains `:` or `]`
- Then: the row cannot be added or saved, with a message saying so

## a Tests table in an older file is still read @v1 [proposed]

- When: a feature file still carries a trailing `### Tests` table
- Then: its urls are the scenario's test links all the same
- And: the next time the app writes the file they come out as `[test: <url> ]` lines

## a url without a line anchor is turned away @v1 [proposed]

- When: the url in the Automated tests field has no `#L<n>` anchor
- Then: Add is disabled and a Save with it pending is blocked with "Needs a line anchor"
- And: a url from an older file that has no anchor is still read and shown

## an unsafe or malformed reference is left as text @v1 [proposed]

- When: the url in `[test:<url>]` is not an http or https url, or the token is followed by `(` like an ordinary markdown link
- Then: it is shown as it was typed — an ordinary markdown link keeps working, and `[test:javascript:…]` never becomes a link

## no name can be read @v1 [proposed]

- When: the file cannot be fetched (a private repository without a sign-in, no such file), the language is not one the app reads, or no test is declared near the line
- Then: the link is still added, with an empty name, and a hint under the field says why nothing was found
- And: typing a name by hand fills it in

## the reference stays in the feature file @v1 [proposed]

- When: a scenario with a `[test: <url> ]` reference is saved, proposed or published
- Then: a reference on a line of its own is the scenario's test link — shown in the Automated tests table, editable in the pane — and is written back as a `[test: <url> ]` line at the end of the scenario, never as a `### Tests` table
- And: a reference inside a rule's line stays in the body exactly where it stood
- And: GitHub renders either as plain text

## the test name is read from the code on GitHub @v1 [proposed]

- When: a GitHub url with a `#L<n>` line anchor is typed into the Automated tests field, or an existing row's url is changed
- Then: the app fetches the file through the GitHub API with the app's sign-in token and fills the Name field with the test function declared at that line
- And: an anchor on a `@Test` annotation, decorator or comment names the declaration right under it; an anchor inside a test's body names the test it sits in
- And: Java and Kotlin methods, `test(…)` / `it(…)` titles and functions in TypeScript and JavaScript, Python `def`s and Go `func`s are recognised
- And: the name can be changed by hand before or after it is filled in, and is written to the file as `[test: <name> : <url> ]`

## the test name is the file name @v1 [proposed]

- When: a `[test:<url>]` url ends in a file with an extension, a `#L<n>` line anchor or a query string
- Then: the display name is the file name with the extension, anchor and query dropped
- And: a url with no path is displayed as the url itself

## trace a test to a rule @v1 [proposed]

- When: a rule inside a scenario body carries `[test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]`
- Then: the scenario card and the editor preview show that rule with a link reading `QuoteHomeownersPolicyTest`
- And: the link opens the full url, line anchor included, in a new tab, and the tooltip shows the full url
