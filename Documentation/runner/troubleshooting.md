---
title: Troubleshoot the Direct Agent Runner
description: Fix local page authentication, enrollment code failures, connection and protocol problems, and workers that cannot start.
---

## The local page does not open

Run `direct-runner status` to check the service and local page address. As the same user who runs the daemon,
run `direct-runner open`, then open its one-time address within a minute. Opening the bare address does not
authenticate you. Do not share the printed address or the runner's `interface.token` file.

If the daemon is stopped, start it. If it exits at startup, read its error; a port conflict is only one
possible cause. Set `--Runner:InterfacePort=7290` for a different local page port. Pass that same setting to
`status` and `open` if needed; see [Configure the runner](./configuring.md#start-up-settings).

The page binds to loopback and requires a local ownership proof. Use code enrollment on a headless machine;
forwarding the port alone does not authenticate your browser.

## "Direct could not be reached"

The runner could not contact the Direct address. Check that the address is right, that the machine can reach it
over HTTPS, and that a proxy or firewall does not block outbound connections.

## I cannot enroll the runner

Browser-code approval is no longer available. Create a single-use code under **Settings → Runners → Add
runner**, or ask a maintainer for one, then restart the runner with it. Any active member can create a code;
a runner enrolled with a member's code runs work only for that member. Set `Runner:DirectUrl` before enrollment
if your organization uses a different Direct address. See [Enroll without a browser](./getting-started.md#enroll-without-a-browser).

## The runner is enrolled but shows as not connected

The runner page says **Connecting to Cratis Direct...**, or Direct shows the runner without a green dot.

- Check the machine still reaches Direct. The runner retries with a delay of up to one minute.
- Check the runner is running. A stopped runner shows as not connected in Direct.
- Check Direct still lists the runner. If its credential expired or was revoked, the page shows **Not a runner
  any more** after an authentication refusal. Enroll again with a new code.

## "This runner is too old for Cratis Direct"

The runner and Direct agree which version of their protocol to speak each time the runner connects. When Direct no
longer speaks any version this runner does, it refuses the connection and the runner page shows the reason, with the
minimum protocol version Direct needs. Install the latest runner and restart it; see
[Install the runner](./installing.md). An incompatible daemon waits a minute between connection attempts;
installing a file alone does not upgrade an already running process.

If the message says **Cratis Direct must be updated**, the runner is newer than Direct. It connects once Direct is
updated.

## The runner is connected but gets no work

A connection is necessary but not sufficient for work placement. Check:

- **Who it runs for.** In **Settings → Runners**, the runner's audience may not include the person whose work is scheduled.
  See [Choose who a runner runs for](./configuring.md#choose-who-a-runner-runs-for).
- **Who enrolled it.** The person who created the runner's enrollment code must still be an active member of
  the organization. When they are removed, the runner stops receiving work.
- **Availability.** A manual or scheduled pause, a maintainer hold or a quarantine can exclude the runner.
- **Capabilities.** The runner must meet the work's requirements within its enrolled capability ceiling.
- **Images.** Worker images must be allowed locally by exact repository and SHA-256 digest. An empty allow-list refuses all workers.
- **How full it is.** CPU, memory, disk reserves and worker slots affect eligibility. Another runner or the
  cluster can take the work only if it meets the requirements.
- **Its worker limit.** Direct shows `N of M workers running`. At the maximum, no more work starts there.
- **Its protocol version.** **Settings → Runners** shows each connected runner's protocol version. A runner that
  speaks an older version than Direct shows an upgrade hint, and work that needs a newer version runs elsewhere.
  Install the latest runner.
- **Docker.** Run `docker info`. When Docker is stopped, a worker cannot start.

## Workers cannot start

The runner starts each worker as a Docker container, named `cratis-ai-agent-` followed by the session.

- Docker is not running, or the runner cannot reach it. Start Docker, or set `Runner:DockerEndpoint` when Docker
  does not use the default endpoint.
- On Linux, your user cannot use Docker. Add it to the `docker` group, then sign in again.
- On Linux, workers cannot reach the runner. Workers call back through `host.docker.internal`. The runner also
  listens on Docker's bridge address for this. Check Docker's default bridge network is up.

## A harness session refuses to start

This applies to adapters for [local sessions](./harnesses.md), a preview. A harness refuses to start when a check on the
launch proof file fails, and Direct refuses a claim it cannot accept.

| What you see | What it means and what to do |
| --- | --- |
| The harness refuses the proof file | One of the [numbered checks](./harnesses.md#checks-before-trusting-the-file) failed. Fix the cause named by the check, such as a file readable by other users or a path inside the worktree. Do not loosen the check. |
| The proof file is missing | The launch expired or the proof was already used. Ask Direct for a new offer. |
| `401` | The token is missing, mistyped, expired or revoked. Create a new one. See [Connect a client](../mcp/connecting.md#troubleshooting). |
| `401` with a tenant header problem | Send exactly one `x-cratis-tenant-id` header with your tenant id, and no other identity or session headers. |
| `403` | The token lacks the `direct:work` scope, or you are not an active member. |
| `not-found` on a claim | Direct does not say whether the work exists. Check that local sessions are turned on, the harness is configured for your organization, and the launch is current. |
| `lease-lost`, `stale-epoch` or `cancelled` | The claim no longer has authority. Stop the local work. A heartbeat or release does not bring it back. Start again from a new offer. |
| `invalid-state` or `idempotency-conflict` | Correct the requested step or `idempotencyKey`. Do not report success. |

The runner is meant to remove a leftover proof file, but that is not implemented yet. If a session ended without
deleting its proof file, delete the file yourself.

## The runner shows a problem about its enrollment code

| Message | What to do |
| --- | --- |
| Direct did not accept the enrollment code | Check the code for typing mistakes, and that the runner points at the right Direct. |
| The enrollment code has expired | Create a new code under **Settings → Runners → Add runner**. |
| The enrollment code was already used | Codes work once. Create a new one. |
| The enrollment code was revoked | The person who created it, or a maintainer, revoked it. Create a new one. |
| This machine is already a runner in Direct | Remove the runner in Direct, disconnect the machine, and enroll again. |
| This Direct does not accept enrollment codes | Your Direct is older than the runner. Update Direct, then restart the runner; it keeps the code until then. |

The runner removes a code that Direct accepted or refused for good. If Direct could not be reached, the runner keeps the code and tries again.

## Start over

To reset a runner, remove it under **Settings → Runners**, then select **Disconnect this machine** on the
authenticated local page. Restart with a new enrollment code. Disconnecting removes the
local identity and stored credentials; deleting `identity.json` alone is not credential cleanup.

## Get help

Report problems and ask questions at [github.com/Cratis/DirectPublic/issues](https://github.com/Cratis/DirectPublic/issues).
