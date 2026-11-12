---
title: Configure the Direct Agent Runner
description: Settings for the runner's ports, concurrency, Docker endpoint and files folder, how to connect and disconnect it, and how to choose whose work it runs.
---

Use **Direct's Runners settings** to enroll, name and manage the runner. The **local page** shows its state
and lets you pause, resume or disconnect it. Start-up settings control its address, ports and local policy.

## The local page

The runner serves a page at `http://127.0.0.1:7190` on its own machine only. Run `direct-runner open` as the
same user who runs the daemon, then open the one-time address it prints within a minute. Opening the bare
address does not authenticate you. Never share the one-time address. The page shows these connection states.

| State | What you see | What you can do |
| --- | --- | --- |
| Not connected | **Not connected** | Restart with a maintainer-created enrollment code. |
| Enrollment needed | **Enrollment requires a maintainer-created code** | Use [code enrollment](./getting-started.md#enroll-without-a-browser), not browser approval. |
| Connecting | **Connecting to Direct...** | Wait. The runner retries with a growing delay of up to one minute. |
| Connected | The runner's name, organization, running workers, Direct address and whether it takes work | Pause or resume new work, or disconnect the machine. |
| Not a runner any more | A message that the credential expired or was revoked | Enroll again with a new code. |

**Disconnect this machine** makes the machine forget its identity so you can connect it again from scratch. It
does not remove the runner in Direct. Remove it under **Settings → Runners** for that.

## Direct's Runners settings

In Direct, open **Settings → Runners**. The built-in **Direct cluster** is always listed first, followed by
every enrolled runner. Each row shows:

- A green dot when the runner is connected, and its machine, operating system and how many of its workers run.
- Meters for the runner's CPU, memory and the disk its workers use, as it last reported them.
- Who the runner runs for, a pencil to rename it and a bin to remove it.

### Choose who a runner runs for

By default a runner takes work for everyone in the organization. To limit it, select the audience on the
runner's row (for example *Everyone*), pick the people, and select **Save**. Only work those people ask for
runs on the runner. Pick nobody to run everyone's work again.

This is how you keep a personal machine for your own work, or put a shared build machine at a team's disposal.

### Add a runner with a code

Select **Add runner**, optionally enter a name, choose how long the code stays valid and select **Create code**. Direct shows the code once, with the command to run on the machine. Codes that nobody has used yet are listed under **Waiting for a machine**, with the time left and who created them. Select **Revoke** to stop one from working.

Treat the code like a password. Both `--enroll <code>` and `install --enroll <code>` expose it in process
arguments and may leave it in shell history. Do not paste real codes into shared terminals, logs or notes.
The `DIRECT_RUNNER__Runner__EnrollmentCode` environment variable avoids command-line arguments, but is not
secret storage and may be visible to processes with sufficient access. Remove it from the environment after
use. Only `install` persists a newly supplied code in the owner-only `runner.json`; a foreground run does not.

### Rename a runner

Select the pencil on the runner's row, enter the new name and save. The name is only a label in Direct.

### Remove a runner

Select the bin on the runner's row and confirm. This revokes its authority to receive new work. Do not use
removal as a way to stop containers already on the machine. The local page shows **Not a runner any more**
when the daemon receives an authentication refusal.

## Start-up settings

The runner keeps its settings in `runner.json` in its folder (see [Where the runner keeps its files](#where-the-runner-keeps-its-files)).
It creates the file the first time it starts, with the settings it started with, and reads it on every start after that.
Edit the file and restart the runner to change a setting, or pass it once as an argument or environment
variable and the runner saves it. Enrollment codes are the exception: foreground arguments and environment
codes are not persisted, as described above. The settings listed below are optional.

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
| `DirectUrl` | `https://cratis.direct` | The Direct to connect to. Set this before enrollment. |
| `InterfacePort` | `7190` | The port of the local page. |
| `RelayPort` | `7191` | The port worker containers call back to Direct through. |
| `MaximumConcurrentWorkers` | `0` | How many workers run at once. `0` chooses a quarter of the logical cores, and at least one. |
| `PausedDuring` | No scheduled pauses | Local-time windows when no new work starts, such as `Mon-Fri 08:00-18:00`. See [Share your machine](./sharing-your-machine.md#set-paused-hours). |
| `DockerEndpoint` | Docker's platform default | The Docker endpoint, when it is not the default one, such as `unix:///home/me/.docker/run/docker.sock`. |
| `EnrollmentCode` | none | A single-use code a maintainer creates under **Settings → Runners → Add runner**. Used when no identity is saved. Set it with `--enroll <code>` or `DIRECT_RUNNER__Runner__EnrollmentCode`. A saved code is removed after success or permanent refusal; transient failures are retried. |
| `Workers:AllowedImages` | Empty | Exact digest-pinned worker images allowed locally. Empty refuses all workers. Use the repeatable `--allow-image <repository@sha256:digest>` option or configure the array in `runner.json`. |
| `StateDirectory` | `~/.cratis/direct` | The folder the runner keeps its files in. Set it as an argument or environment variable only, since the runner reads the file from there. |

A setting comes from the first of these that has it:

1. A command-line argument, `--Runner:InterfacePort=7290`.
2. An environment variable. Prefix the name with `DIRECT_RUNNER__` and replace each `:` with `__`, as in `DIRECT_RUNNER__Runner__InterfacePort=7290`.
3. `runner.json`.
4. The default.

```shell
direct-runner --Runner:InterfacePort=7290 --Runner:MaximumConcurrentWorkers=2
```

The runner prints the address of the local page on start; use `direct-runner open` for authenticated access.
The local page binds to loopback. The relay also binds to Docker's bridge address on Linux so containers
can reach it. Change a port when something else on your machine uses it.

> [!NOTE]
> The number of workers also changes what Direct sees. A runner reports its maximum, and Direct sends work to
> an eligible runner with available capacity, after checking its enrolled capabilities, audience, availability
> and protocol. Lower the maximum on a machine you also work on.

### Where the runner keeps its files

The default state folder is `~/.cratis/direct`. On macOS and Linux, newly created state files and the folder
are owner-only. Keep them private even though the identity file no longer contains credentials.

| File | What it holds |
| --- | --- |
| `runner.json` | The settings above. |
| `pause.json` | A manual pause and when it began. `direct-runner resume` removes it. |
| `identity.json` | Identity metadata and a reference to its credential store, not the secret token. |
| `identity.secrets` | Credentials when the file store is used (the Linux default). Treat it like a password. |
| `interface.token` | Secret used to authenticate the owning user to the local page. Never share it. |

On macOS, credentials default to the user's Keychain. Failure to use it does not silently fall back to a file.
Treat all credential-store contents as secrets.

An older identity in the platform's configuration folder (`~/Library/Application Support/direct-runner` on
macOS or `~/.config/direct-runner` on Linux) is migrated when no identity exists in the state folder. Older
inline credentials are migrated into the credential store; migration can fail if that store is unavailable.

Use **Disconnect this machine** to forget the identity and remove its stored credentials. Deleting
`identity.json` alone is not credential cleanup. Enrolling again requires a new code and creates a new runner
in Direct.
