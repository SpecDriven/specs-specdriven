# User Info

- lower left corner of the screen should show the current user
- by default this is the user on the OS
- User Menu will have user info, settings and plugins options
- in the desktop app, the User Menu also signs in with a GitHub token and signs out

The lower left corner of the sidebar shows a user chip: an avatar circle with
the user's initials and the user name, with the running app's version (e.g.
"v0.1.209") muted at the right of the name. The name is the GitHub account when
one is known — the signed-in login, or the GitHub CLI's account when that
engine is in effect — and falls back to the OS user otherwise; the server
reports both via `/api/state`.

- Clicking the chip opens the User Menu just above it, with "User info",
  "Settings" and "Plugins". In the desktop and local app it also offers the
  GitHub sign-in: "Sign in with token…" while signed out; "Switch account…"
  and "Sign out" while signed in. "Sign out" is also offered while the
  GitHub CLI's account stands in for a missing sign-in, and stops the app
  using it. The hosted app keeps its own "Account &
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
- Initials come from the identity name when set, otherwise from the OS
  user name.

## Git engine setting

- The Settings page offers the Git engine: `Automatic — GitHub CLI when
  installed (recommended)`, `GitHub CLI (gh)`, `System git` or
  `Built-in (no git needed)` (git-operations.feature.md)
- each option is disabled when its binary is not installed; the built-in
  engine ships with the app, so it is always offered
- The Settings page holds the GitHub account section: it reports what
  the GitHub CLI is signed in as, and "Sign in with GitHub" starts the
  device flow; signed in, it shows the account and a "Sign out" button
- Signing out also stops the app using the GitHub CLI's account: while the
  app has no sign-in of its own it would otherwise fall back to gh's, and
  "signed out" would still push and author commits as gh's user. The
  section then says gh is "not used since you signed out" and offers "Use
  GitHub CLI account" to turn the fallback back on (the `useGhAccount`
  setting, absent meaning on)
- Known limit: the app never edits the machine's git config, so a
  credential helper `gh auth setup-git` wrote there still answers for
  system git after signing out

## S1: open Plugins @v1 [proposed]

- When: the user picks "Plugins" from the user menu, presses `g p` or
  picks Plugins in the command palette
- Then: the Plugins page takes the editor pane at `/plugins`, with the
  plugins that are on in their sidebar order, the ones switched off, the
  plugin folders and the marketplaces (app/plugins.feature.md)
- and Settings is no longer where the plugins are managed

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]

## S2: open Settings @v1 [proposed]

- When: the user picks "Settings" from the user menu or presses `⌘,`
- Then: the Settings page takes the editor pane at `/settings`, its
  sections — General, Sync, Specs, Project management, Workflow, Git &
  GitHub — listed on the left
- and every control saves itself the moment it changes, as it did in the
  dialog
- and the ✕ or ESC goes back to the page that was open before

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]
[test: anything else is not an app page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L350 ]

## S3: open the user menu @v1 [proposed]

- When: the user clicks the chip
- Then: a menu appears with "User info", "Settings" and "Plugins", and in
  the desktop app the GitHub sign-in entries
- and ESC or clicking anywhere else closes it

## S4: see the current user @v1 [proposed]

- When: the app is open and no GitHub account is known
- Then: the lower left corner shows the OS user behind a small avatar chip

## S5: see the GitHub account @v1 [proposed]

- When: a GitHub account is known
- Then: the chip shows that login instead of the OS user

## S6: sign in from settings @v1 [proposed]

- When: the user clicks "Sign in with GitHub" in Settings
- Then: the page shows the one-time code and a link to
  github.com/login/device
- and once the user authorizes there, the page shows
  "Signed in as <login>"

## S7: sign in with a token from the user menu @v1 [proposed]

- When: the user picks "Sign in with token…" from the user menu (or
  "Switch account…" while signed in)
- Then: a dialog asks for the GitHub host — github.com unless changed, a
  GitHub Enterprise host otherwise — and a personal access token
- and signed in already, it names the account the token would replace
- and once the host's /user accepts the token, the dialog closes and the
  chip shows the new login
- and a token the host rejects shows "GitHub did not accept this token —
  check it and its expiry" and keeps the previous sign-in
  (ui/app-start.feature.md "a rejected token is not kept")

[test: signed out, a modal dialog with the host and token fields : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tokenSignInDialog.test.tsx#L14 ]
[test: signed in, it names the account the token would replace : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tokenSignInDialog.test.tsx#L25 ]

## S8: sign out from the user menu @v1 [proposed]

- When: the user picks "Sign out" from the user menu — offered while the
  app is signed in, or while the GitHub CLI's account stands in for it
- Then: the app forgets its GitHub token and stops using gh's account
- and the chip shows the OS user again

[test: sign out clears the token and stops using the GitHub CLI account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L68 ]

## S9: signing out stops using the GitHub CLI account @v1 [proposed]

- Given: the GitHub CLI is signed in on this computer
- When: the user signs out of the app
- Then: the app no longer falls back to gh's account — the chip, commit
  authorship and Issues tab requests stay signed out
- and Settings says gh is "not used since you signed out", with a "Use
  GitHub CLI account" button that turns the fallback back on

[test: signed out with gh signed in, the gh account stands in until sign out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L62 ]
[test: sign out clears the token and stops using the GitHub CLI account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L68 ]
[test: Use GitHub CLI account turns the fallback back on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/signOut.test.ts#L85 ]

## S10: view user info @v1 [proposed]

- When: the user picks "User info" from the menu
- Then: a dialog shows the OS user, the GitHub account, and the name and
  email used for commits to the data repo

[test: signed out, the identity is git's as resolved in the data repo, not the app's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/user.test.ts#L60 ]
