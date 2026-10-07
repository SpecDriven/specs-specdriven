# Coding agent pane

## The agent pane starts Claude in Auto mode @v1 [proposed]

- Given: the coding agent pane, with the Claude agent chosen
- When: a new session starts without the user picking a mode
- Then: it starts in Auto, where Claude handles permission decisions; the
  SDK's "default" is shown as Manual
- and Codex starts in Manual (its "auto" is the same policy), and pi has no
  permission gate

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven_app/blob/main/tests/agentPane.test.tsx#L347 ]

## The mode menu describes each mode @v1 [proposed]

- Given: the pane's permission mode control
- When: the user opens it
- Then: a "Mode" heading sits over one numbered item per mode, each with a
  description under it, and a "Default" badge on the mode a new session
  starts in
- and pressing a number key (1–5) chooses that mode directly

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven_app/blob/main/tests/agentPane.test.tsx#L347 ]

## The folder chip switches the folder the agent works in @v1 [proposed]

- Given: the pane's folder chip, marked with a ▾
- When: the user opens it
- Then: it lists the code repo, the data repo, the folders earlier sessions
  used, and "Other folder…" (a path prompt, with Browse… where the platform
  has a native chooser)
- and a running agent cannot move, so picking another folder starts a new
  session there with the same agent and mode; the old session stays in the
  Sessions menu
- and the new-session form offers "Change folder…" for the same choice
  before starting

[test: the folder menu offers the code repo, the data repo, then folders sessions used : https://github.com/SpecDriven/specdriven_app/blob/main/tests/agentPane.test.tsx#L326 ]

## The server runs a chat in the folder the pane picked @v1 [proposed]

- Given: POST /api/agent/chat with an absolute `cwd`
- When: the session starts
- Then: that folder stands in for the code repo, so the data repo still rides
  along as an additional directory (unless it is the folder itself)
- and a `cwd` that is not absolute, or names a folder that does not exist, is
  refused with an error

[test: a folder the pane picks is where the chat works, over the code repo : https://github.com/SpecDriven/specdriven_app/blob/main/tests/agentChat.test.ts#L580 ]
