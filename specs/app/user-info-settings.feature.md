# User Info

**Created by:** vydra

**Updated by:** vydra

- lower left corner of the screen should show the current user
- by default this is the user on the OS
- User Menu will have user info, settings and plugins options
- in the desktop app, the User Menu also switches the GitHub account and logs out

The lower left corner of the sidebar shows a user chip: an avatar circle with
the user's initials and the user name, with the running app's version (e.g.
"v0.1.209") muted at the right of the name. The name is the GitHub account when
one is known — the GitHub CLI's account when that is the Git access, else
the app's own sign-in — and falls back to the OS user otherwise; the server
reports both via `/api/state`.

- Clicking the chip opens the User Menu just above it, with "User info",
  "Settings" and "Plugins", and on every copy "What's new", the release
  notes (app/release-notes.feature.md). In the desktop and local app it also offers the
  "Switch account…" while signed in with a token, and "Log out" always.
  Log out forgets the app's GitHub token, closes the open folder and goes
  back to the first-run screen, where the user opens a folder or clones
  from GitHub with a token (ui/app-start.feature.md). It leaves the GitHub
  CLI alone: that is a Git access of its own, chosen in Settings. The
  menu has no "Sign in with token…": signing in is the first-run
  screen's, and Settings'. The hosted app keeps its own "Account &
  tokens" and "Sign out" (app/hosted-app.feature.md); the demo has neither.
- Settings and Plugins are pages, not dialogs: each takes the whole editor
  pane, the sidebar stays beside it, and each has its own url (`/settings`,
  `/plugins`), so Back leaves it and a reload or a shared link reopens it.
  A dialog was too small for them.
- Each page lists its sections on the left; picking one scrolls to it, and
  the list follows the scroll. The ✕ in the page header or ESC goes back to
  what was open before; each page links to the other.
- ESC or a click elsewhere closes the menu.
- "User info" opens a dialog showing the OS user, the signed-in GitHub
  account, and the name/email that author the app's commits to the data
  repo — the GitHub identity when signed in
  (git-operations.feature.md), else the system git identity
  (user.name / user.email from git config, read in the data repo's
  folder — where the commits are made — so the repo's own config and an
  `includeIf "gitdir:…"` rule apply, not just the global one). Missing
  values show "not set"; no GitHub account shows "not signed in".
- User info also names the settings file, `~/.specdriven/settings.json`,
  with "View settings file…": a wider, read-only dialog showing that file
  — always the home one — in readable form rather than JSON: grouped
  sections (Folders, Git & GitHub, Sync, Specs, Integrations, Agents,
  Plugins, Logging, Other) with the Settings page's labels, switches as
  On / Off, intervals in seconds, folders as lists written from `~`, and
  keys the app no longer reads marked "no longer used". The file holds no
  secrets — tokens are kept apart from it. The hosted app has no such row.
- Initials come from the identity name when set, otherwise from the OS
  user name.

## Git access

- Settings → Git & GitHub asks where git gets its GitHub credentials, as a
  "Git access" radio group: `GitHub CLI (gh)`, `GitHub Personal Access
  Token`, or `GitHub account sign-in` — the last only while the "GitHub
  account" switch is on (below)
- A note above the radios says the GitHub CLI (gh) is recommended, and its
  radio carries a "Recommended" badge; without gh installed the radio is
  disabled and points at cli.github.com
- With nothing chosen, the GitHub CLI is the access wherever gh is
  installed, else the personal access token
- Under the radios is the chosen access's own panel: the GitHub CLI's
  account (or that gh is signed out — run `gh auth login`, and `gh auth
  login --hostname <host>` for GitHub Enterprise); the token's paste form,
  who is signed in and "Sign out"; or the account sign-in's "Connect to
  GitHub" and device code
- The access picks the engine (git-operations.md): the GitHub CLI runs the
  gh engine, with gh's credential helper for every host gh is logged in to,
  GitHub Enterprise ones too; the token and the account sign-in run system
  git, or the built-in engine where git is not installed. The engine is no
  longer a choice of its own in Settings
- The access is exclusive: with the GitHub CLI chosen, the app's own token
  is not used — for git, the user chip, commit authorship or the Issues
  tab — and with either app sign-in chosen, gh's account is not used
- A clone, pull or push that fails for want of a credential names the way
  in the chosen access offers: `gh auth login` for the GitHub CLI, adding a
  personal access token in Settings for the token, signing in with GitHub
  in Settings for the account sign-in
- Known limit: the app never edits the machine's git config, so a
  credential helper `gh auth setup-git` wrote there still answers for
  system git under the other accesses
- Known limit: the commit author on the gh engine is gh's github.com
  account, even for a GitHub Enterprise repo

## GitHub account behind a switch

- The browser and device-code sign-in are not ready yet, so they sit
  behind a "GitHub account" switch in Settings → Git & GitHub, off unless
  turned on (the `githubAccount` setting, absent meaning off)
- On, "GitHub account sign-in" joins the Git access radios; off, it is not
  offered, and an access left on it falls back to the personal access
  token until the switch comes back on
- The personal access token is not behind the switch; the user menu always
  offers "Log out", and "Switch account…" while signed in with a token
- The hosted app's "Account & tokens" is its own and is not behind the
  switch

## S1: open Plugins @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user picks "Plugins" from the user menu, presses `g p` or
  picks Plugins in the command palette
- Then: the Plugins page takes the editor pane at `/plugins`, with the
  plugins that are on in their sidebar order, the ones switched off, the
  plugin folders and the marketplaces (app/plugins.feature.md)
- and Settings is no longer where the plugins are managed

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]

## S2: open Settings @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user picks "Settings" from the user menu or presses `⌘,`
- Then: the Settings page takes the editor pane at `/settings`, its
  sections — General, Git & GitHub, Sync, Specs, Project management,
  Workflow — listed on the left
- and every control saves itself the moment it changes, as it did in the
  dialog
- and the ✕ or ESC goes back to the page that was open before

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]
[test: anything else is not an app page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L355 ]

## S3: open the user menu @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user clicks the chip
- Then: a menu appears with "User info", "Settings" and "Plugins", and in
  the desktop app "Log out" (and "Switch account…" while signed in)
- and ESC or clicking anywhere else closes it

## S4: see the current user @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the app is open and no GitHub account is known
- Then: the lower left corner shows the OS user behind a small avatar chip

## S5: see the GitHub account @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: a GitHub account is known
- Then: the chip shows that login instead of the OS user

## S6: sign in from settings @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user clicks "Sign in with GitHub" in Settings
- Then: the page shows the one-time code and a link to
  github.com/login/device
- and once the user authorizes there, the page shows
  "Signed in as <login>"

## S7: switch account with a token from the user menu @v1 [proposed]

- When: the user picks "Switch account…" from the user menu, offered
  while signed in with a token
- Then: a dialog asks for the GitHub host — github.com unless changed, a
  GitHub Enterprise host otherwise — and a personal access token
- and signed in already, it names the account the token would replace
- and once the host's /user accepts the token, the dialog closes and the
  chip shows the new login
- and a token the host rejects shows "GitHub did not accept this token —
  check it and its expiry" and keeps the previous sign-in
  (ui/app-start.feature.md "a rejected token is not kept")

[test: signed out, a modal dialog with the host and token fields : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tokenSignInDialog.test.tsx#L14 ]
[test: signed in, it names the account the token would replace : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tokenSignInDialog.test.tsx#L27 ]

## S8: sign out of the token in Settings @v1 [proposed]

- When: the user clicks "Sign out" under the personal access token in
  Settings → Git & GitHub — offered while the app is signed in
- Then: the app forgets its GitHub token
- and the chip shows the OS user again, unless the GitHub CLI is the Git
  access
- and the GitHub CLI is left alone: nothing about it changes

[test: sign out clears the app's token and leaves the GitHub CLI alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L74 ]

## S9: the GitHub CLI is a Git access of its own @v1 [proposed]

- Given: the GitHub CLI is installed and signed in on this computer
- When: the GitHub CLI is the Git access, as it is by default
- Then: gh's account is the one in use — the chip, commit authorship and
  the Issues tab's requests
- and the app's own token, if one was pasted, is not used
- and picking the personal access token instead leaves gh's account out

[test: with the GitHub CLI as the Git access, gh's account is the one in use : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L61 ]
[test: the app's own token is not used while the GitHub CLI is the Git access : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L69 ]

## S10: view user info @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user picks "User info" from the menu
- Then: a dialog shows the OS user, the GitHub account, and the name and
  email used for commits to the data repo

[test: signed out, the identity is git's as resolved in the data repo, not the app's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/user.test.ts#L60 ]

## S11: with the GitHub account switch off, the account sign-in is not offered @v1 [proposed]

- Given: the "GitHub account" switch in Settings is off, as it is by
  default
- When: the user opens Settings → Git & GitHub
- Then: the Git access radios offer the GitHub CLI and the personal access
  token, and no "GitHub account sign-in"
- and the user menu offers "Log out" (and "Switch account…" once signed
  in with a token)
- and turning the switch on adds "GitHub account sign-in" as the third
  radio; turning it off again while that is chosen falls back to the token

[test: the GitHub account setting is off by default and round-trips through the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/githubAccountSetting.test.ts#L47 ]
[test: signed out, it asks for a token, not as an alternative : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tokenSignInDialog.test.tsx#L45 ]
[test: with the GitHub account switch on, the account sign-in is the third radio : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccessChoice.test.tsx#L25 ]
[test: the GitHub account access needs its switch on, and falls back to the token when it goes off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccess.test.ts#L87 ]

## S12: pick the Git access @v1 [proposed]

- Given: the user opens Settings → Git & GitHub
- Then: "Git access" offers radios for the GitHub CLI (gh) and the GitHub
  Personal Access Token, under a note that the GitHub CLI is recommended
- and with nothing chosen and gh installed, the GitHub CLI is selected
- and without gh installed its radio is disabled and points at
  cli.github.com, and the token is selected
- When: the user picks a radio
- Then: the app uses that access from the next git operation on — the gh
  engine for the GitHub CLI, system git (or the built-in engine without
  git) for the token
- and a failed sign-in names the way in that access offers
- and with the GitHub CLI, a github.com repository is cloned from
  github.com even when gh's default host is a GitHub Enterprise one

[test: the GitHub CLI and the token are radios, with a note that the GitHub CLI is recommended : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccessChoice.test.tsx#L12 ]
[test: without gh installed its radio is disabled and says where to get it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccessChoice.test.tsx#L35 ]
[test: the radios show the GitHub CLI by default where it is installed, else the token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccessChoice.test.tsx#L43 ]
[test: with gh installed and nothing chosen, the GitHub CLI is the Git access : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccess.test.ts#L65 ]
[test: picking a Git access sets the engine it runs on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccess.test.ts#L70 ]
[test: a failed sign-in names the way in the chosen Git access offers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitAccess.test.ts#L106 ]
[test: gh's credentials answer for every host it is logged in to, GitHub Enterprise too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L99 ]
[test: the gh engine names github.com in the clone, whatever gh's default host is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gitEngine.test.ts#L124 ]

## S13: view the settings file @v1 [proposed]

- When: the user opens "User info" from the user menu and clicks "View
  settings file…"
- Then: a dialog shows the path ~/.specdriven/settings.json and the
  file's contents as labelled sections, not JSON — switches On or Off,
  intervals in seconds, engines and accesses in words, recent folders as a
  list written from ~
- and keys the app no longer reads are marked "no longer used"
- and it is the home file even when a project settings file is the one in
  effect; the dialog then names that file
- and without a home file the dialog says there is none yet
- and nothing in the dialog changes a setting
- and "Copy contents" copies what the dialog shows to the clipboard as
  plain text — the same labels and values, sections as headings, nested
  values indented — not the JSON

[test: shows ~/.specdriven/settings.json as labelled sections, not JSON : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileDialog.test.tsx#L24 ]
[test: switches read On and Off, intervals in seconds, engines and accesses in words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L23 ]
[test: recent folders are a list of paths from ~ with their kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L52 ]
[test: a key the app no longer reads is marked deprecated : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L85 ]
[test: shows the home file even when a project settings file is the one in effect : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L51 ]
[test: without the file, it says there is none yet : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileDialog.test.tsx#L47 ]
[test: names the file the app reads when a project settings file is in effect : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileDialog.test.tsx#L55 ]
[test: a file that is not JSON says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileDialog.test.tsx#L62 ]
[test: reads the home settings file as written, without upgrading it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L40 ]
[test: with no home file it says so, and does not fall back to the project file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L61 ]
[test: a file that is not JSON is reported, not thrown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L69 ]
[test: is a GET route : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L78 ]
[test: answers with the home file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L83 ]
[test: the hosted app refuses it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileRoute.test.ts#L93 ]
[test: known keys land in their section under the Settings page's label, sections in the app's order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L13 ]
[test: an autosync frequency of 0 reads Off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L47 ]
[test: the plugin profile names each folder's enabled plugins on one line : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L68 ]
[test: an unknown key lands under Other with a readable label, nested objects as rows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L91 ]
[test: plugin settings keep each plugin's id as its label : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L106 ]
[test: an empty file has no sections : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L120 ]
[test: tildePath writes paths under home from ~ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L126 ]
[test: humanizeKey turns camelCase into words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L132 ]
[test: offers Copy contents beside Close : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileDialog.test.tsx#L40 ]
[test: copies the view as plain text, sections as headings and nested rows indented : https://github.com/SpecDriven/specdriven-app/blob/main/tests/settingsFileView.test.ts#L139 ]

## S14: log out from the user menu @v1 [proposed]

- Given: the desktop or local app, with a folder open
- When: the user picks "Log out" from the user menu
- Then: what was saved is synced first, and then the app forgets its
  GitHub token
- and the open folder — a specifications folder or a code repository —
  is closed, so the app shows the first-run screen: "Open a folder on this
  computer" or "Clone from GitHub" with a personal access token
  (ui/app-start.feature.md "first run offers a folder or a clone")
- and a reload shows the first-run screen too
- and the recent folders stay listed, and nothing on disk is removed
- and the GitHub CLI is left alone: nothing about it changes
- and the user menu has no "Sign in with token…"

[test: signed out, the user menu offers Log out and no token sign-in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/accountMenu.test.ts#L7 ]
[test: signed in, it offers Switch account and Log out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/accountMenu.test.ts#L11 ]
[test: the hosted app and the demo offer neither : https://github.com/SpecDriven/specdriven-app/blob/main/tests/accountMenu.test.ts#L15 ]
[test: log out clears the app's token and closes the open folder, keeping the folder lists : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L92 ]
[test: log out closes an open code repository too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L112 ]
