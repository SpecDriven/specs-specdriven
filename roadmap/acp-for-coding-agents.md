# ACP for coding agents

A comparison, not a decision: what the Agent Client Protocol (ACP) is today, how JetBrains Air and Zed use it, how it lines up with the way SpecDriven runs coding agents, and what we would gain and lose by using it more. It belongs with [Coding Agent UI](coding-agent-ui.md) and [Plugin architecture](plugin-architecture.md); the behaviour we have today is specified in [specs/agents/agent-plugins.feature.md](../specs/agents/agent-plugins.feature.md) and [specs/agents/opencode.feature.md](../specs/agents/opencode.feature.md).

External facts were checked against their sources on 2026-11-21. The protocol, the adapters and the registry change weekly, so read the numbers as a snapshot.

## Recommendation

1. **Add one generic ACP plugin, beside the Claude and Codex plugins.** It runs any agent that speaks ACP, so a new agent no longer needs a plugin of its own.
2. **Keep Claude and Codex on their native channels for now.** Our plugins already talk to the same two things the official ACP adapters wrap, and they expose more of them than ACP carries today.
3. **Agents are added as config entries first**: a name, a command, its arguments and environment, in the `agent_servers` shape Zed and JetBrains use, so an entry copies between the three tools.
4. **The public ACP registry comes later**, once we have decided how the app pins and checks what it downloads and runs.
5. **Claude or Codex over ACP stays possible as an ordinary entry.** That is how we measure the gap instead of guessing, and how we notice when it closes.

## ACP today

ACP is JSON-RPC between a client (an editor, or our app) and a coding agent, normally over the agent's standard input and output. Zed started it in August 2025; it has since moved to a vendor-neutral organisation, [github.com/agentclientprotocol](https://github.com/agentclientprotocol), which holds the specification, SDKs for five languages, the adapters, a registry and a conformance kit.

- **Version 1 is the stable protocol.** Version 2 is published as a preview and is not frozen.
- **What the agent answers** (v1, stable): `initialize`, `authenticate`, `logout`, `session/new`, `session/load`, `session/resume`, `session/list`, `session/delete`, `session/close`, `session/prompt`, `session/cancel`, `session/set_mode`, `session/set_config_option`.
- **What the client answers**: `session/update` (the stream), `session/request_permission`, `fs/read_text_file`, `fs/write_text_file`, five `terminal/*` methods, and `elicitation/*`. The file and terminal methods are optional: a client says at `initialize` which it serves.
- **What the stream carries**: message and thought chunks, tool calls and their updates, plans, the list of slash commands, mode and config changes, context usage (`usage_update`: tokens used, window size, optionally cost), session info, subagent and compaction updates, notices.
- **Model, effort and mode are "session config options".** The agent lists them with a category (`model`, `thought_level`, `mode`) and the client sets one with `session/set_config_option`. There is no separate "set model" call.
- **Permissions** come as a request with options the agent chooses, each of one kind: allow once, allow always, reject once, reject always.
- **Sign-in** is a list of methods the agent offers at `initialize`. One kind is handled inside the protocol; the other, `terminal`, asks the client to run the agent's own login in a terminal.
- **MCP servers** are handed over at `session/new`: stdio always, HTTP and SSE when the agent says it supports them.
- **Still unstable or in proposal**: forking and archiving sessions, querying sign-in state, MCP served over the ACP connection itself, proxy chains, and a network transport (HTTP and WebSocket) for remote agents.

### Adapters for Claude and Codex

Neither CLI speaks ACP by itself today; each has an official adapter.

| Agent | Package | What it wraps |
|---|---|---|
| Claude | `@agentclientprotocol/claude-agent-acp` | the Claude Agent SDK |
| Codex | `@agentclientprotocol/codex-acp` | the Codex App Server (`codex app-server`) |

The earlier `@zed-industries/claude-code-acp` and `@zed-industries/codex-acp` are deprecated in favour of these. Both wrap exactly what our own plugins drive directly. Both also document a long list of "AIR extensions": private additions under `_meta.jetbrains.air` that give JetBrains Air more than the standard protocol carries (compact diffs, permission presentation, recommended model and effort values, goals, structured failures).

### Agents that speak ACP themselves

From the registry: OpenCode (`opencode acp`), Gemini CLI (`gemini --acp`), GitHub Copilot CLI (`copilot --acp`), Cursor (`cursor-agent acp`), Junie (`junie --acp=true`), Goose (`goose acp`), Auggie (`auggie --acp`), Cline, Qwen Code, Kiro, Kilo, Mistral Vibe, Devin, Amp and others.

### The registry

[github.com/agentclientprotocol/registry](https://github.com/agentclientprotocol/registry) publishes one JSON file, `https://cdn.agentclientprotocol.com/registry/v1/latest/registry.json`, listing 46 agents on the day of writing. Each entry has an id, a version and a way to start it: an `npx` package, a `uvx` package, or a binary archive per platform. Versions are refreshed by an hourly job. Only agents that offer a sign-in method are admitted. A checksum on a binary is optional, and about a quarter of the binary entries have none.

## How JetBrains Air uses it

- Air in IDEs "communicates with ACP-compatible agents through the Agent Client Protocol", runs them on the user's machine, detects agents that are already installed, and installs others from a curated catalog. The registry publishes an index for Air specifically.
- A custom agent is an entry in `~/.jetbrains/acp.json`:

```json
{
  "agent_servers": {
    "My Agent": {
      "command": "/absolute/path/to/your-agent",
      "args": ["acp"],
      "use_idea_mcp": false,
      "use_custom_mcp": true
    }
  }
}
```

The two switches say whether the agent gets the IDE's own tools and the user's configured MCP servers.
- A session runs in the project folder, in a new Git worktree, or in the cloud (Air Teams, on JetBrains' infrastructure, where custom agents are not available). A preset saves agent, model, effort and mode together.
- The first-party list is Claude Agent, OpenAI Codex, GitHub Copilot and Junie.
- On Claude sign-in, JetBrains' documentation says a Claude subscription works only in Air's terminal surface, and that the chat needs API credits: "The chat relies on an SDK integration, and Anthropic terms do not allow subscriptions to be used this way."

What the standard protocol does not carry, Air gets through its private extensions in the two adapters. A client that is not Air does not get them.

## How Zed uses it

- **One thread model.** Zed "updated [its] own in-process agent to use the same code paths as third-party agents", so every interface element built for its own agent is available to external ones.
- **Custom agents** go in settings, in the same shape as Air's:

```json
{
  "agent_servers": {
    "my-agent": {
      "type": "custom",
      "command": "node",
      "args": ["~/projects/agent/index.js", "--acp"],
      "env": {}
    }
  }
}
```

- **The registry is the main way to install** an agent. Zed caches it and refreshes it at most once an hour, runs `npx` agents on a Node it manages itself, keeps each installed version in its own folder so a running agent is never replaced under it, and tells the user when a newer one is available.
- **Model, mode and effort** are shown from the agent's config options, and a default or favourite is saved per agent.
- **A protocol log** (`dev: open acp logs`) shows the traffic between Zed and an agent.
- **Documented limits for external agents**: restoring threads from history and showing token usage "depend on the agent integration"; steering is only for Zed's own agent; Zed's profiles and skills do not apply; the agent owns its sign-in, billing and model list; MCP servers "may be forwarded", and the agent may also read its own.
- **Recurring complaints** in Zed's issue tracker about Codex over ACP: tool cards that do not show the command about to run or the file being edited, "follow the agent" not following, and a broken install with no way to reinstall or sign in again from the interface.

## Where SpecDriven stands

The app does not know any agent's protocol. Every agent is a plugin that implements one contract, `AgentProvider` and `AgentSession` in `src/plugins/host/agents.ts`, and reports back through `AgentHost`. Four agent plugins exist:

| Plugin | How it talks to the agent |
|---|---|
| Claude | the Claude Agent SDK, in process |
| Codex | `codex app-server`, JSON-RPC, our own client |
| Pi | Pi's RPC mode |
| OpenCode | **ACP**, `opencode acp`, our own client |

So ACP is already in use, for one agent. That client is written by hand with no dependency. About a third of it is generic (the transport and the translation of updates into the pane's events); the rest is specific to OpenCode. It is also partial. It calls `initialize`, `session/new`, `session/prompt`, `session/cancel` and `session/set_config_option`, and answers `session/update` and `session/request_permission`. It does not sign in, list or resume sessions, or serve files and terminals, and it drops thoughts and plans.

## What the contract needs, and who provides it

| The pane needs | ACP v1 | Claude plugin | Codex plugin | OpenCode plugin |
|---|---|---|---|---|
| Start in a folder, send text and images | yes | yes | yes | yes |
| Streamed answer | yes | yes | yes | yes |
| Thinking | yes, thought chunks | yes | no | dropped |
| Stop | yes | yes | yes | yes |
| Permission cards | yes, the agent picks the options | yes, with editable input | yes | yes |
| Remembered answers | "allow always", meaning set by the agent | as a rule | for the session | as a rule |
| Modes | the agent names its own | our five | mapped to approval and sandbox | mapped to four of ours |
| Model list and switch | yes, a config option | yes, from the account or the API | yes | yes |
| Effort | yes, a config option | yes | yes | yes |
| Context usage | totals, optional | breakdown by category | totals | totals |
| Slash commands | yes | yes | no | yes |
| MCP servers handed to a session | yes | yes | yes | yes |
| MCP server list, on and off, sign-in | not in stable v1 | yes | no | no |
| Sign-in from the app | methods listed by the agent, or a terminal login | browser code flow, account or API | ChatGPT account or API | a hint to run a command |
| One-shot answer without tools | no | yes | yes | yes, outside ACP |
| Reconnect after the API fails | no | yes | no | no |

## What ACP would add

- **Reach.** Gemini CLI, Copilot CLI, Cursor, Junie, Goose, Auggie and the rest without a plugin each.
- **Plans and todo lists**, thoughts, and tool calls that carry a diff instead of raw input.
- **Sign-in the agent describes**, including the terminal login, in place of a hint written per plugin.
- **Session list and resume**, which the pane does not have for any agent.
- **A shared test surface**: the conformance kit and the official SDKs.

## What moving Claude and Codex would cost

- **Things the protocol has no place for.** The context breakdown by category (ACP reports totals), the MCP server list with on, off and sign-in as data the pane can show (the Claude adapter offers it as a `/mcp` chat command), the one-shot answer, and the reconnect after the API fails. These would stay outside ACP or be lost.
- **Things we would rebuild on what the adapter offers.** Sign-in in Settings: the browser code flow and the account-or-API choice for Claude, the ChatGPT-account-or-API choice for Codex. The Codex adapter lists ChatGPT login, an API key and a custom gateway as its methods; what the Claude adapter offers a client that is not Air needs a trial. The model list would come from the adapter's config options, not from the account or the API as it does now.
- **A second moving part.** The adapter trails the CLI, and a good share of what it offers beyond the standard is reserved for Air through private extensions.

None of this is permanent. The adapters are improving quickly, and version 2 of the protocol covers more.

## What the app would need for a generic ACP plugin

Named here, not designed.

- **Modes named by the agent.** The pane has a fixed set of five permission modes; an ACP agent brings its own list.
- **Capabilities learned when the session starts.** A provider declares them up front today; an ACP agent reveals its models, modes and effort levels only after `session/new`.
- **Several agents from one plugin.** A marketplace row names one agent and the one command it needs.
- **A plan item and a diff view in the transcript.** Tool rows show their input as JSON.
- **Terminal sign-in**, mapped onto the login hint the pane already shows.
- **A place for the entries**: a settings list in the `agent_servers` shape, per device.
- **The generic part of the OpenCode client moved to shared code**, with the OpenCode plugin rebuilt on it, so there is one ACP client and not two.
- **A protocol log**, as Zed has, for the day an agent misbehaves.

## Suggested order

1. Move the generic ACP client out of the OpenCode plugin, and add what it skips: sign-in methods, thoughts, plans.
2. A generic ACP plugin with config entries; try it against agents already installed (`auggie --acp`, `opencode acp`).
3. The contract changes above, one at a time, each driven by a real agent that needs it.
4. Claude and Codex over ACP as entries, and a measured comparison against the native plugins.
5. The registry: browse and install, after a decision on pinning and checksums.

## Not verified

- Whether a Claude subscription may be used through an SDK-based client. JetBrains says Anthropic's terms do not allow it in Air's chat; we have not read the terms ourselves. Check before relying on account sign-in over ACP.
- Whether the Claude or Codex CLIs will speak ACP themselves and make the adapters unnecessary. No source says either way.
- How much of the two adapters works for a client that does not declare itself as Air. Only a trial will tell.
- What version 2 will look like when it is frozen.

## Sources

- Protocol: [agentclientprotocol.com](https://agentclientprotocol.com), and the method lists in [schema/v1](https://github.com/agentclientprotocol/agent-client-protocol/tree/main/schema/v1)
- Adapters: [claude-agent-acp](https://github.com/agentclientprotocol/claude-agent-acp), [codex-acp](https://github.com/agentclientprotocol/codex-acp)
- Registry: [agentclientprotocol/registry](https://github.com/agentclientprotocol/registry)
- Conformance kit: [agentclientprotocol/acp-tck](https://github.com/agentclientprotocol/acp-tck)
- JetBrains Air: [About Air in IDEs](https://www.jetbrains.com/help/air/air-in-ides-overview.html), [Manage agents](https://www.jetbrains.com/help/air/air-in-ides-manage-agents.html), [Start a session](https://www.jetbrains.com/help/air/air-in-ides-start-a-session.html), [Supported agents](https://www.jetbrains.com/help/air/supported-agents.html), [Claude subscription](https://www.jetbrains.com/help/air/air-in-ides-claude-subscription.html)
- Zed: [External Agents](https://zed.dev/docs/ai/external-agents), [Agent Panel](https://zed.dev/docs/ai/agent-panel), [Bring Your Own Agent to Zed](https://zed.dev/blog/bring-your-own-agent-to-zed), issues [65727](https://github.com/zed-industries/zed/issues/65727), [65864](https://github.com/zed-industries/zed/issues/65864), [62654](https://github.com/zed-industries/zed/issues/62654), [64680](https://github.com/zed-industries/zed/issues/64680)
