# Spec discovery

A coding session should not end with behaviour the specifications do not
describe. At the end of a Claude Code turn in the app repo, the code the
branch added is read, traced into this spec repo, and whatever no scenario
describes is handed back to the agent as specs to write — while it still
knows what the code does.

Discovery is mechanical; wording a scenario is not. The hook
(`scripts/spec-discovery.ts`, registered in `.claude/settings.json`
beside the test-links hook of external-links/test-traceability.feature.md)
finds the gap and names it; the agent writes the feature or the scenario,
the way AGENTS.md says.

## S1: new behaviour is what the branch exports and serves @v1 [proposed]

- When: a branch changes files under `src/`
- Then: the behaviour it added is the names it newly exports — functions,
  classes and constants, never types or interfaces — the `/api` routes it
  newly serves, and the source files it added
- And: a file a diff only moved lines in, and a new file under fifteen lines
  that exports nothing, add no behaviour and are passed over
- And: files git does not track yet count as new, every line of them added,
  and `.d.ts` files and files outside `src/` are not source at all

[test: only source files count, each with the lines it gained and whether it is new : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L42 ]
[test: a line of a file that reads like a diff header is content, not a header : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L49 ]
[test: files git does not track yet are new files, every line of them added : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L62 ]
[test: the surface is what the branch newly exports and serves, never a type : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L68 ]
[test: a new file of substance that exports nothing is a gap of its own; a small one is plumbing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L227 ]

## S2: behaviour is traced to a scenario through the tests @v1 [proposed]

- When: the branch adds an export or a route
- Then: it is described when a scenario's `[test: … ]` link names a test
  file that exercises it — one that names the export as a whole word, or
  calls the route by its path up to its first parameter
- And: it is a gap when no test exercises it at all, and a gap too when the
  only tests that do are linked from no scenario, since there may be no
  scenario to link them to yet
- And: a new file nothing reaches is a gap in its own right
- And: the features whose tests already reach the file are offered as the
  place a new scenario might belong, the most closely linked first

[test: an export is named as a whole word : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L90 ]
[test: a route is called by its path, up to its first parameter : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L96 ]
[test: only relative imports are followed, and only to a source file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L103 ]
[test: behaviour a scenario's test already exercises is described, and said nothing about : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L186 ]
[test: a test no scenario links is named, as the one to link once the scenario exists : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L218 ]

## S3: a turn ends with the specs caught up @v1 [proposed]

- When: a Claude Code turn ends with behaviour on the branch that no
  scenario describes
- Then: the agent is kept working once, with each gap named — the file, the
  export or route, the tests that exercise it, and the features it might
  belong in — and told to write the specs into this repo before finishing
- And: a change a user could never notice — a refactor, an internal helper,
  a build detail — needs no spec: the agent says which, and why, and stops
- And: if the agent stops again with them still open, it is shown as a
  warning and the turn ends; a turn that leaves nothing undescribed ends
  quietly
- And: `bun scripts/spec-discovery.ts` runs the same search by hand —
  `--base <ref>` compares against another commit, `--json` reports it as
  data — and `SPECDRIVEN_SPEC_DISCOVERY=off` silences the hook for a session

[test: a turn that adds no behaviour ends quietly : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L180 ]
[test: an export and a route no test exercises are gaps, and the turn is blocked once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L193 ]
[test: the hook stays out of the way when it is switched off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/specDiscovery.test.ts#L234 ]
