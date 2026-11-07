# GitHub Issues

**Assigned:** David

**Created by:** vydra

**Updated by:** vydra

# GitHub Issue (bug) tracking

**Assigned:** David

Add a Tab for 'Issues' to the right of 'Tasks'.
Use 'code-repo' in project-settings.json to find repo that holds the GH Issues.

Allow to access GitHub issues from the app.

## S1: Close an issue @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: an open issue is open in the pane
- When: the user presses "Close issue" and confirms
- Then: the issue is closed on GitHub as completed; the ⋯ menu also offers
  "Close as not planned", which closes it with that reason
- and the pane shows the closed badge (a check for completed, a slash for not
  planned) and when it was closed; the issue leaves the open list and appears
  under Closed
- and "Reopen" — on a closed issue's pane, or in its ⋯ menu — reopens it on
  GitHub and it returns to the open list

## S2: Comment on an issue @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: an issue is open in the pane
- When: the user looks below the description
- Then: the issue's comments are listed, each with its author, when it was
  written (linking to it on GitHub) and its Markdown rendered, under a
  "N comments" heading
- and a Markdown field at the end with a "Comment" button posts a new comment
  to GitHub and appends it; a blank comment cannot be posted

## S3: Create a new issue @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the Issues tab is open
- When: the user presses "+ New issue" (or "New issue" in the command
  palette), types a title, optionally a Markdown body, and presses "Create
  issue"
- Then: the issue is created on GitHub in the code repository, with the
  title and body as typed; a blank body is left out rather than sent empty
- and the new issue opens in the pane, its number assigned by GitHub, with a
  "Created #N" toast; it is at the top of the open list
- and Enter in the title field creates the issue; a blank title cannot be
  submitted

## S4: Draft of a new issue is kept until it is created @v1 [proposed]

- Given: the user is writing a new issue
- When: they type a title or a description
- Then: what they typed is saved as a draft on this device as they type, one
  draft per code repository, and the form says "Draft saved"
- When: GitHub refuses the create — a lapsed sign-in, no network, a rate
  limit
- Then: the form keeps the title and description, the draft stays, and the
  error toast says the draft is saved, so the user can try again
- and leaving the form (Cancel, another view, a reload, quitting the app)
  keeps the draft; the issue list shows it at the top with Resume and
  Discard, and "New issue" opens the form with the draft filled in
- and only a successful create, or Discard after a confirm, deletes the
  draft; a form emptied by hand leaves no draft behind
- and the demo, which creates no issues, keeps no drafts

[test: a draft is saved per repository, host included, and read back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L77 ]
[test: a blank form is no draft, so saving it clears the saved one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L86 ]
[test: clearing forgets it; a garbled entry or an unknown repo reads as none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L94 ]
[test: names the draft and offers Resume and Discard : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L106 ]
[test: an untitled draft says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L116 ]
[test: a failed create keeps the draft; the form brings it back; a later create clears it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L208 ]
[test: the list shows the draft; Discard deletes it after a confirm : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L242 ]
[test: Discard draft on the form empties it, and only on a yes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issueDraft.test.tsx#L257 ]

## S5: GitHub issues are controlled by a switch in user settings @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the "GitHub issues" setting is off, as it is by default
- When: the user looks at the sidebar
- Then: there is no Issues tab, and nothing in the app asks GitHub for issues
- When: the user turns the setting on
- Then: the tab appears, given a GitHub code repo; the setting's hint names
  that repository, or says that project-settings.json names no GitHub code
  repo yet
- and turning it off again hides the tab; an issues view that is open leaves
  the pane and the app goes home
- and the server refuses the module's requests while the switch is off, not
  just the UI

## S6: Issue views have urls and find their way into search @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the setting is on
- When: the user opens `/issues`, `/issues/new` or `/issues/<number>`
- Then: that view opens with the sidebar on the Issues tab, the window title
  names it ("#N <title>"), and Back / Forward move between views; a number
  that names nothing shows a note with a way back to the list
- and every row's ⋯ menu offers Edit title, Copy link and Open on GitHub,
  beside Close or Reopen
- and the command palette opened from the Issues tab searches the loaded
  issues by number, title and body, and opens the one picked
- and the reload tick refreshes the list, and the issue that is open, only
  while an issues view is on screen — an idle app spends none of GitHub's
  rate limit

## S7: Pasted images in an issue are uploaded to GitHub @v1 [proposed]

- Given: the user writes an issue, a change to one, or a comment whose body
  embeds an image pasted into the app (`/api/images/<name>`, a file only this
  app serves)
- When: the create, edit or comment is sent to GitHub
- Then: each such image is committed into the code repository (under
  `.specdriven/issue-images/`) and its link in the body is rewritten to the
  raw GitHub url, so the image shows on GitHub instead of a broken local link
- and the image's name is content-addressed, so the same screenshot across
  several issues is stored once and is not re-committed
- and an image whose bytes are not on disk, or that GitHub refuses to host (no
  push access, a rate limit), keeps its local link rather than failing the
  issue, and a body that embeds no such image is sent to GitHub untouched

[test: a stored image is committed to the code repo and its url rewritten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issues.test.ts#L309 ]
[test: an already-committed image reuses its url without a second commit : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issues.test.ts#L322 ]
[test: an image with no bytes on disk is left as-is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issues.test.ts#L336 ]
[test: a refused upload keeps the local link rather than failing the issue : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issues.test.ts#L347 ]

## S8: Requests to GitHub carry the user's sign-in @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the user is signed in to GitHub in Settings, or through the GitHub
  CLI
- When: the app lists, reads or writes issues
- Then: the request carries that sign-in — the app's own token first, else
  the CLI's — so private repositories and writes work
- and signed out, reads of a public repository still work within GitHub's
  anonymous rate limit, while a write, a private repository or a used-up
  limit shows GitHub's reason in the pane with a button to Settings, where
  the sign-in lives
- and in demo mode the tab reads one fixed public repository —
  SpecDriven/specdriven-app — anonymously, so the demo has real
  issues to show, and every write is refused: no New issue, Edit, Close,
  Reopen or Comment is offered, and the routes behind them answer 403

## S9: Requests to GitHub go to the code repo's host @v1 [proposed]

- Given: the code repo lives on github.com or on a GitHub Enterprise host,
  and the user is signed in in Settings, through the GitHub CLI, or both
- When: the app reads or writes the code repo's issues, or looks up a pull
  request's title or a test's name
- Then: the request goes to that host's API — api.github.com, `/api/v3` on a
  GitHub Enterprise Server, `api.<name>.ghe.com` on GHE.com
- and it carries the app's sign-in only when that sign-in was made on the
  same host; a GitHub Enterprise token is never offered to github.com or any
  other host, the rule git follows too
- and the GitHub CLI's token goes to github.com only, and only while its
  account is in use — signing out of the app turns it off
- and a GitHub Enterprise code repo gets an Issues tab when the app is signed
  in on its host, with its origin in any spelling git writes

[test: issues of a GitHub Enterprise repository are read from its host's API with its token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L105 ]
[test: a GitHub Enterprise token never reaches another host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L113 ]
[test: a github.com sign-in goes to github.com only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L128 ]
[test: the GitHub CLI token is used for github.com only, while its account is in use : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L137 ]
[test: an Enterprise code repo has issues when the app is signed in on its host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L158 ]
[test: Enterprise remotes in every spelling name the repo on the sign-in's host only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/issuesTokenHost.test.ts#L172 ]

## S10: The Issues tab shows the code repo's GitHub issues @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the GitHub issues setting is on, and the data folder's
  project-settings.json names a `code-repo` whose git `origin` is a GitHub
  repository — in any spelling git writes: https, `git@github.com:`, or
  `ssh://`
- When: the user looks at the sidebar
- Then: an Issues tab appears to the right of Tasks (and left of Tests, when
  that is on), with an Issues link in the collapsed rail and `g i` as its
  shortcut
- and the tab lists the repository's open issues, most recently updated
  first, each as `#<number> <title>`, with an Open / Closed switch above the
  list; the pane shows the same list as a table — number, title with its
  labels, state, who opened it and when, assignees, comment count — with an
  Open / Closed / All filter and a link to the repository on GitHub
- and pull requests, which GitHub lists among issues, are left out
- and a code repo that is not a GitHub one, or none at all, means no tab and
  nothing asks GitHub
- and the repository is found from the code repo's git config on disk — a
  worktree's too — so it works on every git engine, including the built-in
  one on a machine with no git

## S11: Update an issue @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: an issue is open in the pane
- When: the user presses "Edit" beside the description, changes the Markdown
  and presses "Save"
- Then: the body is saved to GitHub and the rendered description follows;
  "Cancel" drops the draft. Nothing autosaves — every save is an edit on
  GitHub, so it happens only on "Save"
- and "Edit title" in the ⋯ menu — of the pane, the list row or the sidebar
  row — asks for a new title in a dialog and saves it
- and while an edit or a comment is being typed, the reload tick leaves the
  issues alone
