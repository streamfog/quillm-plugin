<img src="assets/logo.png" alt="" width="72">

# Quillm plugin

[Quillm](https://quillm.ai) keeps the small tools your agents build for your team: dashboards, trackers, decision docs and calculators. Each page lives at one address your team opens like a doc, with no AI account needed. The numbers live in datasets apart from the page, so an agent keeps a page current by adding rows, run after run, without rewriting it. Every save is test-rendered before readers see it, every revision is kept, and every change is signed by the agent that made it.

This repository connects your agent to Quillm. It holds:

- the connection to Quillm's hosted MCP server, `https://quillm.ai/mcp`, which you sign in to with OAuth in your browser;
- one skill, [`skills/quillm/SKILL.md`](skills/quillm/SKILL.md), that teaches the agent the habits that keep pages useful for months, including how a scheduled agent should add each run's numbers without touching the page.

The server itself is hosted by Streamfog and is not part of this repository.

## Install

| App | How |
|---|---|
| Claude Code | `claude plugin marketplace add streamfog/quillm-plugin`, then `claude plugin install quillm@quillm`. Run `/mcp` and choose Authenticate. |
| claude.ai, Claude Desktop, Cowork | Settings → Connectors → find Quillm in the directory, or add a custom connector with `https://quillm.ai/mcp`. |
| ChatGPT, Codex | Find Quillm in the plugin directory. |
| Cursor | Install Quillm from the Cursor Marketplace. |
| Grok Build | Install Quillm from the plugin marketplace. |
| Anything else that speaks MCP | Add `https://quillm.ai/mcp` as a remote (Streamable HTTP) server and sign in. |

Step-by-step instructions for each app: https://quillm.ai/connect

The first time a tool is used, a browser window asks you to sign in to Quillm with Google, choose a workspace (or name a new one), and decide whether the agent may change things or only read. The agent then appears on the workspace's Agents page, where an admin can limit it to some collections, sign an app out, or revoke it.

### Without a browser (CI, cron, servers)

An admin can create a token for an agent on the workspace's Agents page and give it to the job directly, for example:

```sh
claude mcp add --transport http quillm https://quillm.ai/mcp --header "Authorization: Bearer qlm_…"
```

Keep the token in your CI's secret store, not in a repository.

## What the agent can do

Read the workspace, read and write datasets, create and update pages, test-render a page, share a page by link when asked, leave notes for the next agent, and report a problem with Quillm. The full list, with what each tool reads or changes and how much context the server uses, is at https://quillm.ai/connect#tools.

## Privacy

What Quillm stores and what it sends to other services is listed in the [privacy policy](https://quillm.ai/privacy). Workspace admins can export everything at any time.

## Files

| File | For |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code, claude.ai and Cowork; also read by Grok Build |
| `.cursor-plugin/plugin.json`, `mcp.json` | Cursor |
| `.codex-plugin/plugin.json` | ChatGPT and Codex |
| `.grok-plugin/plugin.json` | Grok Build |
| `plugin.json` | Agent Plugins (portable manifest) |
| `server.json` | The Official MCP Registry entry, `ai.quillm/quillm` |

## Support

Write to kev@limelit.ai, or open an issue in this repository.

## License

MIT. See [LICENSE](LICENSE).
