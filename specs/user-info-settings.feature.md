# User Info

- lower left corner of the screen should show the current user
- by default this is the user on the OS
- User Menu will have user info, settings and plugins options

The lower left corner of the sidebar shows a user chip: an avatar circle with
the user's initials and the user name. The name is the GitHub account when
one is known — the signed-in login, or the GitHub CLI's account when that
engine is in effect — and falls back to the OS user otherwise; the server
reports both via `/api/state`.

- Clicking the chip opens the User Menu just above it, with three options:
  "User info", "Settings" and "Plugins".
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
  (user.name / user.email from git config). Missing values show "not
  set"; no GitHub account shows "not signed in".
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

## open Plugins @v1 [proposed]

- When: the user picks "Plugins" from the user menu, presses `g p` or
  picks Plugins in the command palette
- Then: the Plugins page takes the editor pane at `/plugins`, with the
  plugins that are on in their sidebar order, the ones switched off, the
  plugin folders and the marketplaces (app/plugins.feature.md)
- and Settings is no longer where the plugins are managed

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]

## open Settings @v1 [proposed]

- When: the user picks "Settings" from the user menu or presses `⌘,`
- Then: the Settings page takes the editor pane at `/settings`, its
  sections — General, Sync, Specs, Project management, Workflow, Git &
  GitHub — listed on the left
- and every control saves itself the moment it changes, as it did in the
  dialog
- and the ✕ or ESC goes back to the page that was open before

[test: Settings and Plugins have urls that resolve back to them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L328 ]
[test: anything else is not an app page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L340 ]

## open the user menu @v1 [proposed]

- When: the user clicks the chip
- Then: a menu appears with "User info", "Settings" and "Plugins"
- and ESC or clicking anywhere else closes it

## see the current user @v1 [proposed]

- When: the app is open and no GitHub account is known
- Then: the lower left corner shows the OS user behind a small avatar chip

## see the GitHub account @v1 [proposed]

- When: a GitHub account is known
- Then: the chip shows that login instead of the OS user

## sign in from settings @v1 [proposed]

- When: the user clicks "Sign in with GitHub" in Settings
- Then: the page shows the one-time code and a link to
  github.com/login/device
- and once the user authorizes there, the page shows
  "Signed in as <login>"

## view user info @v1 [proposed]

- When: the user picks "User info" from the menu
- Then: a dialog shows the OS user, the GitHub account, and the name and
  email used for commits to the data repo
