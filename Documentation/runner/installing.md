---
title: Install the Direct Agent Runner
description: Install the Direct Agent Runner on macOS or Linux with Homebrew or a download, enroll it with a code, and manage its login service.
---

The runner is a single self-contained program. It needs no .NET installation. Install it with Homebrew on
macOS or Linux, or download the file for your system.

## Prerequisites

- Docker, running. On macOS install [Docker Desktop](https://docs.docker.com/get-docker/). On Linux
  install Docker Engine and make sure your user can run `docker` without `sudo`.
- Outbound HTTPS access to your Direct address.

## Downloads

Check [github.com/Cratis/DirectPublic/releases](https://github.com/Cratis/DirectPublic/releases) for available
versions. The macOS and Linux archive names have the following form; replace `<version>` with the release
version. A `checksums.txt` file supplies the SHA-256 checksums.

| System | Processor | File |
| --- | --- | --- |
| macOS | Apple silicon | `direct-runner-<version>-osx-arm64.tar.gz` |
| macOS | Intel | `direct-runner-<version>-osx-x64.tar.gz` |
| Linux | x64 | `direct-runner-<version>-linux-x64.tar.gz` |
| Linux | arm64 | `direct-runner-<version>-linux-arm64.tar.gz` |

The archive holds one file: `direct-runner`.

## macOS

### With Homebrew

```shell
brew tap cratis/cratis
brew install direct-runner
```

Update later with `brew upgrade direct-runner`.

### From the download

Set `VERSION` to the version listed on the release page, without the leading `v`:

```shell
VERSION='replace-with-release-version'
curl -fLO "https://github.com/Cratis/DirectPublic/releases/download/v$VERSION/direct-runner-$VERSION-osx-arm64.tar.gz"
tar -xzf "direct-runner-$VERSION-osx-arm64.tar.gz"
sudo install -m 755 direct-runner /usr/local/bin/direct-runner
```

Use `osx-x64` on an Intel Mac. If you downloaded the file with a browser and macOS refuses to open it, remove
the quarantine flag: `xattr -d com.apple.quarantine /usr/local/bin/direct-runner`.

### Start the runner at login

Choose one way to start the runner; do not register it with both service managers.

If you installed from the download, put the executable in its permanent location first, then run as your
normal user, without `sudo`:

```shell
direct-runner install
```

Running `install` again updates and restarts a registration it created. It refuses to overwrite one it did
not create. Enrollment is separate; see [Enroll the runner with a code](#enroll-the-runner-with-a-code).

This creates a LaunchAgent that starts at login and restarts the runner if it stops. Its log is
`~/Library/Logs/direct-runner.log`.

If you installed with Homebrew, let it manage the runner as a background service instead:

```shell
brew services start direct-runner
```

The service restarts the runner if it stops and writes its log to `$(brew --prefix)/var/log/direct-runner.log`.
Stop it with `brew services stop direct-runner`. `direct-runner install` refuses to register a second service
when Homebrew manages the runner.

## Linux

### With Homebrew

Homebrew on Linux installs the same formula:

```shell
brew tap cratis/cratis
brew install direct-runner
```

### From the download

Set `VERSION` to the version listed on the release page, without the leading `v`:

```shell
VERSION='replace-with-release-version'
ARCH=x64   # or arm64
curl -fLO "https://github.com/Cratis/DirectPublic/releases/download/v$VERSION/direct-runner-$VERSION-linux-$ARCH.tar.gz"
curl -fLO "https://github.com/Cratis/DirectPublic/releases/download/v$VERSION/checksums.txt"
sha256sum --check --ignore-missing checksums.txt
tar -xzf "direct-runner-$VERSION-linux-$ARCH.tar.gz"
sudo install -m 755 direct-runner /usr/local/bin/direct-runner
```

The `sha256sum` line prints `OK` for the file you downloaded.

### Start the runner at login

If you installed from the download, run:

```shell
direct-runner install
loginctl enable-linger "$USER"   # keep it running when you are not logged in
```

Run `install` as your normal user, without `sudo`. It writes and enables
`~/.config/systemd/user/direct-runner.service`, starts it and restarts it if it stops. `enable-linger` lets the
user service keep running after logout; your system may require administrator approval. Follow its log with
`journalctl --user -u direct-runner -f`.

If you installed with Homebrew, use `brew services start direct-runner` instead. Use only one service manager.

As an alternative to `install`, create the unit yourself at `~/.config/systemd/user/direct-runner.service`.
`direct-runner install` will not overwrite a unit it did not write:

```ini
[Unit]
Description=Direct Agent Runner
After=network-online.target docker.service

[Service]
ExecStart=/usr/local/bin/direct-runner
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Then enable it:

```shell
systemctl --user daemon-reload
systemctl --user enable --now direct-runner
```

Follow its log with `journalctl --user -u direct-runner -f`.

## Enroll the runner with a code

To start the runner already enrolled, pass a code a maintainer created under **Settings → Runners → Add runner**:

```shell
direct-runner --enroll ABCD-EFGH-2345-6789-JKMN-PQRS
```

The code above is made up; replace it with the maintainer's code. Stop any existing daemon before running
this foreground command. For a downloaded runner's login service, use `direct-runner install --enroll <code>`
instead, replacing `<code>` with that code. Do not use `install` for a Homebrew-managed runner.

Only `install` writes a newly supplied code to `runner.json`. A foreground run's argument or environment code
is not written there. A code already in the file is removed after successful enrollment or permanent refusal;
transient failures are retried. See [Enroll without a browser](./getting-started.md#enroll-without-a-browser).

Before running work, approve each worker image locally by its exact repository and SHA-256 digest. Use
`--allow-image <repository@sha256:digest>` on a foreground run or on `install`, replacing the placeholder with
a real, approved digest reference. Repeat the option for multiple images; a supplied list replaces the saved
list. An empty list refuses every worker. Enrollment and a connection alone do not guarantee work eligibility.

## Verify the installation

Start the runner, then run `direct-runner open` as the same user. Open the one-time address it prints within
a minute. The page is at `http://127.0.0.1:7190` by default, but opening that address directly does not
authenticate you. Continue with [Get started](./getting-started.md#3-connect-it-to-direct) to enroll it.

## Check the runner

```shell
direct-runner status
```

This shows whether the login service is installed and running, and whether the runner is connected to Direct.
It also reports whether new work is paused and how many worker images are allowed when the daemon answers.
It does not print an enrollment code or an approval link. Use `direct-runner open` for authenticated page access.

## Update the runner

Install the new version over the old one and restart the runner. The runner keeps its identity and settings in `~/.cratis/direct`, so it stays connected as the same runner. See [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files).

## Uninstall the runner

1. In Direct, open **Settings → Runners** and remove the runner to revoke its authority for new work. If you
   also want to erase local credentials, select **Disconnect this machine** on the authenticated local page
   before stopping the daemon. Disconnecting locally does not remove the runner's listing in Direct.
2. Stop and remove the login service with `direct-runner uninstall`, or use `brew services stop direct-runner`
   if Homebrew manages it. Then remove the program
   (`brew uninstall direct-runner`, or delete the file). `direct-runner uninstall` keeps the identity and settings
   and credentials in `~/.cratis/direct` (or the macOS Keychain). For a hand-written systemd unit, disable it with `systemctl --user disable --now direct-runner`
   and remove the unit yourself; `uninstall` only removes a service it wrote.
3. After disconnecting, optionally delete the remaining state folder `~/.cratis/direct` to remove settings.
   Deleting the folder alone does not remove credentials from the macOS Keychain. See
   [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files).
