# open-source-license

The app is open source under the Apache License 2.0 (`Apache-2.0`), see
roadmap/moving-to-proper-open-source.md. Every copy — the desktop app, the
public demo, the hosted service — links to the source of the version it
runs.

We do not accept pull requests: merging outside code is a security
liability, and reviewing it is a cost we can't carry. Users propose specs
instead (CONTRIBUTING.md in the code repo says how).

## S1: Pull requests are not accepted @v1 [proposed]

- Given: someone without write access opens a pull request from a fork
- Then: it is closed automatically with a comment that points to
  CONTRIBUTING.md and to proposing specs instead
- and pull requests from the team's own branches are left open
- and the workflow that closes them never checks out or runs the pull
  request's code

[test: a pull request from a fork by someone without write access is closed, pointing to CONTRIBUTING.md and to proposing specs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L93 ]
[test: pull requests from the team's own branches, or by people with write access, are left open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L106 ]
[test: the close workflow never checks out or runs the pull request's code : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L118 ]

## S2: The app is Apache-2.0 @v1 [proposed]

- Given: the code repo
- Then: LICENSE.md holds the Apache License 2.0 text
- and NOTICE.md names the copyright holder and the license
- and package.json names the license by its SPDX id, `Apache-2.0`
- and the desktop app ships LICENSE.md and NOTICE.md

[test: package.json names the license by its SPDX id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L17 ]
[test: LICENSE.md is the Apache License 2.0 and NOTICE.md names the copyright holder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L21 ]
[test: the desktop app ships the license and the notice : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L31 ]

## S3: The app links to the source of the running version @v1 [proposed]

- Given: the app is running, on the desktop, in the public demo or hosted
- When: I open the user menu
- Then: it offers "Source code (Apache-2.0)"
- and that opens the release of the running version on
  SpecDriven/specdriven-app-release, whose source zip is that version's
  source
- and a deployment running modified code sets `SPECDRIVEN_SOURCE_URL`, and
  the link opens that instead
- and until the app knows that url, the menu has no Source code item

[test: the source link names the release of the running version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L36 ]
[test: a deployment running modified code names its own source : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L42 ]
[test: the user menu offers Source code, which opens the source url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L49 ]
[test: state carries the source url of the running version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L363 ]
[test: no Source code item before the state names the url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L59 ]
