---
title: Direct MCP tools
description: The tools Direct offers to MCP clients, grouped by what they do, with the token scope each one needs and whether it changes anything.
---

Each tool needs the scope shown below. Direct refuses a call whose token lacks the scope, and depending on how your
Direct is set up the client may not list that tool at all. To change what a client can do,
[create a token](./connecting.md#create-a-personal-access-token) with different scopes. Scopes are named in
[Connect a client](./connecting.md#scopes).

Every tool that changes something takes an `idempotencyKey`. Use a new key for each intended change, and reuse the same
key only to retry the same call. Tools that list or search return a `nextCursor` while more results remain, and take a
`limit` of 1 to 50. No tool approves anything. Where a change needs a person's approval, the tool returns a link for
that person to follow in Direct.

## Read

These tools need the `direct:read` scope and change nothing.

| Tool | What it does |
| --- | --- |
| `direct_my_work` | Lists what is waiting on you. |
| `direct_knowledge_search` | Searches the products, visions and repositories you can see, and the Cratis glossary. |
| `direct_work_search` | Searches work by repository, stage, status and more. |
| `direct_journey_get` | Reads one journey. |
| `direct_gate_latest` | Reads the latest gate evaluation of a journey. |
| `direct_evidence_list` | Lists the evidence behind an evaluation. |
| `direct_evidence_get` | Reads the metadata of one piece of evidence. |
| `direct_issue_read` | Reads an issue's body, labels and story as Direct holds them. |
| `direct_issue_list` | Lists the issues of one story, delivery or journey. Listing a story also needs the story read scope, which the token form does not offer. |
| `direct_runners_list` | Lists the runners that take your work, with state, health and load. |
| `direct_work_where` | Shows where work you started runs. |
| `direct_runners_eligible` | Explains which runners are eligible for work you can see. |
| `direct_runners_security` | Reads a runner's security state. |
| `direct_runners_launches` | Reads the audit of work launched on a runner. |

## Issues and work

| Tool | Scope | What it does |
| --- | --- | --- |
| `direct_issue_file` | `direct:write` | Files an issue in a repository of your organization. It checks for likely duplicates first. With no likely duplicates it files the issue; otherwise it returns the candidates and files nothing until you call it again to link or file anyway. `direct:work` is also accepted, but is deprecated for this tool. |
| `direct_issue_comment` | `direct:work` | Comments on an issue. |
| `direct_issue_label` | `direct:work` | Sets the complete list of labels on an issue. Labels you leave out are removed, so read the existing labels first. |
| `direct_journey_start` | `direct:write` | Asks Direct to start governed work on an issue. If the request needs a higher ceiling than you may set, nothing starts and the tool returns a link for an eligible person to approve. `direct:work` is also accepted, but is deprecated for this tool. |
| `direct_gate_explain` | `direct:write` | Supplies the explanation a gate requirement asked for. `direct:work` is not accepted. |
| `direct_question_answer` | `direct:write` | Answers an open agent question waiting on you. `direct:work` is not accepted. |
| `direct_runner_hold_request` | `direct:write` | Asks to pause or drain a runner. Nothing changes until a different eligible person approves in Direct. |

## Content

| Tool | Scope | What it does |
| --- | --- | --- |
| `direct_context_get` | `direct:read` | Reads your content context. |
| `direct_content_search` | `direct:read` | Searches your content pieces. |
| `direct_content_get` | `direct:read` | Reads one content piece. |
| `direct_content_draft_status` | `direct:read` | Checks whether a requested draft is ready. |
| `direct_content_approval_status` | `direct:read` | Checks an approval request. |
| `direct_content_suggest_from_shipped` | `direct:read` | Suggests content from recorded releases, each citing its sources. |
| `direct_content_create_brief` | `direct:content.write` | Briefs a new content piece. |
| `direct_content_revise` | `direct:content.write` | Revises the text of a draft. |
| `direct_content_request_approval` | `direct:content.write` | Asks the author to approve a piece. It does not approve it. |
| `direct_content_handoff` | `direct:content.write` | Hands off approved copy. |
| `direct_content_record_published` | `direct:content.write` | Records the URL where a piece was published. |
| `direct_content_request_draft` | `direct:content.write` and `direct:work` | Requests a generated draft. It starts work that uses AI capacity, so it needs both scopes. |

## Runner enrollment

Connecting a runner to Direct is done in the browser, as [Get started with the runner](../runner/getting-started.md)
describes. There is no MCP tool for everyone to do it.
