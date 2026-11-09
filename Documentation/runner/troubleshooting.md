---
title: Troubleshoot the Direct Agent Runner
description: Fix a Direct Agent Runner that does not connect, cannot be approved, shows as not connected in Direct, is too old for Direct, or does not take work.
---

## The local page does not open

The runner is not running, or it uses another port. Start it, and read the line it prints:

```text
Direct runner 3.20.3 - open http://127.0.0.1:7190 to connect it to Direct
```

If the runner exits at start, another program is using the port. Choose another with
`--Runner:InterfacePort`. See [Configure the runner](./configuring.md#start-up-settings).

The page only answers on `127.0.0.1` and `localhost` on the machine the runner runs on. It is not reachable from
another computer, by design. To configure a runner on a remote machine, forward the port over SSH:
`ssh -L 7190:127.0.0.1:7190 user@host`, then open `http://127.0.0.1:7190` locally.

## "Direct could not be reached"

The runner could not contact the Direct address. Check that the address is right, that the machine can reach it
over HTTPS, and that a proxy or firewall does not block outbound connections.

## "That is not a Direct address"

The address on the page is not a valid URL. Include the scheme, for example `https://cratis.direct`.

## "No machine is waiting under that code"

The code in Direct's approval dialog expired, or the runner connected again and made a new one. Select **Connect to
Direct** on the runner's page for a new code and approve that one.

## I cannot approve the code

Only maintainers can approve a runner. Ask a maintainer of your organization to open the approval link, or to type the
code under **Settings → Runners**.

## The runner is approved but shows as not connected

The runner page says **Connecting to Direct...**, or Direct shows the runner without a green dot.

- Check the machine still reaches Direct. The runner retries with a delay of up to one minute.
- Check the runner is running. A stopped runner shows as not connected in Direct.
- Check Direct still lists the runner. If a maintainer removed it, the page shows **Not a runner any more**. Connect
  it again.

## "This runner is too old for Cratis Direct"

The runner and Direct agree which version of their protocol to speak each time the runner connects. When Direct no
longer speaks any version this runner does, it refuses the connection and the runner page shows the reason, with the
minimum protocol version Direct needs. Install the latest runner, see [Install the runner](./installing.md). The runner
tries again once a minute, so it connects shortly after the upgrade.

If the message says **Cratis Direct must be updated**, the runner is newer than Direct. It connects once Direct is
updated.

## The runner is connected but gets no work

Work runs on the runner with the most of its machine free that is open to whoever asked. Check:

- **Who it runs for.** In **Settings → Runners**, the runner's audience may not include the person whose work is scheduled.
  See [Choose who a runner runs for](./configuring.md#choose-who-a-runner-runs-for).
- **How full it is.** A machine near its CPU, memory or disk limits is passed over, and the cluster runs the work.
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

## Start over

To reset a runner, in Direct remove it under **Settings → Runners**, then on the machine select **Disconnect this
machine** and connect and approve it again.

## Get help

Report problems and ask questions at [github.com/Cratis/DirectPublic/issues](https://github.com/Cratis/DirectPublic/issues).
