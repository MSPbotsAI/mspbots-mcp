---
name: mspbots-mcp
description: Access MSPbots platform data through the MSPbots MCP server. Use when the user wants to query, search, aggregate, or analyze MSPbots data (datasets, integrations, business metrics). Guides the agent to connect the MCP first, then discover its tools.
---

# MSPbots MCP

## Overview

MSPbots MCP is the data gateway to the MSPbots platform. Through it you can:

- Discover and search the user's datasets
- Query dataset data and run aggregations
- Explore the integrations installed in the user's MSPbots environment
- Work with MSPbots business data on the user's behalf

**Important:** Do NOT assume specific tool names or parameters. After connecting to the MCP server, read its tool list and tool descriptions to learn what is available and how to call it.

## Prerequisite: connect the MSPbots MCP server

Before doing any MSPbots data work, verify that the "MSPbots MCP" server is connected. If it is not, add it with:

```json
{
    "mcpServers": {
        "mspbots-mcp": {
            "type": "http",
            "url": "https://owl.mspbots.ai/data-cli/mcp/"
        }
    }
}
```

That is the whole configuration. **No token, no headers.**

Two details worth not "fixing":

- **Keep the trailing slash on `/mcp/`.** Without it the server answers 307 to the
  slashed form, and a POST that crosses a redirect can lose its `Authorization`
  header in some clients.
- **No `headers` block.** The server is an OAuth 2.1 resource server; your client
  handles authorization itself (next section). Pasting a token in here is the
  fallback path, not the normal one.

## Authorization

The server implements standard MCP authorization — OAuth 2.1 with PKCE and dynamic
client registration (RFC 7591). **Your client does this for you.** You do not
generate codes, poll anything, store a token, or write a credential to disk.

What happens on first connect:

1. The server answers `401` with a `WWW-Authenticate` header pointing at its metadata.
2. Your client discovers the authorization server, registers itself, and opens a
   browser page for the user to approve.
3. The user approves once. Your client keeps the token and refreshes it on its own.

So: **add the server, then let the client connect.** If the user has not authorized
yet, surface whatever authorization prompt your client raises (in Claude Code, `/mcp`)
and wait for them to finish in the browser. Do not try to run the flow by hand.

### Fallback: client does not support OAuth

Only if your MCP client cannot do OAuth at all. Ask the user for their MSPbots
platform token and add it as a header:

```json
{
    "mcpServers": {
        "mspbots-mcp": {
            "type": "http",
            "url": "https://owl.mspbots.ai/data-cli/mcp/",
            "headers": { "Authorization": "Bearer <TOKEN>" }
        }
    }
}
```

The server accepts both schemes, so this keeps working. It is worse for the user —
the token does not refresh, and it ends up in a config file — which is why it is the
fallback and not the default. Never print the token in output.

## Workflow

1. Check that the MSPbots MCP server is connected; if not, add it as above.
2. If the client reports it is unauthorized, prompt the user to authorize and wait.
3. Read the MCP server's tool list to discover its capabilities.
4. Use the discovered tools to fulfill the user's data request.

## Claude post-install starter prompts
The following examples must be listed after successful installation. 10 of 10 must be listed.

1. Show time records from the last 30 days with missing notes.
2. Find records where billed effort does not match actual effort.
3. Show records with missed response targets in the last 30 days.
4. List unresolved records older than 7 days.
5. Show workload by assignee for the last 30 days.
6. Which team members are handling the most complex work?
7. Find devices not seen in the last 7 days.
8. Show devices missing expected security coverage.
9. List offline alerts from the last 30 days.
10. Summarize offline events by site.
