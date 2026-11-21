---
title: How work picks a provider, a harness and a runner
description: How Cratis Direct chooses an AI provider and harness for a piece of work, which runner can take it, and what happens when nothing fits.
---

Before a worker starts, Cratis Direct decides two separate things: which AI provider and harness the work
uses, and which runner, if any, it runs on. The first decision can fall back to another provider. The
second only filters runners, and it never changes the provider or the harness.

## What a runner advertises

A runner tells Direct what its machine can offer, using the `Capabilities` setting described in
[Declare capabilities](./configuring.md#declare-capabilities).

- **Harnesses**: the worker harnesses the machine can run. These are `claude`, `pi`, `copilot`, `codex` and
  `opencode`.
- **Local models**: models served on the machine, each with a model name and an endpoint that workers can reach.
- **Credentials**: which kinds of credential are available, by provider and mode. This is a label only. The
  runner never sends a key or token.

An advertisement is a claim. Direct places work only within the capabilities the runner was enrolled with,
so a runner cannot gain access by advertising more.

> [!NOTE]
> You can advertise `codex` and `opencode`, but no provider resolves to them yet, so a runner that
> advertises only those is not matched.

## Two kinds of provider

How a piece of work is placed depends on its provider.

| | Provider with no runner requirement | Provider set to run only on runners |
| --- | --- | --- |
| Which runner | The eligible runner with the most free capacity, compared with the cluster. | Only a runner that advertises the exact harness, local model and endpoint, and credential the provider needs. |
| Nothing fits | The work runs on the Direct cluster. | The work stays queued. |
| Cloud fallback | Yes. | Never. |

A runner must also be enrolled with matching capabilities, be available to the person who asked for the
work, not be paused or held, and have room: a free worker slot, free memory and free disk, as described in
[Share your machine](./sharing-your-machine.md). Among the runners that qualify, the one with the most free
capacity is chosen.

When nothing fits, the work waits in the queue and shows why. For example, it reports that no eligible
runner can run the provider, harness, model and credential mode, and that the work remains queued. It does
not move to the cloud, to another provider or to another harness. It starts when a matching runner becomes
available.

## Fallback happens before placement

Fallback applies while Direct resolves the provider, which is before it looks for a runner.

1. **Provider or pool member.** An agent uses either one provider or a pool of providers. A provider is
   skipped when it has been removed, is restricted to other people, is at its concurrency limit, is over its
   usage limit or has no usage allowance left. A pool then tries its next member, in the order the pool
   defines. If the pool's fallback is set to pause and ask, it tries only its first choice and the work waits
   for a person instead of moving on.
2. **Harness.** For a provider with no runner requirement, Direct tries the agent's own harness first, then
   the other harnesses the provider allows. For a provider that runs only on runners, only the required
   harness is used. If the provider does not allow that harness, no runner can match, and the work waits.
3. **Runner.** Direct looks for a runner as described above.

Once a member that runs only on runners is selected, it stays selected. If no runner has room, the work
waits. It is not handed to a later pool member because no runner is free.

### Keep work off the cloud

A provider that is skipped is skipped before a runner is considered, so a pool set to continue can move
from a skipped runner-only member to a later member that runs in the cloud. To keep work off the cloud,
either:

- put only providers that run only on runners in the pool, or
- set the pool's fallback to pause and ask.

## Troubleshoot

| What you see | What to check |
| --- | --- |
| Work stays queued and says no eligible runner can run it | Whether a runner is online, enrolled, available to the person who asked, not paused and has room. Its `Capabilities` must list the harness, and when the provider needs one, the local model with the same endpoint and the credential provider and mode. |
| The runner is connected but never gets the work | The harness the provider resolves to is advertised, and the runner's enrolled capabilities include what it advertises. A maintainer can enroll a runner with more. |
| Work moved to the cloud | The provider has no runner requirement, or the pool continued to another member. See [Keep work off the cloud](#keep-work-off-the-cloud). |

Related pages:

- [Configure the runner](./configuring.md) covers the `Capabilities` setting and who a runner runs for.
- [Share your machine](./sharing-your-machine.md) covers pausing and the room a runner keeps for you.
- [Troubleshooting](./troubleshooting.md) covers a runner that does not connect.
