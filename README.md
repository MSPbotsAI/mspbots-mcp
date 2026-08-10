# MSPbots MCP — Claude Code plugin

A Claude Code skill that connects the [MSPbots](https://mspbots.ai) DataCLI MCP server and lets an agent query, search, aggregate, and analyze MSPbots platform data (datasets, integrations, business metrics) on your behalf.

This repository is a Claude Code **plugin marketplace**. Installing it adds the `mspbots-mcp` skill to your agent.

## Install

In Claude Code, run:

```
/plugin marketplace add MSPbotsAI/mspbots-mcp
/plugin install mspbots-mcp@mspbots-mcp
```

That's it — the `mspbots-mcp` skill is now available. The next time you ask the agent to work with MSPbots data, it will follow the skill to install/connect the MCP server and use its tools.

## Requirements

- A valid MSPbots access token. You don't need to fetch it manually — the skill acquires it interactively (either by asking you for it, or via the MSPbots browser authorization flow) and reuses it on later runs.

## What's in here

```
.claude-plugin/
  marketplace.json   # marketplace manifest
  plugin.json        # plugin manifest
skills/
  mspbots-mcp/
    SKILL.md         # the skill
```
