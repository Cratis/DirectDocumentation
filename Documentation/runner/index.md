---
title: Direct Agent Runner
description: What the Direct Agent Runner is, how it connects to Direct, and what it means for where your agents run and what they can reach.
---

The Direct Agent Runner is a small program you run on your own machine. It connects out to Direct and
runs agent workers there, in Docker, the same way the Direct cluster runs them. Work that Direct
schedules lands on your machine instead of the cluster whenever your runner has the most room.

Use a runner when agents need something only your machine has: more CPU or memory than the cluster
gives you, a fast local disk, or access to a network you control.

## How it works

1. You install the runner and start it. It serves a configuration page on your machine only.
2. You connect it to Direct from that page. The runner shows a short code.
3. A maintainer approves the code in Direct, under **Settings → Runners**. The runner is now one of your
   organization's runners.
4. For each piece of work, Direct picks the runner that is open to whoever asked for the work and has the
   most of its machine free. When no runner has more room than the cluster, the work runs on the cluster.
5. The runner starts a worker container for the work in your local Docker and streams the worker's
   console back to Direct.

Each runner reports its processor, memory and disk use to Direct every few seconds. That is how Direct
knows where there is room.

## What the runner needs

- A machine running macOS, Linux or Windows, on x64 or arm64.
- [Docker](https://docs.docker.com/get-docker/) (Docker Desktop or any engine), running. Workers run as
  containers.
- Outbound HTTPS access to your Direct address (`https://cratis.direct` unless your organization runs its
  own).

The runner never accepts connections from the network. Its configuration page and the relay workers call
back through listen on the loopback address of your machine (plus Docker's bridge on Linux, so containers
can reach the relay).

## Trust and security

A runner is handed the credentials of every piece of work it runs. Treat the machine like any other place
you keep your organization's secrets.

- **Only a maintainer can approve a runner.** The approval dialog shows the machine's name, system, the
  address it asked from and the code, so you can check it is yours.
- **Only approve a code you see on your own machine.** Never approve a code someone sends you.
- **The configuration page is local.** It answers on `127.0.0.1` only, and it refuses changes that do not
  come from the page itself, so another website open in your browser cannot connect or disconnect the runner.
- **Removing a runner in Direct takes effect at once.** Its token stops working. Work already running on it
  is not stopped.

## Next steps

- [Get started](./getting-started.md) - install, connect and approve your first runner.
- [Install the runner](./installing.md) - every installation option for macOS, Linux and Windows.
- [Configure the runner](./configuring.md) - settings, and who a runner runs work for.
- [See what your runners are doing](./runner-page.md) - each runner's status, its current work, and why work was not picked for it.
- [Troubleshooting](./troubleshooting.md) - when the runner does not connect or does not take work.
