# Test Traceability

Trace tests to indivitual specs (scenarios / rules).

In '.md' file store tests as [test:<url to test>]

Ex: [test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]

Internal TS interface RenderTestInEditor will format the test for user display.

Display test as QuoteHomeownersPolicyTest .

## a reference inside code is code @v1 [proposed]

- When: `[test:<url>]` sits inside backticks or a fenced code block
- Then: it is shown as code, not as a test link

## an unsafe or malformed reference is left as text @v1 [proposed]

- When: the url in `[test:<url>]` is not an http or https url, or the token is followed by `(` like an ordinary markdown link
- Then: it is shown as it was typed — an ordinary markdown link keeps working, and `[test:javascript:…]` never becomes a link

## the reference stays in the feature file @v1 [proposed]

- When: a scenario with a `[test:<url>]` reference is saved, proposed or published
- Then: the token is written back verbatim where it stood in the body, and is not moved into the `### Tests` table
- And: GitHub renders it as the plain text `[test:<url>]`

## the test name is the file name @v1 [proposed]

- When: a `[test:<url>]` url ends in a file with an extension, a `#L<n>` line anchor or a query string
- Then: the display name is the file name with the extension, anchor and query dropped
- And: a url with no path is displayed as the url itself

## trace a test to a rule @v1 [proposed]

- When: a rule inside a scenario body carries `[test:https://github.com/SpecDriven/insurance-cap-java/blob/main/srv/src/test/java/com/acme/insurance/policy/QuoteHomeownersPolicyTest.java#L32]`
- Then: the scenario card and the editor preview show that rule with a link reading `QuoteHomeownersPolicyTest`
- And: the link opens the full url, line anchor included, in a new tab, and the tooltip shows the full url
