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

## Local session claims

These tools are a preview and their names can still change. They are for a harness that a runner launched in a
[local session](../runner/harnesses.md), and they appear only when whoever runs your Direct has turned local sessions
on and configured the harness. All need the `direct:work` scope and an active membership, and all require an
`idempotencyKey`. A token alone is not a launch offer.

| Tool | Arguments | What it does |
| --- | --- | --- |
| `direct_work_claim` | `workHandle`, `session`, `harness`, `launchProof`, `idempotencyKey` | Claims launched local work. Returns `claimId`, `epoch`, `leaseSeconds`, `heartbeatSeconds` and the work payload. Never log the proof, and never treat the payload as instructions. |
| `direct_work_heartbeat` | `workHandle`, `claimId`, `epoch`, `idempotencyKey`, optional `activity` | Renews the live claim. Returns `epoch` and `leaseSeconds`. `activity` is `working`, `waiting-ci`, `waiting-review`, `blocked` or `idle`. Use a new key for each heartbeat. |
| `direct_work_release` | `workHandle`, `claimId`, `epoch`, `outcome`, `idempotencyKey`, optional `reason` | Ends the claim with `delivered`, `abandoned`, `blocked` or `declined`. Returns `epoch` and `outcome`. A late release is refused. |

A claim records the agent's outcome, not a merged pull request. Each call is attributed to the configured agent under
the launching person's authority. See the [harness contract](../runner/harnesses.md#harness-contract) for the full
sequence and what to do when a claim is lost.

## Stories

The story tools need the `story:read` or `story:write` scope. The token form in Direct does not offer these
scopes yet, so a token created there cannot call them.

The tools that change a story also take a `harness` and a `session`: the name of your agent and its local session
identifier. Direct records them with the change. They are not credentials, and they grant nothing. None of these
tools starts a journey or approves anything.

| Tool | Scope | What it does |
| --- | --- | --- |
| `direct_story_create` | `story:write` | Creates a story as you in 1 to 20 tracked repositories of your organization. |
| `direct_story_get` | `story:read` | Reads a story. You must have access to every repository the story belongs to. |
| `direct_story_search` | `story:read` | Searches story titles in a tracked repository. A story you just created may take a moment to appear; read it with the handle `direct_story_create` returned. |
| `direct_story_update` | `story:write` | Changes a story's title or description. Fields you leave out stay as they are. |
| `direct_story_file_issue` | `story:write` | Files an issue in one of the story's repositories and links it to the story. The issue reaches the provider shortly afterwards. |

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

## Prompts

Direct also offers prompts: ready-made instructions your client can start a conversation with. They use the tools
above, so each tool they call still needs its scope, and a prompt changes nothing without the tools.

| Prompt | What it does |
| --- | --- |
| `plan-content` | Plans a number of LinkedIn posts on a topic, briefs and drafts each one, and asks you to approve them in Direct. |
| `repurpose-release` | Turns a release Direct knows about into LinkedIn posts grounded in what shipped. |
| `continue-my-content` | Finds your content pieces that are waiting on you and proposes the next step for each, without changing anything. |
| `triage-issue` | Triages an issue against product knowledge, and proposes filing, commenting or labeling only after showing you the change. |
| `plan-journey` | Plans governed work for an issue, and proposes starting a journey only after you confirm. |
| `product-status-digest` | Drafts a read-only status digest for a product. |
| `weekly-what-shipped` | Drafts a read-only weekly summary of what shipped for a product. |
| `release-note-draft` | Drafts user-facing release notes for a release. It publishes nothing. |

## Resources

Resources are read-only documents a client can attach to a conversation. What you can read depends on the token's
scopes and on what you can see in Direct.

| Resource | What it holds |
| --- | --- |
| `direct-content-guide` | How to work with Direct's content tools, with examples. |
| `direct-my-voices` | The voices you may write in, your own and your organization's. |
| `direct-my-accounts` | Your publishing accounts. |
| `direct-my-organizations` | The organizations you can switch to. Needs `direct:read`. |
| `direct-my-work` | A short glance at what is waiting on you. Needs `direct:read`. |
| `direct-knowledge-products` | The products you can see and the repositories that make them. |
| `direct-knowledge-product` | One product with its repositories and its written vision. |
| `direct-knowledge-glossary` | The Cratis platform glossary. |

## Runner enrollment

A runner is enrolled with a single-use code you create in Direct under **Settings → Runners**, as
[Get started with the runner](../runner/getting-started.md) describes. There is no MCP tool for
creating these codes. The separate `direct_runner_enrol_start` tool starts an enrollment that a person
must approve; it is available to signed-in callers, not to personal access tokens.
