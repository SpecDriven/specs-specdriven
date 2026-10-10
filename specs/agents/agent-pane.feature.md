# Coding agent pane

## A Claude turn fails fast when the API proxy does not answer @v1 [proposed]

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

[test: a Claude turn behind a proxy that does not answer is stopped after the wait : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentChat.test.ts#L878 ]
[test: a Claude turn whose API answers in time is left to run : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentChat.test.ts#L901 ]
[test: a Claude turn without a proxy, or with no limit set, has no wait : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentChat.test.ts#L914 ]
[test: the first-response wait is a setting, 15 seconds unless changed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentChat.test.ts#L925 ]

## The agent pane starts Claude in Auto mode @v1 [published]

- Given: the coding agent pane, with the Claude agent chosen
- When: a new session starts without the user picking a mode
- Then: it starts in Auto, where Claude handles permission decisions; the
  SDK's "default" is shown as Manual
- and Codex starts in Manual (its "auto" is the same policy), and pi has no
  permission gate

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L360 ]

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

[test: the folder menu offers the code repo, the data repo, then folders sessions used : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L326 ]

## The mode menu describes each mode @v1 [proposed]

- Given: the pane's permission mode control
- When: the user opens it
- Then: a "Mode" heading sits over one numbered item per mode, each with a
  description under it, and a "Default" badge on the mode a new session
  starts in
- and pressing a number key (1–5) chooses that mode directly

[test: the mode menu numbers every mode and badges the agent's default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentPane.test.tsx#L360 ]

## The server runs a chat in the folder the pane picked @v1 [proposed]

- Given: POST /api/agent/chat with an absolute `cwd`
- When: the session starts
- Then: that folder stands in for the code repo, so the data repo still rides
  along as an additional directory (unless it is the folder itself)
- and a `cwd` that is not absolute, or names a folder that does not exist, is
  refused with an error

[test: a folder the pane picks is where the chat works, over the code repo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentChat.test.ts#L598 ]
