# Coding agent pane

**Created by:** vydra

**Updated by:** vydra

## S1: A Claude turn fails fast when the API proxy does not answer @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a Claude session whose API goes through a proxy (ANTHROPIC_BASE_URL,
  from the environment or Claude Code's settings files) that is not running
- When: the user sends a message and nothing comes back from the API within
  the wait — 15 seconds unless Settings → Workflow says otherwise
- Then: the pane stops the turn and shows an error naming the proxy's address
  and how many times Claude Code retried, instead of a spinner for the
  minutes Claude Code would keep retrying
- and any streamed event counts as an answer, the API's keepalive pings
  included; Claude Code's own retries do not
- and the session stays up, so the message can be sent again once the proxy
  is back
- and a wait of 0 turns the limit off; a session that talks to the API
  directly is never held to it

[test: a Claude turn behind a proxy that does not answer is stopped after the wait : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L990 ]
[test: a Claude turn whose API answers in time is left to run : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1013 ]
[test: a Claude turn without a proxy, or with no limit set, has no wait : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1026 ]
[test: the first-response wait is a setting, 15 seconds unless changed : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L1037 ]

## S2: The agent pane starts Claude in Auto mode @v1 [published]

**Created by:** vydra

**Updated by:** vydra

- Given: the coding agent pane, with the Claude agent chosen
- When: a new session starts without the user picking a mode
- Then: it starts in Auto, where Claude handles permission decisions; the
  SDK's "default" is shown as Manual
- and Codex starts in Manual (its "auto" is the same policy), and pi has no
  permission gate

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L378 ]

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
  stays in the Sessions menu
- and on the new-session screen the same chip picks the folder the session
  will start in
- and the data repo is offered by its absolute path, though settings may
  name it relative to the app

[test: the folder menu offers the code repo, the data repo, then folders sessions used : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L339 ]
[test: GET /api/state names the open folder's absolute path, though settings name it relative to the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/recentFolders.test.ts#L157 ]

## S4: The mode menu describes each mode @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the pane's permission mode control
- When: the user opens it
- Then: a "Mode" heading sits over one numbered item per mode, each with a
  description under it, and a "Default" badge on the mode a new session
  starts in
- and pressing a number key (1–5) chooses that mode directly

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L378 ]

## S5: The server runs a chat in the folder the pane picked @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: POST /api/agent/chat with an absolute `cwd`
- When: the session starts
- Then: that folder stands in for the code repo, so the data repo still rides
  along as an additional directory (unless it is the folder itself)
- and a `cwd` that is not absolute, or names a folder that does not exist, is
  refused with an error

[test: a folder the pane picks is where the chat works, over the code repo : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L629 ]

## S6: The composer bar picks the folder, model, effort and mode @v1 [proposed]

- Given: the coding agent pane, with a session or on the new-session screen
- Then: the message box holds a bar under the text, as in Claude Desktop:
  the folder on the left; the model, the effort and the permission mode on
  the right; Send at the end
- and the bar is always one row: a narrow pane shortens the folder's name
  and drops the chips' chevrons rather than wrapping, the model, effort and
  mode keep their names whole, and the pane is never narrower than 420px
  (only a phone-wide screen shortens every name to keep Send in the box)
- and the model menu lists the models the agent offers, by display name
  with a line about each (left off when it only repeats the name), the one
  the session runs on checked and the one new sessions start on badged
  "Default"; a session that reports the full id of an alias still finds its
  row
- and the model menu lists what the agent offers now, not what it offered
  when the window opened: the pane loads the list again once it is a minute
  old and as the menu opens, so an agent whose sign-in gains or loses a
  model is followed without switching its plugin off and on
- and the effort menu offers "Default" (the agent decides) and each level
  the chosen model takes — Low, Medium, High, Extra high, Max — and is
  hidden for a model that takes none
- and picking a model or effort in a running session switches it from the
  next request on, notes nothing in the transcript, and makes it what the
  agent's new sessions start on
- and an agent that offers no models shows the model it reports as plain
  text, and one with no permission gate says "No permission gate" where the
  mode would be
- and the header keeps the sessions menu, the status dot, the billing chip,
  Stop and close

[test: the pane stays between its minimum and 60% of the window : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L192 ]
[test: the model menu lists the agent's models, the running one checked and the default badged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L430 ]
[test: a model off the list still shows, and with no default the agent's own leads the menu : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L446 ]
[test: a model's description that only repeats its name is left off, as a proxy's rows carry it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L475 ]
[test: the effort menu offers the levels the model takes, after the agent's own default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L455 ]
[test: the pane's model list is loaded again once it has gone stale, and readers at once share the load : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelRefresh.test.tsx#L83 ]
[test: the pane's model list is loaded again when the model menu opens, and the menu lists what came back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentModelRefresh.test.tsx#L112 ]
[test: the pane switches a running session's model and effort, and the pick becomes the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L226 ]
[test: an agent's choices and defaults are listed for the pane and kept through the provider : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L267 ]

## S7: A new session starts from the message box @v1 [proposed]

- Given: the pane with no session open, or "New session" from the Sessions
  menu
- Then: it shows a message box with the bar under it — the agent (when more
  than one can start), the folder, the model, the effort and the mode — and
  no form to fill in
- When: the user sends the first message
- Then: a session starts with that message as its first turn, in the folder,
  on the model and effort and in the mode the bar shows, named after the
  first line of the message
- and a model or effort picked here becomes what the agent's sessions start
  on, as a pick in a running session does
- and POST /api/agent/chat takes the `model` and `effort`; a model with
  spaces or an effort that is no level is refused, and an agent with no
  effort setting is never handed one

[test: a new session starts from the message box, with the folder and mode in its bar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L482 ]
[test: a session is named after the first line of its first message : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L476 ]
[test: a session starts on the model and effort the pane picked, and refuses ones that are none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L203 ]

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
[test: images pasted into a message reach the provider, are kept in the session's folder and served back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L431 ]
[test: a session started with pasted images hands them to the provider with its first turn : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L471 ]
[test: pasted images are refused for an agent that reads none, and when they are not images, too many or too big : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentEngine.test.ts#L487 ]
[test: a pasted image gets a reply that says the simulation cannot look at it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/demoAgent.test.ts#L297 ]
[test: pasted images go to Claude as image blocks before the text, and images alone carry no text block : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-claude/test/pane.test.ts#L431 ]
[test: pasted images go to Codex as localImage items, read from the copies the app keeps : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-codex/test/codexChat.test.ts#L680 ]
[test: pasted images go to pi with its prompt command, as its image items : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-pi/test/piChat.test.ts#L476 ]
