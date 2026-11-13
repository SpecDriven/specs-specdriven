# Integration with OpenCode agent

**Created by:** David Vydra

**Updated by:** David Vydra

OpenCode (opencode.ai) is an open-source terminal coding agent that works with any model its user is signed in to — OpenAI, Anthropic, GitHub Copilot, OpenRouter, local models. The `agent-opencode` plugin puts it in the coding agent pane beside Claude, Codex and Pi, driven over OpenCode's Agent Client Protocol server (`opencode acp`). Written against OpenCode 2.x.

## S1: Choose OpenCode agent @v1 [proposed]

- Given: the `opencode` command is on PATH
- When: the app starts
- Then: it installs the `agent-opencode` plugin from the marketplace, and OpenCode is one of the agents the coding agent pane and "Start session" offer
- and a session runs `opencode acp` in the session's folder, starts on the model and effort picked, and hands OpenCode the app's MCP servers
- and the pane shows OpenCode's version, model and folder, and its slash commands in the "/" popup
- and the installed plugin also finds OpenCode in `~/.opencode/bin`, where OpenCode's installer puts it, when PATH has none
- and with no `opencode` anywhere the agent is offered as not installed, with how to get it

[test: an OpenCode session spawns `opencode acp` on the work dir, asking for edits and commands : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L132 ]
[test: Start session can open the generated prompt in an OpenCode pane session : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L518 ]
[test: the app's MCP servers as ACP's stdio servers : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L196 ]
[test: slash commands, with the hint for their input : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L190 ]
[test: PATH first, else where OpenCode's installer puts it — a desktop app's PATH lacks that folder : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L261 ]
[test: OpenCode needs its CLI : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L540 ]

## S2: Model and effort menus offer OpenCode's models @v1 [proposed]

- Given: OpenCode is signed in to one or more model providers
- When: the user opens the pane's model menu
- Then: it lists the data repo's `.specdriven/opencode-models.json` when that lists any, else the default and the models OpenCode used last, else the first models `opencode models` reports
- and picking a model or an effort switches the running session from its next request, and new sessions start on it
- and "Max" effort is OpenCode's highest ("xhigh"); an effort the model does not take is refused with the ones it does
- and listing the models does not leave OpenCode's background service running

[test: the model and effort switch mid-session, and become what new sessions start on : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L429 ]
[test: the model menu is the data repo's list when it has one, else OpenCode's recent models : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L465 ]
[test: the pane's effort levels as OpenCode offers them : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L182 ]

## S3: Permission cards follow the pane's mode @v1 [proposed]

- Given: an OpenCode session in the pane
- When: OpenCode wants to edit a file, run a command or fetch from the web
- Then: in Default mode the pane shows a permission card — Allow once, Allow always, or Deny
- and in Accept edits mode file changes go ahead without a card, commands and fetches still ask
- and in Plan mode OpenCode runs as its Plan agent, which cannot edit the project
- and in Bypass permissions mode nothing asks
- and what the user's own OpenCode config denies stays denied in every mode, Bypass included
- and "Allow always" is OpenCode's own: it keeps a rule for the folder (for a command, every command that starts the same way) and stops asking, in later sessions too
- and Deny ends OpenCode's turn; a reason typed with it goes to OpenCode as a follow-up, and the turn carries on
- and the mode can change mid-session; the user's own OpenCode config files are never written

OpenCode runs edits and commands without asking by default. The app starts it with every edit, command and fetch it would allow turned into one it asks for — for that process only — and answers what the mode allows itself.

[test: OpenCode's permission requests are the pane's cards: once, always, or a refusal : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L305 ]
[test: the user's own denials, and the Plan agent's on edits, stand: only what OpenCode allows becomes a question : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L170 ]

[test: a refusal without a reason ends OpenCode's turn, and the pane says so : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L348 ]
[test: bypass answers every request itself, and accept edits answers the edits : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L363 ]
[test: plan mode is OpenCode's Plan agent, and leaving it is its Build agent : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L384 ]
[test: allow becomes ask, in order, and a deny or an ask stays what it was : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L218 ]

## S4: Stop cancels OpenCode's turn @v1 [proposed]

- Given: OpenCode is working on a turn, maybe with a permission card open
- When: the user presses Stop
- Then: OpenCode stops, the open card closes, a running command is interrupted, and the composer is free again

[test: Stop cancels the turn and withdraws an open card : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L399 ]

## S5: The chat shows OpenCode's replies, tools, context and cost @v1 [proposed]

- Given: an OpenCode session in the pane
- When: the user sends a message, with pasted images or not
- Then: OpenCode's reply streams in, each tool it uses is a row with its result, the session popover shows how full the context window is, and the turn's footer shows its duration and cost
- and a turn OpenCode refuses, or an OpenCode that exits or never starts, ends with its own words
- and the app's one-shot turns (a task estimate) can run on OpenCode, read-only

[test: a turn goes out as session/prompt, and its reply, tools, context and cost stream into the log : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L204 ]
[test: a tool's row waits for its input, and its result closes it : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L81 ]
[test: pasted images go to OpenCode as image blocks of the prompt : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L254 ]
[test: a turn OpenCode refuses ends with its words : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L421 ]
[test: OpenCode exiting on its own ends the session, with its last words on a failure : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/opencodeChat.test.ts#L485 ]
[test: `opencode run` with the Plan agent; the text parts are the answer, parsed when a schema asks : https://github.com/SpecDriven/plugins/blob/main/plugins/agent-opencode/test/acp.test.ts#L347 ]
