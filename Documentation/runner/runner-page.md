---
title: See what your runners are doing
description: Read the status of each runner under Settings → Runners, see the work a runner is running, and find out why a piece of work did not go to a runner.
---

Cratis Direct shows how every runner stands, what it is working on, and why a piece of work did or did not
land on it. Use this page when you want to know whether a runner takes work, or why your work ran on the cluster
instead of your machine.

## Read a runner's status

Each runner under **Settings → Runners** shows one status. It tells you whether Direct sends new work there.

| Status | What it means | What the row says |
| --- | --- | --- |
| Online | The runner is connected but has not reported what it has yet. Direct does not send it work until it does. | `Online - waiting for its first report` |
| Idle | It runs no workers and takes new work. | `Idle - 0 of 4 workers running` |
| Busy | It runs some workers and still has a worker slot open. | `Busy - 2 of 4 workers running` |
| Full | Every worker slot is taken. No more work starts there until one finishes. | `Full - 4 of 4 workers running` |
| Paused | It refuses new work. The workers it already runs finish. | `Paused at the machine since 09:15 - 1 of 4 workers running` |
| Offline | It is not connected, or has been silent too long. | `Offline` |

While Direct is still reading a runner, the row shows `Checking…`.

An idle or busy runner that is low on memory or disk adds `· low on memory or disk` to its status. It is connected
and not paused, but Direct passes it over for new work until it has room again.

A pause wins over the number of workers. A runner that is paused and also full reads as paused, because the pause is
why it takes no new work.

### Why a runner is paused

A paused runner says why.

| What the row says | What happened |
| --- | --- |
| `Paused at the machine since 09:15` | Someone paused the runner on its own machine at that time. |
| `Paused at the machine` | The same, without a time. |
| `Paused until 18:00` | The runner's schedule keeps it from taking work until that time. |
| `Paused by its schedule` | The same, without an end time. |
| `Paused` | The runner is paused, but does not say why. Runners on an older version of the runner program show this. |

Times are shown in your local time. Everyone sees a pause set on the machine the same way, whoever set it.

To change a pause, change it where it was set: on the runner's machine, or in the runner's schedule. See
[Configure the runner](./configuring.md).

## See what a runner is working on

Under its status, a runner row lists the work it runs, as `Working on:` followed by one link per piece of work.
Each link reads as the kind of work and its state, for example `Implementation - Running`, and opens that work in
Direct.

- **Work you may not see.** You only see links to work you have access to. Work you may not see appears as
  `work (hidden)`, with no link. You can tell the runner is busy, but nothing else about the work is shown.
- **Work the runner has not confirmed.** A link whose runner has not confirmed that it runs the work shows
  `Not confirmed by the runner` when you point at it. The work may have started after the runner connected.
- **Workers Direct does not track.** When the runner runs more workers than Direct accounts for, the row adds a count
  such as `· 1 session not tracked by Direct`.
- **A runner that is offline.** The label changes to `Last known working on:`. It shows what the runner was running
  when Direct last heard from it, not what it runs now.

A runner that runs nothing shows no `Working on:` line.

## Find out why a runner was not picked

When work did not go to your runner, ask Direct why.

1. Open the work item.
2. Select the **Runner** tab.

The tab starts with one sentence saying where the work would run if Direct placed it now:

- `Placed now, this work would run on <runner name>.`
- `Placed now, this work would run on the cluster.`
- `Placed now, this work would run on a runner not shown to you.`
- `Its worker may still be running, so this work stays on <place>.` - see
  [When the work stays where it is](#when-the-work-stays-where-it-is).

Under it, every runner you can see is listed. The runner Direct would pick is marked `Would be picked`, or
`Stays here` when the work stays where it is. Every other runner shows one or more reasons.

| Reason | What it means |
| --- | --- |
| Offline | The runner is not connected. |
| Not reported yet | The runner is connected but has not said what it has room for. |
| Not open to the requester | The runner does not take work for the person who asked for this work. See [Choose who a runner runs for](./configuring.md#choose-who-a-runner-runs-for). |
| Paused | The runner is paused. |
| No free worker slot | The runner already runs as many workers as it allows. |
| Low on memory | The machine has less free memory than a worker needs. |
| Low on disk | The disk that holds workers has less free space than a worker needs. |
| Another runner has more room | The runner could take the work, but another runner has more of its machine free. |
| The cluster has more room | The runner could take the work, but it has no more free than the cluster. |
| The work stays where its worker runs | The runner could take the work, but the work stays where it was placed. |

A runner can show more than one reason. A paused runner that is also full shows both.

The explanation uses the same checks Direct uses to place work, so it matches what happens. It reads how the runners
stand when you open the tab. It does not update by itself and it does not show what happened when the work was
first placed. Select **Check again** to read it again.

### Runners you cannot see

You are only shown the runners that are open to everybody or to you. Other runners are only counted, under the
list: `1 other runner is not shown to you.` or `3 other runners are not shown to you.` If one of them is where the
work would go, the sentence at the top says `a runner not shown to you`.

Maintainers of the organization and global administrators see every runner.

If the tab shows nothing, Direct cannot explain the work to you. That is the case when you may not see the work, or
when it does not exist in your organization. Direct does not say which.

### When the work stays where it is

Work that was placed on a runner or on the cluster keeps that placement while its worker may still be running.
The tab then says `Its worker may still be running, so this work stays on …`. The other runners show
`The work stays where its worker runs`, whatever room they have.

## When your work did not go to your runner

Read the **Runner** tab of the work, then compare it with the runner's row under **Settings → Runners**.

- If your runner shows `Not open to the requester`, a maintainer needs to add the person who asked for the work. See
  [Choose who a runner runs for](./configuring.md#choose-who-a-runner-runs-for).
- If it shows `Offline` or `Not reported yet`, check that the runner is running and connected. See
  [Troubleshooting](./troubleshooting.md#the-runner-is-approved-but-shows-as-not-connected).
- If it shows `Paused`, find out why from the row's pause reason, and resume it where it was paused.
- If it shows `No free worker slot`, `Low on memory` or `Low on disk`, wait for work to finish or free up the machine.
- If it shows `The cluster has more room` or `Another runner has more room`, the runner works, and another place had
  more free when the work was placed.

## Next steps

- [Configure the runner](./configuring.md) - settings, and who a runner runs work for.
- [Troubleshooting](./troubleshooting.md) - when the runner does not connect or does not take work.
