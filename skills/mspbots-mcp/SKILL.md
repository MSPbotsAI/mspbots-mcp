---
name: mspbots-mcp
description: Access MSPbots platform data through the MSPbots MCP server. Use when the user wants to query, search, aggregate, or analyze MSPbots data (datasets, integrations, business metrics). Guides the agent to install and connect the MCP first, then discover its tools.
---

# MSPbots MCP

## Overview

MSPbots MCP is the data gateway to the MSPbots platform. Through it you can:

- Discover and search the user's datasets
- Query dataset data and run aggregations
- Explore the integrations installed in the user's MSPbots environment
- Work with MSPbots business data on the user's behalf

**Important:** Do NOT assume specific tool names or parameters. After connecting to the MCP server, read its tool list and tool descriptions to learn what is available and how to call it.

**Important:** Token acquisition MUST be completed by the main agent BEFORE any other step of this skill runs (including any subagent work). Both token options require direct user interaction (asking the user, or waiting for the user to authorize in the browser), which subagents cannot do. Once obtained, the token MUST be saved to the token config file described below so that subsequent steps and future sessions can reuse it.

## Token config file

The token is persisted in the skill directory at:

```
<skill-dir>/config/token.json
```

Format:

```json
{
    "token": "<MSPbots access token>",
    "saved_at": "<ISO-8601 UTC timestamp, e.g. 2026-07-06T08:30:00Z>"
}
```

Rules:

- Before acquiring a token, first check this file. If it exists and the token still works (the MCP server accepts it), reuse it and skip acquisition.
- After acquiring a new token (either option), write/overwrite this file immediately.
- If the MCP server rejects the stored token (401/unauthorized), delete or overwrite the file and re-run token acquisition.
- This file contains a secret: never commit it to git (ensure `config/token.json` under the skill directory is gitignored) and never print the token in output.

## Prerequisite: install the MSPbots MCP server

Before doing any MSPbots data work, verify that the "MSPbots MCP" server is connected. If it is not, install it with the following configuration:

```json
{
    "mcpServers": {
        "mspbots-mcp": {
            "type": "http",
            "url": "https://owl.mspbots.ai/data-cli/mcp/",
            "headers": {
                "Authorization": "Bearer <TOKEN>"
            }
        }
    }
}
```

Replace `<TOKEN>` with an MSPbots access token obtained as described below.

## Getting the TOKEN

### Option 1: ask the user

Ask the user to provide their MSPbots platform token directly.

### Option 2: browser authorization flow

**How this flow works (read carefully):**

- The authorization page will NEVER display a token to the user. Do NOT expect the user to copy/paste a token from the page.
- Do NOT start any polling loop or background process. The token is fetched by YOU (the agent) in a single request, but only AFTER the user confirms they have authorized.
- The flow is strictly: generate code → show auth URL to user → user authorizes in browser → user tells you they are done → you fetch the token once with the code.

**Failure handling:** run this flow at most once. If the token fetch fails or any step fails, do NOT retry or restart the flow — fall back to Option 1 and ask the user to provide their MSPbots token directly.

1. Generate a one-time code and build the authorization URL:

```python
import uuid

code = uuid.uuid4().hex
auth_page_url = f"https://app.mspbots.ai/auth-data-cli?code={code}"
```

2. **Display the full authorization URL to the user in your reply** (this is mandatory — the user must be able to see and click/copy it). Depending on the environment, you may additionally try to open it in the user's browser (e.g. `Start-Process <url>` on Windows, `open <url>` on macOS, `xdg-open <url>` on Linux), but showing the URL in text is always required in case auto-open fails.

3. Ask the user to click "Authorize" on that page, then **stop and wait**. Do not run any command while waiting. Resume only when the user replies confirming that authorization is complete.

4. After the user confirms, fetch the token with a single request (no polling):

```python
from urllib import request

TOKEN_URL_TEMPLATE = "https://owlstg.mspbots.ai/owl-agent/api/v1/auth_token/{code}"

def fetch_auth_token(code: str) -> str:
    with request.urlopen(TOKEN_URL_TEMPLATE.format(code=code), timeout=10) as resp:
        if resp.status != 200:
            raise RuntimeError(f"Token fetch failed with status {resp.status}.")
        return resp.read().decode("utf-8", errors="ignore")
```

If this request fails (non-200, network error, or empty token), do not retry — fall back to Option 1.

## Workflow

1. **(Main agent only)** Ensure a valid token is available: read `<skill-dir>/config/token.json`; if missing or invalid, acquire a token via Option 1 or Option 2 and save it to that file. Do not proceed until this step succeeds.
2. Check that the MSPbots MCP server is connected; if not, install it as above using the token from the config file.
3. Read the MCP server's tool list to discover its capabilities.
4. Use the discovered tools to fulfill the user's data request.

## Claude post-install starter prompts version
The following examples must be list after successful installation.

Try asking:
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
