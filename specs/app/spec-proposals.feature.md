# spec-proposals

We don't accept pull requests (app/open-source-license.feature.md); people
propose specs instead (roadmap/moving-to-proper-open-source.md). The spec
proposals service is a separate mode of the app (`SPECDRIVEN_PROPOSALS=1`) run
as its own Railway service: anyone signed in with GitHub proposes a scenario on
an existing feature, or a new feature with its first scenario, into one spec
repository — this one by default. A proposal is committed straight to the
repository's default branch as `@v1 [proposed]`, pushed with a bot token,
authored by the visitor; we decide which proposals to publish and build.

Nothing else can change anything there: no editing, publishing, deleting,
settings, plugins or agents. The site reads the repository's feature files to
anyone who opens it, so it is pointed only at a repository that is public.

## A daily allowance per GitHub account @v1 [proposed]

- Given: the service allows 5 proposals a day per account (the default)
- When: I have made 5 today
- Then: my next proposal is refused until 00:00 UTC, and the page shows how
  many I have left
- and a refused proposal does not use my allowance
- and an account with write access to the spec repository has no allowance
- and the service as a whole takes at most 100 a day (the default), whoever
  proposes

[test: an account has a daily allowance, and a refused proposal does not use it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L471 ]
[test: contributors have no per-account limit, but the service's daily cap holds for everyone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L485 ]
[test: the day a quota counts is the UTC day : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L147 ]
[test: a spent allowance is shown as such and stops proposing; a contributor has none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L47 ]
[test: signed in, it shows the login, what is left of today's allowance, and Sign out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L39 ]
[test: the scenario form disables itself when proposing is not allowed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L58 ]

## Everything else is refused @v1 [proposed]

- Given: the proposals service
- When: anything calls the app's own API — saving, publishing, deleting,
  settings, sign-in, agents, plugins, logs
- Then: it answers 403
- and reading a feature serves the file alone, never the drafts its document
  keeps

[test: everything else is refused, %s %s → 403 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L317 ]
[test: anyone can read the spec tree and a feature file — without the documents' drafts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L347 ]

## Proposals at the same time all land @v1 [proposed]

- Given: someone pushed to the spec repository after the service last synced
- When: two visitors propose at the same moment
- Then: both proposals and the other push are all on the default branch
- and a push the remote rejects is retried from the remote's latest commit

[test: a push someone made directly, then two proposals at once, and all three land : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L526 ]

## Proposals only add @v1 [proposed]

- Given: a feature with published and proposed scenarios
- When: anyone proposes
- Then: the server builds the markdown itself from a name and a text, and
  every existing scenario, the feature's name and its description are left
  exactly as they were
- and a text with a markdown heading, an authorship, ticket or test line, or
  a name the feature already has is refused, and nothing reaches the remote
- and so is a scenario with no name or no text, a name over 120
  characters or a text over 8 KB
- and a proposal is never published

[test: appending keeps every existing scenario and adds the new one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L82 ]
[test: the append-only check catches a changed, removed or published scenario : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L118 ]
[test: a body that would sign the scenario as someone else is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L99 ]
[test: a body that would add a ticket or a test link is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L109 ]
[test: a name the feature already has is refused, whatever its case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L95 ]
[test: a scenario for a feature that is not there is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L436 ]
[test: refuses %s : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L65 ]
[test: refused, and nothing reaches the remote — %s : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L424 ]

## Propose a new feature @v1 [proposed]

- Given: I am signed in
- When: I choose an existing folder, give a feature name, an optional
  description and a first scenario, and propose
- Then: a new feature file holding them is on the default branch, its
  scenario `@v1 [proposed]`
- and a folder that does not exist, a feature name that does, or a name with
  a slash is refused

[test: a new feature lands in an existing folder with its first scenario : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L442 ]
[test: a new feature in a folder that does not exist, or with a name that does, is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L464 ]
[test: a new feature holds its name, description and one proposed scenario : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L134 ]
[test: a new feature name with a slash, or no letters, is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L140 ]

## Propose a scenario on an existing feature @v1 [proposed]

- Given: I am signed in and reading a feature
- When: I give a scenario name and its Given / When / Then steps and propose
- Then: the scenario is on the repository's default branch as
  `## <name> @v1 [proposed]`, after the feature's other scenarios
- and the commit is authored by my GitHub account, as its noreply address,
  with a `Proposed-by: @<login>` trailer
- and the feature's Automerge document travels with the file and records me
  as the scenario's creator

[test: a proposed scenario lands on main as proposed, authored by the visitor and signed in its document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L387 ]
[test: a proposed scenario is always @v1 and proposed, with its body tidied : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L60 ]

## Sign in with GitHub to propose @v1 [proposed]

- Given: I open the proposals site signed out
- Then: I can read every feature and its scenarios, grouped by folder,
  with a filter that narrows the list
- and proposing asks me to sign in with GitHub
- When: I sign in
- Then: GitHub's browser flow runs with PKCE and a state nonce bound to my
  browser, and I come back signed in, the page showing my login and Sign
  out
- and my GitHub token is used once, to read who I am, and never stored
- and a callback that is not my browser's sign-in signs nobody in
- and a sign-in that comes back without a session — cancelled, expired or
  failed — says why

[test: signed out, the session says so and proposing asks for a sign-in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L338 ]
[test: sign-in goes to GitHub with PKCE and back with a session; no token is kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L373 ]
[test: a callback that is not this browser's sign-in is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L360 ]
[test: signing out ends the session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L499 ]
[test: signed out, it offers Sign in with GitHub : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L32 ]
[test: signed in, it shows the login, what is left of today's allowance, and Sign out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L39 ]
[test: a sign-in that came back without a session says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L85 ]
[test: features group by folder, in order, and the filter narrows them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L72 ]

## The service is configured on Railway @v1 [proposed]

- Given: a Railway service built from the app's repository
- When: it sets `SPECDRIVEN_PROPOSALS=1`, the bot token, the sign-in GitHub
  App's client id and secret, and `PUBLIC_URL`
- Then: it runs the proposals service, answering to that domain only, as an
  unprivileged user with its data on the volume
- and without the bot token, a working one, or `PUBLIC_URL` it refuses to
  start

[test: the bot token and the public url are required; quotas must be whole numbers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L174 ]
[test: the defaults are SpecDriven/specs-specdriven, 5 a day per account, 100 a day in all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L162 ]
[test: on the hosted image the service answers to its public domain only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L181 ]
[test: proposals mode is SPECDRIVEN_PROPOSALS=1 or true : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L156 ]

## The site links to its source @v1 [proposed]

- Given: the proposals site
- Then: its footer says the app is AGPL-3.0 and links to the source of the
  running version (app/open-source-license.feature.md)
- and says where proposals go and that we don't accept pull requests

[test: the footer links the spec repo and the source code under the AGPL : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposalsApp.test.tsx#L64 ]
