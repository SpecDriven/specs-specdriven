# open-source-license

The app is free software under the GNU Affero General Public License,
version 3 only (`AGPL-3.0-only`), see
roadmap/moving-to-proper-open-source.md. The AGPL's §13 asks a copy that
users reach over a network — the public demo, the hosted service — to offer
those users its source, so every copy links to the source of the version it
runs.

We do not accept pull requests: merging outside code is a security
liability, and reviewing it is a cost we can't carry. Users propose specs
instead (CONTRIBUTING.md in the code repo says how).

## The app is AGPL-3.0-only @v1 [proposed]

- Given: the code repo
- Then: LICENSE.md holds the GNU AGPL version 3 text
- and NOTICE.md names the copyright holder and says "version 3 only"
- and package.json names the license by its SPDX id, `AGPL-3.0-only`
- and the desktop app ships LICENSE.md and NOTICE.md

[test: package.json names the license by its SPDX id : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L15 ]
[test: LICENSE.md is the GNU AGPL version 3 and NOTICE.md says version 3 only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L19 ]
[test: the desktop app ships the license and the notice : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L29 ]

## The app links to the source of the running version @v1 [proposed]

- Given: the app is running, on the desktop, in the public demo or hosted
- When: I open the user menu
- Then: it offers "Source code (AGPL-3.0)"
- and that opens the release of the running version on
  SpecDriven/specdriven-app-release, whose source zip is that version's
  source
- and a deployment running modified code sets `SPECDRIVEN_SOURCE_URL`, and
  the link opens that instead

[test: the source link names the release of the running version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L34 ]
[test: a deployment running modified code names its own source : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L40 ]
[test: the user menu offers Source code, which opens the source url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/license.test.ts#L47 ]
[test: state carries the source url of the running version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demo.test.ts#L363 ]

## Pull requests are not accepted @v1 [proposed]

- Given: someone without write access opens a pull request from a fork
- Then: it is closed automatically with a comment that points to
  CONTRIBUTING.md and to proposing specs instead
- and pull requests from the team's own branches are left open
- and the workflow that closes them never checks out or runs the pull
  request's code
