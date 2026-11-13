---
title: Direct Agent Runner
description: What the Direct Agent Runner is, how it connects to Direct, and what it means for where your agents run and what they can reach.
---

The Direct Agent Runner is a small program you run on your own machine. It connects out to Direct and
runs agent workers there, in Docker, the same way the Direct cluster runs them. Work that Direct
schedules can land on your machine when it meets the work's requirements and has capacity.

Use a runner when agents need something only your machine has: more CPU or memory than the cluster
gives you, a fast local disk, or access to a network you control.

## How it works

1. You install the runner. A maintainer creates a single-use enrollment code under **Settings → Runners → Add runner**.
2. You start the runner with that code. No browser or further approval is needed on the runner machine.
3. You allow worker images locally by their exact repository and SHA-256 digest. An empty allow-list refuses all workers.
4. Direct checks enrolled capabilities, who the runner runs for, availability, protocol and capacity before
   choosing where work runs. Cluster fallback is possible only when the cluster meets the same requirements.
5. For admitted work, the runner starts a worker container in your local Docker and streams its console back
   to Direct.

Current Direct asks each connected runner to report its processor, memory and disk use every ten seconds. That is how Direct
knows where there is room.

## What the runner needs

- A machine running macOS or Linux, on x64 or arm64.
- [Docker](https://docs.docker.com/get-docker/) (Docker Desktop or any engine), running. Workers run as
  containers.
- Outbound HTTPS access to your Direct address (`https://cratis.direct` unless your organization runs its
  own).

The local page binds to loopback. The relay workers call back through also binds to Docker's bridge on
Linux so containers can reach it. Do not expose either port to other machines.

## Trust and security

A runner is handed the credentials of every piece of work it runs. Treat the machine like any other place
you keep your organization's secrets.

- **Only a maintainer can create an enrollment code.** Create one only for a machine you control and trust.
- **The local page requires authentication.** Run `direct-runner open` as the daemon's user and open the
  one-time address it prints within a minute. Do not share that address or the local authentication files.
- **An enrollment code works once.** A maintainer creates it, Direct stores only a hash of it, it stops working when it expires or is revoked, and the runner removes it from its settings after use. Only create a code for a machine you control, and treat it like a password until it is used.
- **Removing a runner revokes its authority for new work.** Do not rely on removal to stop containers
  already running on the machine.

## Next steps

- [Get started](./getting-started.md) - install and enroll your first runner.
- [Install the runner](./installing.md) - installation options for macOS and Linux.
- [Configure the runner](./configuring.md) - settings, and who a runner runs work for.
- [Share your machine](./sharing-your-machine.md) - pause new work, set paused hours and protect worker containers from cleanup.
- [Troubleshooting](./troubleshooting.md) - when the runner does not connect or does not take work.
