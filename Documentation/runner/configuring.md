---
title: Configure the Direct Agent Runner
description: Settings for the runner's ports, concurrency, Docker endpoint and files folder, how to connect and disconnect it, and how to choose whose work it runs.
---

The runner has two places to configure: the **local page** on the machine, which connects it to Direct, and
**Direct's Runners settings**, which decide what the runner is called and whose work it takes. Everything
else is a small set of start-up settings.

## The local page

The runner serves a page at `http://127.0.0.1:7190` on its own machine only. It shows one of these states.

| State | What you see | What you can do |
| --- | --- | --- |
| Not connected | The Direct address and **Connect to Direct** | Connect the runner. |
| Waiting for approval | A code and **Approve in Direct** | Open the approval page, or connect again for a new code. |
| Connecting | **Connecting to Direct...** | Wait. The runner retries with a growing delay of up to one minute. |
| Connected | The runner's name, organization, running workers, Direct address and whether it takes work | Pause or resume new work, or disconnect the machine. |
| Not a runner any more | A message that Direct removed the machine | Connect again. |

![The Direct runner page connected](./images/local-connected.png)

**Disconnect this machine** makes the machine forget its identity so you can connect it again from scratch. It
does not remove the runner in Direct. Remove it under **Settings → Runners** for that.

## Direct's Runners settings

In Direct, open **Settings → Runners**. The built-in **Direct cluster** is always listed first, followed by
every runner a maintainer approved. Each row shows:

- A green dot when the runner is connected, and its machine, operating system and how many of its workers run.
- Meters for the runner's CPU, memory and the disk its workers use, as it last reported them.
- Who the runner runs for, a pencil to rename it and a bin to remove it.

![Direct's Runners settings with two runners](./images/direct-runners.png)

### Choose who a runner runs for

By default a runner takes work for everyone in the organization. To limit it, select the audience on the
runner's row (for example *Everyone*), pick the people, and select **Save**. Only work those people ask for
runs on the runner. Pick nobody to run everyone's work again.

This is how you keep a personal machine for your own work, or put a shared build machine at a team's disposal.

### Rename a runner

Select the pencil on the runner's row, enter the new name and save. The name is only a label in Direct.

### Remove a runner

Select the bin on the runner's row and confirm. The runner's token stops working at once and it takes no more
work. Work running on it now is not stopped. The local page then shows **Not a runner any more**.

## Start-up settings

The runner keeps its settings in `runner.json` in its folder (see [Where the runner keeps its files](#where-the-runner-keeps-its-files)).
It creates the file the first time it starts, with the settings it started with, and reads it on every start after that.
Edit the file and restart the runner to change a setting, or pass the setting once as an argument or environment variable
and the runner saves it. All settings are optional.

```json
{
  "Runner": {
    "DirectUrl": "https://cratis.direct",
    "InterfacePort": 7190,
    "RelayPort": 7191,
    "MaximumConcurrentWorkers": 0
  }
}
```

| Setting | Default | What it does |
| --- | --- | --- |
| `DirectUrl` | `https://cratis.direct` | The Direct to connect to. The local page pre-fills this and you can change it there. |
| `InterfacePort` | `7190` | The port of the local page. |
| `RelayPort` | `7191` | The port worker containers call back to Direct through. |
| `MaximumConcurrentWorkers` | `0` | How many workers run at once. `0` chooses a quarter of the logical cores, and at least one. |
| `PausedDuring` | No scheduled pauses | Local-time windows when no new work starts, such as `Mon-Fri 08:00-18:00`. See [Share your machine](./sharing-your-machine.md#set-paused-hours). |
| `DockerEndpoint` | Docker's platform default | The Docker endpoint, when it is not the default one, such as `unix:///home/me/.docker/run/docker.sock`. |
| `StateDirectory` | `~/.cratis/direct` | The folder the runner keeps its files in. Set it as an argument or environment variable only, since the runner reads the file from there. |

A setting comes from the first of these that has it:

1. A command-line argument, `--Runner:InterfacePort=7290`.
2. An environment variable. Prefix the name with `DIRECT_RUNNER__` and replace each `:` with `__`, as in `DIRECT_RUNNER__Runner__InterfacePort=7290`.
3. `runner.json`.
4. The default.

```shell
direct-runner --Runner:InterfacePort=7290 --Runner:MaximumConcurrentWorkers=2
```

The runner prints the address of the local page on start, so check it uses the port you set. Both ports
listen on the loopback address only. Change a port when something else on your machine uses it.

> [!NOTE]
> The number of workers also changes what Direct sees. A runner reports its maximum, and Direct sends work to
> the runner with the most of its machine free. Lower the maximum on a machine you also work on.

### Where the runner keeps its files

The runner keeps its JSON files in `~/.cratis/direct` on the machine it runs on, on every operating system
(`%USERPROFILE%\.cratis\direct` on Windows). Only your user can open the folder on macOS and Linux.

| File | What it holds |
| --- | --- |
| `runner.json` | The settings above. |
| `pause.json` | A manual pause and when it began. `direct-runner resume` removes it. |
| `identity.json` | The identity the runner connected with, including its secret token. Treat it like a password. |

An identity saved by an earlier version in the platform's configuration folder (`~/Library/Application Support/direct-runner`,
`~/.config/direct-runner` or `%APPDATA%\direct-runner`) moves to the new folder the first time the runner starts, so it stays connected.

Deleting `identity.json`, or selecting **Disconnect this machine**, makes the runner forget its identity. You connect and
approve it again, and it becomes a new runner in Direct.
