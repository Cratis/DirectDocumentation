---
title: Share your machine with Direct
description: Pause the runner, set the hours it stays out of your way, and keep cleanup scripts away from its worker containers.
---

When you need your machine back, pause the runner instead of stopping its workers. You can also set the hours
it stays paused, and tell local cleanup scripts which containers belong to the runner.

You need an [installed, connected runner](./getting-started.md) and Docker running on the same machine.

## Pause new work

Run this as the user who runs the runner, without `sudo`:

```shell
direct-runner pause
```

Workers that are running finish. The runner starts no new work, and Direct sends new work to another runner
or the cluster once it receives the next report, within about ten seconds. The runner also refuses new work
during that reporting delay.

On the [local page](./configuring.md#the-local-page), **Pause** does the same thing. The page shows why the
runner is paused. You can also check from a terminal:

```shell
direct-runner status
```

To take new work again:

```shell
direct-runner resume
```

Or select **Resume** on the local page. Pausing twice keeps the original pause; resuming twice is harmless.
The commands work even when the daemon is stopped. A manual pause survives a restart or reboot until you resume it.

## Set paused hours

Add `PausedDuring` under `Runner` in your existing [runner.json](./configuring.md#where-the-runner-keeps-its-files).
For example, pause on weekday working hours and all weekend:

```json
{
  "Runner": {
    "PausedDuring": "Mon-Fri 08:00-18:00; Sat,Sun 00:00-24:00"
  }
}
```

Restart the runner after changing the setting. It uses the machine's local time, not UTC.

Each window is `[days ]HH:mm-HH:mm`. Separate windows with a semicolon. Day names are `Mon` through `Sun`,
case-insensitive; use a range such as `Mon-Fri` or a comma-separated list such as `Sat,Sun`. Omit the days to
apply a window every day. `24:00` is allowed only as an end time. The start is included and the end is not.

A window can span midnight. `Fri 22:00-06:00` pauses from Friday at 22:00 until Saturday at 06:00; the day names
when the window starts. A blank setting means no scheduled pauses. An invalid setting stops the runner from
starting and prints what is wrong.

A scheduled pause finishes automatically at the window's end. **Resume** only clears a manual pause: it does
not override the schedule. The page and `direct-runner status` show when the scheduled pause ends.

## Keep cleanup scripts away from runner workers

Every worker container the runner starts carries the `io.cratis.direct-runner` Docker label. Its value is the
runner's id. To list those containers, including stopped ones:

```shell
docker ps -a --filter label=io.cratis.direct-runner
```

If you prune stopped containers, exclude that label:

```shell
docker container prune --filter 'label!=io.cratis.direct-runner'
```

This can remove other stopped containers, so review your cleanup policy before running it. The filter protects
runner workers; it does not protect your other containers.

For scripts that remove containers by id, skip the ids returned by:

```shell
docker ps -aq --filter label=io.cratis.direct-runner
```

The runner makes the same promise in return: it only manages worker containers carrying its own label value.
It leaves containers without the label, or labelled for another runner, alone even when their names look like
worker containers.

Workers started by an earlier runner version are not labelled. They keep running, but the updated runner no
longer manages them. Once they finish, remove those containers with `docker rm -f`, specifying only their ids.
