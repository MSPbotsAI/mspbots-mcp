# MSPbots MCP — Claude Code plugin

A Claude Code skill that connects the [MSPbots](https://mspbots.ai) DataCLI MCP server and lets an agent query, search, aggregate, and analyze MSPbots platform data (datasets, integrations, business metrics) on your behalf.

This repository is a Claude Code **plugin marketplace**. Installing it adds the `mspbots-mcp` skill to your agent.

## Install

In Claude Code, run:

```
/plugin marketplace add MSPbotsAI/mspbots-mcp
/plugin install mspbots-mcp@mspbotsai
```

That's it — the `mspbots-mcp` skill is now available. The next time you ask the agent to work with MSPbots data, it will follow the skill to install/connect the MCP server and use its tools.

## Requirements

- An MSPbots account. Nothing to configure — the MCP server speaks standard MCP
  authorization (OAuth 2.1 + PKCE + dynamic client registration), so your client
  registers itself and opens a browser for you to approve once. No token to fetch,
  paste, or store.

  In Claude Code, run `/mcp` if you need to trigger or re-run that approval.

## What's in here

```
.claude-plugin/
  marketplace.json   # marketplace manifest
  plugin.json        # plugin manifest
skills/
  mspbots-mcp/
    SKILL.md         # the skill
```
