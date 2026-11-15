---
title: Get started with the Direct Agent Runner
description: Install the Direct Agent Runner on macOS or Linux, enroll it with a single-use code you create in Cratis Direct, and check its connection and worker setup.
---

In this tutorial you install the runner and enroll it with a single-use code. When you finish, your machine
is one of your organization's runners. Enrollment does not by itself authorize worker images.

## Before you begin

- Docker is installed and running. Check with `docker info`.
- You can sign in to Cratis Direct as an active member of your organization to create an enrollment code.
  A runner enrolled with a member's code runs work only for that member. To enroll a runner that runs work
  for others, ask a maintainer to create the code.

## 1. Install the runner

Pick the command for your system. Other options, including downloading the binary and starting the runner
at login, are in [Install the runner](./installing.md).

**macOS and Linux with Homebrew**

```shell
brew tap cratis/cratis
brew install direct-runner
```

## 2. Get an enrollment code

Create a code as described below, or ask a maintainer for one. Stop any existing runner daemon before
starting a foreground run; do not run a second copy alongside a login service.

## 3. Connect it to Direct

Use the code flow below on both desktop and headless machines. The local page no longer starts browser
approval. The default Direct address is `https://cratis.direct`; for another deployment, pass
`--Runner:DirectUrl=https://direct.example` with the enrollment command, replacing the made-up address.

## Enroll without a browser

You create a single-use code in Direct, and you start the runner with it. No browser is needed on the
runner machine.

1. In Direct, open **Settings → Runners** and select **Add runner**. Choose the kind of machine, optionally
   enter a name, and choose how long the code is valid: 1, 8 or 24 hours. Select **Create code**.
2. Copy the code. Direct shows it once and keeps only a hash of it, so it cannot show it again.
3. On the machine, start the runner with the code:

   ```shell
   direct-runner --enroll ABCD-EFGH-2345-6789-JKMN-PQRS
   ```

The code above is made up; replace it with the code you received. The runner enrolls itself and uses the
chosen name, or its host name if no name was chosen. There is no further approval step. A code authorizes
one machine, expires after its chosen lifetime and can be revoked. The person who created the code must
stay an active member of the organization; if they are removed, the runner stops receiving work.

For a downloaded runner managed by a login service, use `direct-runner install --enroll <code>` instead.
Do not use this command for a Homebrew-managed runner. See [Install the runner](./installing.md#enroll-the-runner-with-a-code).

## 4. Check that it is connected

When the runner starts, it prints a line like this:

```text
Cratis Direct runner 1.2.3 - run `direct-runner open` for status and enrollment instructions (the page is at http://127.0.0.1:7190)
```

In another terminal, as the same user, run `direct-runner status`. Run `direct-runner open` and open the
one-time address it prints within a minute to view the local page. It shows **Connected** once the daemon
has connected. Opening `http://127.0.0.1:7190` directly does not authenticate you.

In Direct, open **Settings → Runners** to check its connection and resource reports.

Before it can start work, allow the required worker images locally by digest; see
[Install the runner](./installing.md#enroll-the-runner-with-a-code). Direct also checks enrolled capabilities,
who the runner runs for, availability, protocol and capacity. It does not send every job to the machine
with the most room regardless of those checks. You can watch workers with `docker ps`; their names start
with `cratis-ai-agent-`.

## Next steps

- [Start the runner at login](./installing.md#start-the-runner-at-login) so it is there when work arrives.
- [Choose who the runner runs for](./configuring.md#choose-who-a-runner-runs-for) and change its settings.
- [Troubleshooting](./troubleshooting.md) if the page does not say **Connected**.
