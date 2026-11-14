---
title: Connect to Direct over MCP
description: Let an AI agent or script on your machine work in Direct as you, over the Model Context Protocol (MCP), with a personal access token that carries only the scopes you choose.
---

Direct serves a set of tools to AI agents over the Model Context Protocol (MCP). An agent that connects can read
your work, search product knowledge and, when you allow it, file issues and comment on them. It acts as you: in the
organization you created its token in, and never with more rights than you have.

## Who this is for

Use MCP access when you run a coding agent, such as Claude Code, Codex or pi, or a script on your own machine and want
it to use Direct without you copying data between them. You do not need the [Direct Agent Runner](../runner/index.md)
for this. The runner executes Direct's work on your machine; MCP access lets your own agent call Direct.

## How it works

1. You create a personal access token in Direct and choose its scopes and lifetime.
2. You give the token to your MCP client as a bearer token.
3. The client lists Direct's tools. Each tool needs a scope, and Direct refuses a call the token's scopes do not allow.
4. Every call that changes something needs an idempotency key, so a retry does not repeat the change.

A token never approves anything. A request for approval always waits for a person in Direct.

## Pages in this section

| Page | What it covers |
| --- | --- |
| [Connect a client](./connecting.md) | Create a token, point your client at Direct, keep the token safe, revoke it and fix a rejected token. |
| [Tools](./tools.md) | The tools Direct offers over MCP, and the scope each one needs. |
