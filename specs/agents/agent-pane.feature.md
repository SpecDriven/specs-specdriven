# Coding agent pane

**Created by:** vydra

**Updated by:** vydra

## S1: A Claude turn fails fast when the API proxy does not answer @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a Claude session whose API goes through a proxy (ANTHROPIC_BASE_URL,
  from the environment or Claude Code's settings files) that does not answer
- When: the user sends a message and nothing comes back from the API within
  the wait — 15 seconds unless Settings → Claude says otherwise
- Then: the pane stops that try, says it is trying again, and sends the same
  message again after a pause — 2 seconds, then 4 — until the API has been
  asked three times; a request Claude Code retried on its own counts as a try
- and when the third try gets no answer either, the pane stops the turn and
  shows an error naming the proxy's address and the number of tries, instead
  of a spinner for the minutes Claude Code would keep retrying
- and a try that gets an answer runs on as the turn; the tries the pane gave
  up on leave no result of their own in the pane
- and Stop during the pause between tries ends the turn with nothing sent
  again
- and any streamed event counts as an answer, the API's keepalive pings
  included; Claude Code's own retries do not
- and the session stays up, so the message can be sent again once the proxy
  is back
- and a wait of 0 turns the limit off; a session that talks to the API
  directly is never held to it

[test: a Claude turn behind a proxy that does not answer is stopped after the wait : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1041 ]
[test: a Claude turn whose request the proxy leaves unanswered is sent again, three tries in all : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1073 ]
[test: a Claude turn whose second try gets an answer runs on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1112 ]
[test: Claude Code's own retries count toward the three tries : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1135 ]
[test: Stop while the pane waits to try again ends the turn, with nothing sent again : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1155 ]
[test: a prompt a session starts with is tried again the same way : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1185 ]
[test: a Claude turn whose API answers in time is left to run : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1203 ]
[test: a Claude turn without a proxy, or with no limit set, has no wait : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1216 ]
[test: the first-response wait is a setting, 15 seconds unless changed : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1227 ]

## S2: The agent pane starts Claude in Auto mode @v1 [published]

**Created by:** vydra

**Updated by:** vydra

- Given: the coding agent pane, with the Claude agent chosen
- When: a new session starts without the user picking a mode
- Then: it starts in Auto, where Claude handles permission decisions; the
  SDK's "default" is shown as Manual
- and Codex starts in Manual (its "auto" is the same policy), and pi has no
  permission gate

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L706 ]

## S3: The folder chip switches the folder the agent works in @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the folder chip in the bar under the pane's message box, marked
  with a ▾
- When: the user opens it
- Then: it lists the code repo, the data repo, the folders earlier sessions
  used, and "Other folder…" (a path prompt, with Browse… where the platform
  has a native chooser)
- and a running agent cannot move, so picking another folder starts a new
  session there with the same agent, mode, model and effort; the old session
  keeps its tab
- and on the new-session screen the same chip picks the folder the session
  will start in
- and the data repo is offered by its absolute path, though settings may
  name it relative to the app

[test: the folder menu offers the code repo, the data repo, then folders sessions used : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L667 ]
[test: GET /api/state names the open folder's absolute path, though settings name it relative to the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L165 ]

## S4: The mode menu describes each mode @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the pane's permission mode control
- When: the user opens it
- Then: a "Mode" heading sits over one numbered item per mode, each with a
  description under it, and a "Default" badge on the mode a new session
  starts in
- and pressing a number key (1–5) chooses that mode directly

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L706 ]

## S5: The server runs a chat in the folder the pane picked @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: POST /api/agent/chat with an absolute `cwd`
- When: the session starts
- Then: that folder stands in for the code repo, so the data repo still rides
  along as an additional directory (unless it is the folder itself)
- and a `cwd` that is not absolute, or names a folder that does not exist, is
  refused with an error

[test: a folder the pane picks is where the chat works, over the code repo : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L678 ]

## S6: The composer bar picks the folder, model, effort and mode @v1 [proposed]

- Given: the coding agent pane, with a session or on the new-session screen
- Then: the message box holds a bar under the text, as in Claude Desktop:
  the folder on the left; the model chip and the permission mode on the
  right; Send at the end
- and the bar is always one row: a narrow pane shortens the folder's name
  and drops the chips' chevrons rather than wrapping, the model and the
  mode keep their names whole until the folder's is down to a few letters,
  and the pane is never narrower than 420px ("Footer cmponents" has the
  order in which things give way)
- and the model chip shows the agent's icon, the model's name and, after
  it, the effort (left off while the agent decides it); it opens one picker
  for the agent, the model and the effort, since the effort levels depend
  on the model and the models on the agent
- and on the new-session screen, when more than one agent can start, the
  picker's first row is an icon per agent: Claude's, OpenAI's for Codex,
  OpenCode's and Pi's own marks, and a robot for any other agent; the
  picked one sits on a filled box with a rule under it; hovering or
  focusing an icon shows the agent's name over its description; picking
  one swaps the models and the levels under it and puts the mode back to
  that agent's default, and writes nothing; a running session has no such
  row, its agent being fixed
- and the picker lists the models the agent offers, by display name with a
  line about each (left off when it only repeats the name), the one the
  session runs on checked and the one new sessions start on badged
  "Default"; a session that reports the full id of an alias still finds its
  row
- and a list of up to nine models picks by its number keys; a longer one
  has a search field instead, focused as the picker opens, that keeps the
  models holding every word typed in their name or their id; Enter there
  picks the first one left, Down goes into the list
- and the picker lists what the agent offers now, not what it offered when
  the window opened: the pane loads the list again once it is a minute old
  and as the picker opens, so an agent whose sign-in gains or loses a model
  is followed without switching its plugin off and on
- and under the models is a row of effort buttons: "Default" (the agent
  decides) and each level the chosen model takes — Low, Medium, High, Extra
  high, Max — each explained in its tooltip; the row is left out for a
  model that takes none
- and picking a model keeps the picker open, so the effort can be picked
  for it; picking an effort, ESC or a click outside closes it
- and picking a model or effort in a running session switches it from the
  next request on, notes nothing in the transcript, and makes it what the
  agent's new sessions start on
- and an agent that cannot switch models in a running session offers only
  the effort there, one that offers neither shows the model it reports as
  plain text, and one with no permission gate says "No permission gate"
  where the mode would be
- and the header keeps the sessions menu, the status dot, the billing chip,
  Stop and close

[test: the pane stays between its minimum and 60% of the window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L453 ]
[test: one chip names the agent by its icon, the model and the effort, and opens one picker for all three : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L185 ]
[test: each agent is an icon named by its label, and its tooltip gives the name and what the agent is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L216 ]
[test: an agent's mark is drawn in the text colour, hidden from assistive tech : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L244 ]
[test: picking another agent swaps the models and the effort levels under it, and the session it starts : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L261 ]
[test: picking a model keeps the picker open with that model's effort levels, and picking an effort closes it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L281 ]
[test: a short list picks a model by its number, and the arrows move through the models and the levels : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L308 ]
[test: a long list gets a search field where typing narrows it, Enter picks the first match and digits are text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L331 ]
[test: a running session picks only its model and effort, switching the session itself : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L365 ]
[test: an agent that cannot switch models in a session still picks its effort, and one with nothing to pick is plain text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelPicker.test.tsx#L384 ]
[test: the model list holds the agent's models, the running one checked and the default badged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L759 ]
[test: a model off the list still shows, and with no default the agent's own leads the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L773 ]
[test: a model's description that only repeats its name is left off, as a proxy's rows carry it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L782 ]
[test: a search keeps the models that hold every word typed, in the name or the value : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L793 ]
[test: the effort row offers the levels the model takes, after the agent's own default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L803 ]
[test: the pane's model list is loaded again once it has gone stale, and readers at once share the load : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelRefresh.test.tsx#L83 ]
[test: the pane's model list is loaded again when the model picker opens, and it lists what came back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelRefresh.test.tsx#L112 ]
[test: the pane switches a running session's model and effort, and the pick becomes the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L251 ]
[test: an agent's choices and defaults are listed for the pane and kept through the provider : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L292 ]

## S7: A new session starts from the message box @v1 [proposed]

- Given: the pane with no session open, or the + at the end of the session
  tabs
- Then: it shows a message box with the bar under it — the folder, the
  model chip (where the agent is picked when more than one can start, with
  the model and the effort) and the mode — and no form to fill in
- When: the user sends the first message
- Then: a session starts with that message as its first turn, in the folder,
  on the model and effort and in the mode the bar shows, named after the
  first line of the message
- and a model or effort picked here becomes what the agent's sessions start
  on, as a pick in a running session does
- and POST /api/agent/chat takes the `model` and `effort`; a model with
  spaces or an effort that is no level is refused, and an agent with no
  effort setting is never handed one

[test: a new session starts from the message box, with the folder and mode in its bar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L826 ]
[test: a session is named after the first line of its first message : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L820 ]
[test: a session starts on the model and effort the pane picked, and refuses ones that are none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L228 ]

## S8: Paste an image into the message @v1 [proposed]

- Given: the coding agent pane, with a session or on the new-session screen,
  and an agent that reads images (Claude, Codex and Pi do)
- When: the user pastes an image into the message box — a screenshot, or an
  image copied in a browser
- Then: it shows as a thumbnail above the text, with a button to remove it;
  up to 10 go with one message
- and the message may be the images alone, with no text
- and Send hands the agent the images with the text: Claude gets image
  blocks before the text, Codex `localImage` items, Pi the images of its
  `prompt` command
- and the app keeps each image in the session's own folder beside its
  record (never the data repo), and the message's bubble in the transcript
  shows it; a click opens it full size over the pane, a click or Escape
  closes it
- and a paste that also carries plain text — a copy from a spreadsheet, a
  word processor or Finder — pastes the text, not the picture beside it
- and an image that is not a PNG, JPEG, GIF or WebP, or is over 3.75 MB, is
  scaled down and re-encoded in the browser until it fits (the Claude API
  refuses one whose base64 is over 5 MB)
- and an agent that reads no images refuses the paste by name, and the
  server refuses images for it with a 409; images that are not images, too
  many or too big are a 400 (413 for the size)
- and the demo's simulated Claude takes a pasted image and says it cannot
  look at it

[test: a paste's image files are taken, unless it also carries text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentImagePaste.test.tsx#L87 ]
[test: an image goes as it is when its type and size fit, else it is scaled to fit : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentImagePaste.test.tsx#L100 ]
[test: a pasted image shows as a thumbnail, can be removed, and is sent with the first message : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentImagePaste.test.tsx#L146 ]
[test: an agent that reads no images refuses the paste by name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentImagePaste.test.tsx#L221 ]
[test: images pasted into a message reach the provider, are kept in the session's folder and served back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L593 ]
[test: a session started with pasted images hands them to the provider with its first turn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L633 ]
[test: pasted images are refused for an agent that reads none, and when they are not images, too many or too big : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L649 ]
[test: a pasted image gets a reply that says the simulation cannot look at it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L352 ]
[test: pasted images go to Claude as image blocks before the text, and images alone carry no text block : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L447 ]
[test: pasted images go to Codex as localImage items, read from the copies the app keeps : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L682 ]
[test: pasted images go to pi with its prompt command, as its image items : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-pi/test/piChat.test.ts#L476 ]

## S9: A session can run in its own git worktree @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the new-session screen of the coding agent pane, with a folder in
  a git repo picked in the bar
- When: the user ticks Worktree beside the folder, as in Claude Desktop, and
  sends the first message
- Then: the session gets a git worktree of its own — a new branch
  `specdriven/<session name>-<id>` off the branch the folder has checked
  out — and the agent works there instead of in the folder, so its edits
  never touch the user's checkout and two sessions never share one
- and where the repo has an `origin`, the session starts from up-to-date
  code: the folder has to be on origin's main branch (its default branch,
  else `main`), and before the worktree branches off it the app fetches
  origin and fast-forwards a main that fell behind, leaving uncommitted
  changes as they are
- and what a fast-forward cannot fix refuses the session with a 409 that
  says what to do, and nothing is made: another branch or a detached HEAD
  checked out, a main with commits origin does not have, or a fast-forward
  that uncommitted changes stand in the way of — the app never switches,
  rebases or resets the user's checkout; the box's tooltip names the branch
  sessions start from, and says so when the folder is not on it
- and with origin out of reach the session still starts, from main as last
  fetched, and a notice that stays on screen says so, as does the session
  popover; a worktree session shows no "Start from up-to-date source"
  dialog, since the app has made sure itself
- and the specifications folder is left as it stands — the app keeps it
  level with its remote itself — as is a repo with no `origin`, whose
  sessions branch off whatever is checked out
- and this works for every agent (Claude, Codex, Pi, any agent plugin): the
  app makes the worktree and hands the agent its path as the folder to work
  in, and a folder deep inside the repo keeps its place inside the worktree
- and worktrees live under the settings folder (`~/.specdriven/worktrees/
  <repo>/<name>`), outside every repo, so neither the repo's own tools nor
  the app's data-folder watcher see them
- and the box remembers the last pick; it is greyed out, with the reason in
  its tooltip, for a folder that is not in a git repo, a repo with no commit
  yet, or when git is not on PATH, and POST /api/agent/chat refuses
  `worktree: true` there with a 400; a tight bar shows the box with a branch
  icon in place of the word
- and a worktree session's bar shows the folder it was started in and the
  branch it works on; picking another folder starts the new session in a
  worktree too, where that folder can have one
- and ending the session removes its worktree when nothing in it is
  uncommitted, and keeps it — saying where — when something is; the branch
  and every commit on it always stay
- and a worktree whose agent cannot start is removed again with its branch

[test: a session started with a worktree works in one of its own, on a new branch off the folder's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L103 ]
[test: a folder inside the repo keeps its place inside the worktree, and two sessions never share one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L127 ]
[test: a worktree of the data repo has no second folder beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L139 ]
[test: a folder that cannot have a worktree is refused, and nothing is left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L149 ]
[test: a worktree whose agent cannot start goes again, branch and all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L174 ]
[test: closing a session removes its worktree when nothing is uncommitted, keeps it otherwise, and keeps the branch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L182 ]
[test: the pane asks whether a folder can have a worktree, and on which branch : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L216 ]
[test: a session name becomes a branch-safe slug : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L228 ]
[test: a worktree session starts from main level with origin, and a main that fell behind is fast-forwarded first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L264 ]
[test: a folder on another branch, or a main with commits origin lacks, is refused with nothing left behind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L287 ]
[test: a main that cannot be fast-forwarded over uncommitted changes is refused, and the changes stay : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L323 ]
[test: with origin out of reach the worktree starts from main as last fetched, and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L338 ]
[test: the specifications folder, which the app keeps level itself, is neither fetched nor moved : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L351 ]
[test: the pane learns which branch a worktree session has to start from : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktree.test.ts#L366 ]
[test: the Worktree box offers a worktree for a repo, remembers the pick, and the first message asks for one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L155 ]
[test: a folder that is no repo greys the box out, and the remembered pick is not sent there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L178 ]
[test: a worktree session shows the folder it was started in and its branch, and closing it says a kept worktree's place : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L201 ]
[test: a worktree session on the code repo starts with no source dialog, and says when origin was out of reach : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L273 ]
[test: the Worktree box says which branch the session starts from, and when the folder is not on it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L329 ]

## S10: A worktree session opens in a Ghostty shell @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: Settings → Workflow → "Open worktrees in Ghostty" is on, Ghostty is
  installed where the app runs, and the coding agent pane shows a session
  that runs in its own git worktree
- When: the user clicks the worktree's branch in the pane's bar
- Then: a Ghostty shell opens in the folder the agent works in — the picked
  folder's place inside the worktree — so the user can look at, run and
  commit the agent's work by hand
- and on a Mac the app hands the folder to Ghostty.app (`open -a`), which
  opens it in the instance already running, as a window or a tab as
  Ghostty's own settings say; elsewhere it runs `ghostty
  --working-directory=<folder>`; Ghostty has no Windows build
- and the server takes the folder from the session's own record, never from
  the request (POST /api/agent/chat/:id/ghostty), and hands it to Ghostty as
  one argument, never through a shell
- and the setting is off by default and kept in .specdriven/settings.yaml
  (`ghostty`); off, the branch stays a plain label and the route refuses
  with a 409
- and without Ghostty the setting's hint says where to get it and a click
  toasts the same (502); a worktree removed since says its folder is gone
  (404), and a launcher that fails says why
- and the demo and the hosted app run on a server, not the user's machine:
  they hide the setting, and refuse it and the route with a 403

[test: Ghostty is the app bundle on a Mac, the binary on PATH elsewhere, and nothing on Windows : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L107 ]
[test: the folder is handed over as one argument, never through a shell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L127 ]
[test: the Ghostty setting defaults to OFF and only an explicit true turns it on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L138 ]
[test: the state reports the Ghostty setting and whether Ghostty is installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L145 ]
[test: the Ghostty setting round-trips through the settings file, OFF stored as an absent key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L154 ]
[test: the demo and the hosted app cannot turn Ghostty on, nor open it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L167 ]
[test: the open-in-Ghostty route is a POST on a session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L184 ]
[test: a worktree session opens Ghostty in the folder its agent works in, inside the worktree : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L189 ]
[test: with the setting off, Ghostty missing or the folder gone, nothing opens and the reason is said : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L198 ]
[test: a launcher that fails is reported, not swallowed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ghostty.test.ts#L223 ]
[test: with Ghostty on, a click on the worktree's branch asks the server to open a shell there, and a refusal is toasted : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentWorktreePane.test.tsx#L222 ]

## S11: The session popover shows the folder, the branch and the context window @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a session in the coding agent pane
- When: the user clicks the ring beside the session's name in the pane's
  header, as in Claude Desktop
- Then: a popover opens on the folder the session works in (its name, then
  the folder it sits in, the home folder written as ~), the branch — for a
  worktree session the session's own branch with a Worktree tag, the branch
  it was made from and where the worktree is; otherwise the branch the folder
  has checked out, or "Not in a git repository" — and how full the agent's
  context window is
- and the window reads "15% used · 151.5k / 1M" over a thin bar split into
  what fills it (MCP tools, system tools, messages, system prompt, memory
  files, skills), the reserve the agent keeps for compacting and the free
  space, each named in a legend with its tokens and share; the parts keep
  their colours as they grow, and tool schemas the agent has not loaded are
  listed but not counted
- and under the legend each part made of things — MCP tools, memory files,
  skills, system tools — opens on a list of them, biggest first, with the
  count beside its name ("MCP tools 132")
- and the ring fills with the share in use and says it ("15%"), amber from
  70% and red from 90%; it stays empty until the agent has counted
- and the agent counts its window once the session is ready, after every
  turn once the message box is free again, and after compacting; Refresh
  asks it to count every category with the API ("Measured"), and waits
  while a turn runs; the count comes down the session's stream, and the log
  keeps only the newest
- and Claude reports what fills the window and offers Refresh; Codex reports
  the count and its window's size alone; an agent that reports nothing (Pi)
  says so; POST /api/agent/chat/:id/context refuses with a 409 an agent that
  cannot count on request, or a session still starting
- and the session list leaves the count out and the session's record keeps
  its totals only; the demo's simulated session counts too; the home folder
  is never sent from the demo or a shared server

[test: a worktree session shows its folder, branch, base and the window by category : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L229 ]
[test: Refresh is there only for an agent that counts on request, and waits for the turn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L252 ]
[test: an agent that counts nothing says so, and one that counts says when : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L268 ]
[test: a measured count says so, and a count with no window shows no bar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L276 ]
[test: the header button shows how full the window is, and nothing while unknown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L284 ]
[test: write counts the way Claude Desktop does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L96 ]
[test: give a share of the window to one decimal : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L110 ]
[test: the home folder is ~, and a long path loses its middle, whole folders first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L121 ]
[test: a folder's parent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L134 ]
[test: the percentage needs a window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L144 ]
[test: the bar draws used parts in their colour order, then the reserve and free space : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L156 ]
[test: an agent with totals only gets one used part, a derived reserve and what is left : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L176 ]
[test: a context event leaves the transcript as it was : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L188 ]
[test: the header ring opens the session popover on the folder, its branch and the window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRail.test.tsx#L140 ]
[test: Refresh asks the agent to count every category, and a count on the stream moves the ring : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRail.test.tsx#L233 ]
[test: a provider's context count reaches the pane, the log keeps the newest, and the record its totals : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L412 ]
[test: the pane's Refresh asks the provider to count again, and is refused where it cannot : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L460 ]
[test: a demo session counts its context window, and counts it again on request : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L302 ]
[test: Codex's token usage is the context count, the last request's tokens of the model's window : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L1002 ]
[test: a Codex session shows its context count as Codex reports it, once per change : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L1019 ]

## S12: Each session has its own tab, at most five @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the coding agent pane with sessions open, as in Claude Code
- Then: the pane's header line holds one tab per open session, on the one
  line with the pane's chips and buttons rather than on a row of its own,
  in the order they started, each with its name and a dot for how it is
  doing (working, waiting for an answer, ready, ended, failed); the full
  name, the agent, its state and its folder show on hover
- When: the user clicks a tab, or moves to it with the arrow keys, Home or
  End
- Then: the pane shows that session's transcript and composer, its tab in
  front; the other sessions run on behind it, each followed live,
  so their dots keep up and a question one of them asks still raises a toast
- and on reload every open session comes back as a tab, the one last shown
  in front
- When: the user clicks the + after the tabs
- Then: the new-session screen opens on a "New session" tab of its own,
  which has nothing to close and goes when another tab is picked
- and at most five sessions are open at once, ended ones included, since an
  ended session keeps its tab until it is closed: with five open the + is
  greyed out and says "At most 5 sessions — close one to start another",
  the new-session screen says why it cannot send, and POST /api/agent/chat
  (as every other way a session starts — a spec's Start session, a
  self-fix) answers 409 "At most 5 coding agent sessions can be open at once
  — close one first"
- When: the user closes a tab with its ×, a middle-click or Delete
- Then: the session ends and its tab goes, and the tab to its right is
  shown, else the one to its left, else the new-session screen
- and a session whose agent is still working, or waiting for an answer,
  asks "End this session?" first, and No keeps it; one that is ready, ended
  or failed closes at once

[test: each open session has a tab in start order, every one followed, and a click shows its transcript : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L201 ]
[test: the arrow keys, Home and End move between tabs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L225 ]
[test: + opens the new-session screen on a tab of its own, until five tabs are open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L244 ]
[test: with five tabs open, ended ones among them, + and the message box wait for one to close : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L258 ]
[test: × closes an idle tab at once and shows the tab to its right : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L273 ]
[test: × on a tab whose agent is at work asks first, and a no keeps the session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L289 ]
[test: closing the last tab leaves the new-session screen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L312 ]
[test: tabs run in the order their sessions started, and a closed tab hands over to its right, else its left : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L175 ]
[test: five open tabs, ended ones included, leave no room for another : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L183 ]
[test: a session asks before closing only while its agent is at work : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L190 ]
[test: a tab's tooltip names the session, its agent, its state and its folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentTabs.test.tsx#L196 ]
[test: at most five sessions are open at once, an ended one holding its tab until it is closed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L120 ]

## S13: The pane header says whether the agent spends an account or the API @v1 [proposed]

- Given: an agent that reports its sign-in, such as Claude
- When: the coding agent pane is open — with a session in it, or with none
  yet, on the new-session screen
- Then: the right of the pane's header always shows one of two icons: a
  plug while the agent's calls go through the API — an API key, a bearer
  token, an API proxy or a cloud provider such as Bedrock — or a person
  while it spends a Claude account's plan
- and with no session open the chip speaks for the agent a new session
  would start on, from the sign-in Settings → Claude has chosen; once a
  session reports its own sign-in, the chip shows that
- and an account that is not signed in yet still shows the person, in red
- and a sign-in changed in Settings shows in the chip without a reload
- and the chip's tooltip says which of the two it is, API or Account, then
  keeps the agent's own words for the sign-in — the plan's name, the
  proxy's address, who is signed in and where the credential came from
- and a screen reader hears the same word, API or Account, with the
  sign-in's name, the icon itself being silent
- and Settings → Claude names the second way to sign in "API", next to
  "Claude account (email login)"

[test: the account chip shows the plug for a key, a token, a proxy or a provider, the person for an account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentAccount.test.ts#L9 ]
[test: the account chip's tooltip says API or Account, then the sign-in, who and where from, once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentAccount.test.ts#L20 ]
[test: with no session open, the header shows the plug or the person for the agent a new session starts on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentAccount.test.ts#L57 ]
[test: the header's agent with no session is the one picked, while usable, else the first usable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentAccount.test.ts#L83 ]
[test: a sign-in change asks again for every list already loaded, so the chip follows it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentAccount.test.ts#L91 ]
[test: describeSignIn says API through the proxy, or the Claude account and whether it is signed in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1443 ]
[test: the pane's choices carry the sign-in a new session would start under, when the host says : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1462 ]
[test: the app's choices route carries the sign-in, Not logged in with no one signed in : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1471 ]

## S14: Switching folders tries the agent's API again @v1 [proposed]

- Given: an agent whose API goes through an endpoint of its own, such as
  Claude signed in with API through a proxy
- When: the user switches folders — opens a code repository, or picks
  another specifications folder
- Then: the app tries the agent's API again from the folder now open, so a
  proxy that went quiet while the user was elsewhere is woken before the
  first turn waits on it; the switch itself does not wait
- and Claude checks the proxy a session in that folder would use — the one
  Settings → Claude names, else the folder's own Claude Code settings — up
  to three times, a second apart, and a proxy that refuses the token is an
  answer, not tried again
- and when the API does not answer, the pane says so under its header, with
  the API's address, with or without a session open, and a Retry button
  that tries again from the open folder; once it answers the line goes
- and an agent signed in to an account has nothing to try and shows nothing

[test: a proxy that refuses the token is an answer: not tried again, and reported as not ready : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L84 ]

[test: opening a code repository tries the agent's API from it, and the choices carry the answer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L118 ]
[test: picking another specifications folder tries the API again from that folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L128 ]
[test: a settings change that switches no folder tries nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L136 ]
[test: Retry tries again from the open folder and the answer replaces the last one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L144 ]
[test: an agent with nothing to try has no answer, and a try that throws is a failed one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L155 ]
[test: tries that overlap share one call to the agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L169 ]
[test: the pane warns of an API that did not answer, with Retry, and says nothing once it does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentReconnect.test.ts#L180 ]
[test: a proxy that answers is reported as answering, after one check with the stored token : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L59 ]
[test: an unreachable proxy is tried again a moment later, and answers the second time : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L67 ]
[test: a proxy that never answers is reported down after every try, naming its address : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L74 ]
[test: a proxy that answers something other than the model list still answers : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L91 ]
[test: an account sign-in has nothing to try : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L95 ]
[test: the proxy is the one the folder's own Claude Code settings name, when Settings names none : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L102 ]
[test: withReconnect gives the provider the app's reconnect hook : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/reconnect.test.ts#L113 ]

## S15: Up and Down recall sent messages @v1 [proposed]

- Given: the coding agent pane's message box, with a session or on the
  new-session screen, and messages sent from this device before
- When: the user presses Up with the caret on the box's first line
- Then: the box shows the message sent last; each further Up shows the one
  before it, and at the oldest it stays
- and Down shows the one after, and past the newest the box gets back the
  draft that was being typed when Up was first pressed
- and Down recalls nothing until Up has, and inside a message of several
  lines the arrows move the caret as they always did
- and the history is one list for every session, kept on the device: the
  last 100 messages, a message sent twice in a row kept once
- and while the "/" popup is open the arrows move through its commands
  instead

[test: a sent message joins the history, once in a row, the oldest falling off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L70 ]
[test: Up walks back through sent messages and Down returns to the draft : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L82 ]
[test: Up and Down recall sent messages, from the box's first and last line only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L179 ]

## S16: /help lists the commands @v1 [proposed]

- Given: the coding agent pane's message box
- When: the user sends `/help`
- Then: a panel over the box lists every "/" command — the pane's own and
  the ones the session's agent reported — each with its arguments and what
  it does, and the box's keys
- and nothing is sent to the agent, so it answers while the agent is
  mid-turn and on the new-session screen, where it says the agent's own
  commands are listed once a session has started
- and `/help` is in the "/" popup with the agent's commands; an agent with
  a `/help` of its own has one row there, and the pane still answers it
- and Escape, the panel's close button or the next message closes it

[test: /help is the pane's own command, listed beside the agent's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L99 ]
[test: /help lists the commands in the pane and sends nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L211 ]

## S17: The open feature or file goes with a message @v1 [proposed]

- Given: the coding agent pane beside a feature, or — with a code
  repository open — beside a file in the Files tab
- When: the user sends a message
- Then: the agent is told the open file in a line after the user's words,
  by its path — relative to the folder the agent works in, in full when it
  is elsewhere (a specification while the session runs in the code
  repository), and in the worktree for a worktree session — and reads it
  itself
- and the message box shows the file as a chip above the text, dashed
  because it changes as the main pane does
- and the chip's pin keeps the file in the context when something else is
  opened; a kept file's chip is solid, and the newly opened one joins it
- and every chip has a remove button: removing a kept file drops it, and
  removing the open file stops the open file going with messages — on this
  device, until the chip the row then offers adds it back
- and the message's bubble in the transcript names the files it went with
- and a "/" command goes to the agent as it is, with no files
- and the server takes only paths inside the open folder, at most 20

[test: the open file goes with a message unless removed; pinned files stay : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L113 ]
[test: the open feature goes with a message; it can be kept, removed and added back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L241 ]
[test: a message to a running session carries the files : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentComposer.test.tsx#L309 ]
[test: a file is named relative to the agent's folder, or in full outside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L718 ]
[test: the files follow the user's words in one line; a /command goes as it is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L731 ]
[test: only paths inside the open folder are taken : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L739 ]
[test: a message's files are shown under it and told to the agent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L751 ]

## S18: Switching folders closes the other folder's sessions @v1 [proposed]

- Given: the coding agent pane with sessions open in one folder — a
  specifications folder, or a code repository opened in its place
- When: the user switches to another folder
- Then: every session started while the other folder was open ends, as
  its tab's × would end it, before the switch is done: its agent stops, its
  record says it ended, and a worktree of its own goes unless something in
  it is uncommitted
- and its transcript ends with "The folder was switched; this session
  belonged to another folder." where a tab closed with × says "Session
  ended."
- and a session belongs to the folder open when it started, even when the
  pane picked another folder for it to work in
- and the pane is ready for the folder now open: the closed sessions' tabs
  go, the new-session screen shows with the new folder's default, files
  kept from the other folder go, and the agents' models are read again
- and a toast says how many sessions were closed, as "Closed 2 coding agent
  sessions from the other folder"
- and picking the folder that is already open, or a settings change that
  switches no folder, closes nothing

[test: a session belongs to the folder open when it started, not the folder the pane picked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L96 ]
[test: picking another specifications folder closes the sessions of the one before, before it answers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L103 ]
[test: opening a code repository closes the specifications folder's sessions, and switching back closes its : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L119 ]
[test: the open folder's own sessions stay, and a change that switches no folder closes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L128 ]
[test: the pane drops the closed sessions' tabs after a switch and opens on the new-session screen : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L137 ]
[test: a closed session's transcript says it belonged to another folder; a folder written with a slash is the same folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L154 ]
[test: the toast after a switch says how many sessions went, and nothing when none did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentFolderSwitch.test.ts#L176 ]

## S19: Footer cmponents @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Repo name followed by branch name followed by '[] Worktree'
Repo name and branch name need max width set so they don't take up too much space compared to other fields in the footer.

![image](/api/images/image-markup-6885989d.png)

Note: When Code agent pane has minimal width, the footer is difficult to use.

![image](/api/images/image-0956594e.png)

- Then: the bar under the message box reads, from the left: the folder's
  name, the branch the folder has checked out, and the Worktree box; a
  running session shows the folder and the branch — its own for a worktree
  session — and no box (the agent is picked in the model chip, on the right)
- and the folder's name and the branch's are each cut off with an ellipsis
  at about 18 letters, the whole name in the tooltip; a folder in no git
  repository shows no branch, and a detached HEAD reads "detached"
- and the branch is read again when a session's turn starts or ends and when
  the window is looked at again, since it changes outside the bar
- and the bar stays one row down to the pane's narrowest (420px) with no
  chip on top of another: a narrowing pane drops, in this order, the chips'
  chevrons, the word "Worktree" (a branch icon stands in), a new session's
  branch, and the folder's icon; past that the folder's and the branch's
  names shorten, and only once they are down to a few letters the model
  chip's — its effort before the model's name, its agent icon never — and
  then the mode's

[test: a new session's bar shows the folder, then the branch it has checked out, then the Worktree box : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L141 ]
[test: a session in the folder itself shows the branch the folder has checked out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L162 ]
[test: the branch is read again when the stamp changes and when the window is looked at again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L175 ]
[test: two readers asking again in the same moment share one answer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L218 ]
[test: a folder's answer is kept for a while, asked for again once older than the reader allows, and a failed ask is no repository : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L242 ]
[test: a branch is named as git names it, a detached HEAD as detached, and no repository not at all : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L263 ]
[test: the folder's and the branch's names are capped below the width any chip may take : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L317 ]
[test: the bar is one row laid out chip by chip, so a chip gives way and never sits on its neighbour : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L332 ]
[test: a narrowing bar drops its chevrons, the Worktree word, a new session's branch and the folder's icon, in that order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentBarFooter.test.tsx#L356 ]

## S20: The transcript view shows a session at three levels of detail @v1 [proposed]

- Given: a session in the coding agent pane
- When: the user opens the session popover (the ring in the pane's header)
  and picks Normal, Thinking or Verbose in its Transcript row, as in Claude
  Desktop's Transcript view
- Then: the transcript is redrawn at once in that view, and every session on
  this device opens in it from then on; a device that never picked one shows
  Normal
- and Normal is the conversation: a run of two or more tool calls in a row
  is one line that says what they did ("Read 3 files, ran 2 commands") and
  opens on the calls themselves; a call still running, a call that failed
  and a call alone stay their own row; what the agent thought is left out,
  with one "Thinking…" line while it is thinking; the notes an agent starts
  with are left out
- and Thinking shows every tool call on its own closed row, as the pane did
  before the views, and what the agent thought, dimmed, above the words it
  led to and streamed as it comes
- and Verbose shows everything: every tool call opened on its input and its
  output (long output scrolls in its own box), what the agent thought, the
  notes an agent starts with, and how many turns the reply took
- and what a subagent said and called sits inside the row of the call that
  started it, which says how many calls it made; under Verbose it is shown
  inline where it happened, a step in
- and the user's messages, the agent's replies, permission and question
  cards, the line that ends a turn and every error are shown in all three
- and the view changes nothing about the session: the server keeps and
  sends the same events whatever the view, so switching needs no reload
- and an agent that does not report its thinking has nothing more to show
  under Thinking than a row per call; the Thinking choice says so in its
  tooltip
- and the demo's simulated session thinks and makes a run of calls, so all
  three views can be tried there

[test: Normal folds each run of tool calls into one line and leaves thinking and start-up notes out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L283 ]
[test: Thinking shows every call on its own row and what the agent thought : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L295 ]
[test: Verbose shows everything, in the order it happened : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L299 ]
[test: a call still running or one that failed stays out of a run in Normal : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L303 ]
[test: Normal says a thought is happening only while it streams; an error note shows in every view : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L323 ]
[test: a subagent's traffic sits inside its call's row, and inline a step in under Verbose : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L331 ]
[test: a run's line counts each kind of call, in the order they first appear : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L404 ]
[test: thinking streams into one item, and the settled block replaces the stream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L176 ]
[test: an empty thinking block adds nothing, and a turn's result ends a thought still streaming : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L196 ]
[test: what a subagent says and calls is marked with the call that started it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L208 ]
[test: each transcript view renders its rows, a folded run, the thinking or every call opened : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L514 ]
[test: Normal shows a thought still streaming as one line, and a subagent call says how many calls it made : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L554 ]
[test: a subagent call counts the calls made under it, folded runs and deeper subagents included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L351 ]
[test: the view is kept on the device, Normal until one is picked and for a value it does not know : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L369 ]
[test: the popover offers Normal, Thinking and Verbose, with Normal picked on a device that never chose : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L200 ]
[test: the Thinking choice says when the agent does not report its thinking : https://github.com/SpecDriven/specdriven-app/blob/main/tests/sessionRail.test.tsx#L215 ]
[test: the popover's Transcript row switches the view, the transcript follows, and the choice is kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentRail.test.tsx#L177 ]
[test: the script streams its text before settling it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L163 ]
[test: a demo session starts and plays a scripted Claude Code turn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L214 ]
