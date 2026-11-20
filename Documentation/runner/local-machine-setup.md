---
title: Set up a machine for local sessions
description: One path from a clean machine to a first piece of dispatched work, in order - what to ask the operator for, a token in the keychain, an MCP client, one runner install, image approval, the GitHub App and a first check.
---

This page puts the steps for a new machine in order and links to the page that owns each one. It does not repeat
them. Follow the steps in order and stop at the first check that fails.

> [!NOTE]
> [Local sessions](./harnesses.md) are a preview. The runner does not yet start a harness itself, so steps 1 to 6
> prepare the machine and the connection, and step 7 checks that work reaches it.

## Who does what

Some steps need a person. An agent may help with the others, but it never needs to see a secret.

| Step | A person must | An agent may |
| --- | --- | --- |
| 1. Ask the operator | Ask whoever runs your Direct | Draft the request |
| 2. Create a token | Create it in Direct, copy it once and store it in the keychain | Run read checks afterward |
| 3. Configure the client | Choose the scopes; keep the token out of committed files | Write the client configuration that refers to an environment variable |
| 4. Install the runner | Create the enrollment code in Direct and run the install | Check `direct-runner status` and `docker info` |
| 5. Approve images | Decide which exact digests to trust | Look up the digest of an image a person names |
| 6. Install the GitHub App | Be an administrator of the repository owner on GitHub | Nothing |
| 7. Check real work | Start the work in Direct | Read where the work runs and how it ended |

Never paste a token into a prompt, a note or a file you commit.

## 1. Ask the operator

Ask whoever runs your Direct to confirm:

- MCP access with personal access tokens is turned on. If it is not, the MCP address answers `404`.
- Local sessions are turned on and a harness is configured for your organization. Without that, Direct refuses a
  claim.
- You are an active member of the organization, and you know your tenant id and your Direct address.

## 2. Create a token and store it in the keychain

Create the token as [Connect a client](../mcp/connecting.md#create-a-personal-access-token) describes. For the
checks on this page, **Read** is enough; local session claims also need **Do work**. Direct shows the token once.

Store it in your operating system keychain instead of a file. These commands are your system's own tools, and each
prompts for the secret so it stays out of your shell history:

```shell
# macOS
security add-generic-password -a "$USER" -s direct-token -w

# Linux with libsecret
secret-tool store --label="Direct token" service direct-token
```

Load it into the environment of the client at start-up, from the keychain:

```shell
# macOS
export DIRECT_TOKEN="$(security find-generic-password -a "$USER" -s direct-token -w)"

# Linux with libsecret
export DIRECT_TOKEN="$(secret-tool lookup service direct-token)"
```

## 3. Configure your MCP client

Configure [Claude Code](../mcp/connecting.md#claude-code), [Codex](../mcp/connecting.md#codex) or
[pi](../mcp/connecting.md#pi) as that page shows. Each reads the token from `DIRECT_TOKEN`, and each sends your
tenant id in `x-cratis-tenant-id`.

Verify the connection by reading your organizations. This is a resource and a route, not a tool, so a tool list will
not show it:

- Read the MCP resource `direct://me/organizations`, or
- send `GET /v1/me/organizations` with the same two headers.

Either needs the **Read** scope and should list your organization. If it fails, use the
[troubleshooting table](../mcp/connecting.md#troubleshooting).

Your client may list fewer tools than [Tools](../mcp/tools.md) describes. A deployment can limit personal access
tokens to a reviewed set of tools, and the client lists only what your token may call. Tools that are not listed are
not available to the token.

## 4. Install the runner

Choose exactly one way to run the runner on a machine, and never both:

- a package-manager service (`brew services start direct-runner`), or
- the `direct-runner install` command for a downloaded runner.

[Install the runner](./installing.md) describes both, what `install` refuses, and how to switch. Then enroll the
runner with a single-use code from **Settings → Runners → Add runner**, as
[Get started](./getting-started.md#enroll-without-a-browser) describes. Run `direct-runner status` to confirm it is
connected. Start the foreground command only when no daemon is running.

## 5. Approve worker images by exact digest

Enrollment does not allow any worker image. Allow each image by its exact repository and SHA-256 digest with
`--allow-image <repository@sha256:digest>`, as
[Install the runner](./installing.md#enroll-the-runner-with-a-code) describes. An empty list refuses every worker. A
tag is not enough; the runner accepts only a digest reference.

## 6. Install the GitHub App for the repository owner

Work in a repository needs the Cratis Direct GitHub App installed on the account that owns it. An administrator of
that organization installs it from **Connect to GitHub** in Direct's settings and chooses the repositories in
GitHub's installation picker. If you are not an administrator, ask one.

## 7. Check real dispatched work

Start a small piece of work in Direct on a repository the App covers, for example by starting a journey on a
low-risk issue. Then:

1. Read where it runs with `direct_work_where`, or the `direct://me/runners` resource for the runner's state.
2. On a Docker-based runner, watch for a container whose name starts with `cratis-ai-agent-`.
3. Wait for the work to end. A local session ends when its claim is released with `delivered`, `abandoned`, `blocked`
   or `declined`.

If it does not reach your runner, ask `direct_runners_eligible` which runners are eligible. Its reasons are listed
next.

## Why work waits

`direct_runners_eligible` and the Runners settings in Direct name why a runner is not chosen for a piece of work.

| Reason | What it means | What to do |
| --- | --- | --- |
| `Offline` | The runner is not connected. | Start it and check Docker; see [Troubleshooting](./troubleshooting.md). |
| `NotReportedYet` | It is connected but has not reported what it has room for. | Wait a moment, then check again. |
| `NotInAudience` | It does not take work for the person who asked. | [Choose who a runner runs for](./configuring.md#choose-who-a-runner-runs-for). |
| `Paused` | It is paused on the machine or on a schedule. | [Share your machine](./sharing-your-machine.md). |
| `PausedByOperator` | A maintainer paused it in Direct. | Ask a maintainer. |
| `DrainingByOperator` | A maintainer is draining it. | Ask a maintainer. |
| `NoFreeWorkerSlot` | It runs as many workers as it allows. | Wait for a worker to finish. |
| `AtServerCeiling` | The server limit is full although the machine has a slot. | Wait for other work to finish. |
| `CapacityConstrained` | The runner is holding new work until host reserves recover. | Free memory or disk on the machine. |
| `LowOnMemory` | Less memory is free than a worker needs. | Free memory. |
| `LowOnDisk` | The volume holding workers has too little space. | Free disk space. |
| `LacksHarness` | It does not advertise the harness the work needs. | [Declare capabilities](./configuring.md#declare-capabilities). |
| `LacksLocalModel` | It does not advertise the local model the work needs. | [Declare capabilities](./configuring.md#declare-capabilities). |
| `LacksCredential` | It does not advertise the credential provider and mode the work needs. | [Declare capabilities](./configuring.md#declare-capabilities). |
| `AnotherRunnerHasMoreRoom` | It could take the work, but another runner has more free. | None needed. |
| `ClusterHasMoreRoom` | It could take the work, but the cluster has more free. | None needed. |
| `WorkStaysOnCurrentRunner` | The work's worker is still running where it was placed. | None needed. |

A runner whose images are not allowed locally is refused at launch rather than listed here; see
[Troubleshooting](./troubleshooting.md#the-runner-is-connected-but-gets-no-work).

## Next steps

- [Build a harness adapter for local sessions](./harnesses.md)
- [Troubleshooting](./troubleshooting.md)
