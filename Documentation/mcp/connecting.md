---
title: Connect a client to Direct over MCP
description: Create a personal access token in Direct, configure an MCP client with it as a bearer token, keep it out of repositories and prompts, revoke it, and fix a token that is rejected.
---

An MCP client connects to Direct with a personal access token. This page shows how to create the token, configure a
client, keep the token safe and fix the common failures. See [Tools](./tools.md) for what the client can then do.

## Before you start

- You are an active member of the Direct organization you want to work in. A token only works for an active member.
- MCP access is turned on for your Direct. If it is not, the MCP address answers `404`. Ask whoever runs your Direct.
- You know your Direct address. The MCP endpoint is the path `/mcp` on that address, shown below as
  `https://<your Direct address>/mcp`. It uses streamable HTTP.

## Create a personal access token

1. Sign in to Direct and open **My profile**.
2. In the **Personal access tokens** card, type what the token is for in the name box, for example
   `Claude Code on my laptop`.
3. Choose how long it works: **7 days**, **30 days** or **90 days**. A token never works for longer than 90 days.
4. Under **What it may do**, select the scopes the agent needs and no more. **Read** is selected for you.
5. Select **Create token**.
6. Select **Copy** while the message **Copy the token now - it is not shown again.** is showing. Direct stores only a
   hash of the token. If you lose it, revoke it and create another.

The token acts as you, in the organization it was created in.

### Scopes

| Option in Direct | Scope | What it allows |
| --- | --- | --- |
| **Read** | `direct:read` | Read work, content and status through the API and MCP. |
| **Write content** | `direct:content.write` | Draft and revise content as you. |
| **Do work** | `direct:work` | Start and steer work as you. |
| **Request changes** | `direct:write` | Admit journeys, file issues and answer questions as you. Never approves a request. |

[Tools](./tools.md) names the scope each tool needs.

## Connect your client

Every client sends the token in an `Authorization` header:

```text
Authorization: Bearer <personal access token>
```

Put the token in an environment variable and have the client read it from there. The examples use `DIRECT_TOKEN`.
Read it without echoing it so it stays out of your shell history, or load it from your password manager:

```bash
read -rs DIRECT_TOKEN && export DIRECT_TOKEN
```

### Claude Code

Add the server to a `.mcp.json` file. Claude Code expands `${DIRECT_TOKEN}` when it reads the file, so the token
itself is not in the file:

```json
{
  "mcpServers": {
    "direct": {
      "type": "http",
      "url": "https://<your Direct address>/mcp",
      "headers": { "Authorization": "Bearer ${DIRECT_TOKEN}" }
    }
  }
}
```

### Codex

Add the server to `~/.codex/config.toml` and name the variable that holds the token:

```toml
[mcp_servers.direct]
url = "https://<your Direct address>/mcp"
bearer_token_env_var = "DIRECT_TOKEN"
```

### Other clients

Any MCP client that supports streamable HTTP and a static bearer header works. Configure the URL and the
`Authorization` header as above. A client that can only sign in through a browser cannot connect, because Direct
does not run a sign-in for MCP clients.

## Verify the connection

Ask the client to list its MCP tools. Tool names start with `direct_`. Read tools such as `direct_my_work` and
`direct_knowledge_search` need only the **Read** scope.

## Keep the token safe

- Never commit a token to a repository, and never put its value in a configuration file you commit. Refer to an
  environment variable instead.
- Never paste a token into a prompt or a chat with an agent. The agent does not need to see it; the client sends it.
- Choose the shortest lifetime and the fewest scopes that work.
- Create one token per client and machine, so you can revoke one without breaking the others.

## Revoke a token

Open **My profile**, find the token in **Personal access tokens** and select **Revoke**. Confirm **Revoke** for that
token. Anything using it stops working. Direct remembers a successful check for up to 60 seconds, so a revoked token
can still be accepted for up to a minute. An expired token shows **Expired** in the list and stops working.

## Troubleshooting

| What you see | What it means and what to do |
| --- | --- |
| `404` from the MCP address | The address is wrong, or MCP access is not turned on for your Direct. Check the address, then ask whoever runs your Direct. |
| `401` | There is no token, or Direct does not accept it: it is mistyped, expired or revoked. Check that `DIRECT_TOKEN` is set in the environment the client starts from. Create a new token if needed. |
| `403` with `insufficient_scope` | The tool needs a scope the token does not have. The response names it. Create a new token with that scope. |
| `403` with "You are not an active member" | A token only works for an active member of the organization. Ask an administrator to invite you, accept the invitation in Direct, then try again. |
| `429` or `503` | Direct is checking too many tokens at once. Wait a moment and try again. |
| The client asks you to sign in | The client expects a browser sign-in. Configure the bearer header as shown above. |
