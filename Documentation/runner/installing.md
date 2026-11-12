---
title: Install the Direct Agent Runner
description: Install the Direct Agent Runner on macOS, Linux and Windows with Homebrew or a download, verify the archive, and start the runner automatically at login.
---

The runner is a single self-contained program. It needs no .NET installation. Install it with Homebrew on
macOS or Linux, or download the file for your system.

## Prerequisites

- Docker, running. On macOS and Windows install [Docker Desktop](https://docs.docker.com/get-docker/). On Linux
  install Docker Engine and make sure your user can run `docker` without `sudo`.
- Outbound HTTPS access to your Direct address.

## Downloads

Every release is published at [github.com/Cratis/DirectPublic/releases](https://github.com/Cratis/DirectPublic/releases)
with one file per system and a `checksums.txt` file with the SHA-256 of each.

| System | Processor | File |
| --- | --- | --- |
| macOS | Apple silicon | `direct-runner-<version>-osx-arm64.tar.gz` |
| macOS | Intel | `direct-runner-<version>-osx-x64.tar.gz` |
| Linux | x64 | `direct-runner-<version>-linux-x64.tar.gz` |
| Linux | arm64 | `direct-runner-<version>-linux-arm64.tar.gz` |
| Windows | x64 | `direct-runner-<version>-win-x64.zip` |
| Windows | arm64 | `direct-runner-<version>-win-arm64.zip` |

The archive holds one file: `direct-runner` (`direct-runner.exe` on Windows).

## macOS

### With Homebrew

```shell
brew tap cratis/cratis
brew install direct-runner
```

Update later with `brew upgrade direct-runner`.

### From the download

```shell
VERSION=3.20.3
curl -fLO "https://github.com/Cratis/DirectPublic/releases/download/v$VERSION/direct-runner-$VERSION-osx-arm64.tar.gz"
tar -xzf "direct-runner-$VERSION-osx-arm64.tar.gz"
sudo install -m 755 direct-runner /usr/local/bin/direct-runner
```

Use `osx-x64` on an Intel Mac. If you downloaded the file with a browser and macOS refuses to open it, remove
the quarantine flag: `xattr -d com.apple.quarantine /usr/local/bin/direct-runner`.

### Start the runner at login

Choose one way to start the runner; do not register it with both service managers.

If you installed from the download, run:

```shell
direct-runner install
```

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

```shell
VERSION=3.20.3
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

This writes and enables `~/.config/systemd/user/direct-runner.service`. Follow its log with
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

## Windows

### From the download

In PowerShell:

```powershell
$version = "3.20.3"
$arch = "x64"   # or arm64
Invoke-WebRequest "https://github.com/Cratis/DirectPublic/releases/download/v$version/direct-runner-$version-win-$arch.zip" -OutFile direct-runner.zip
Expand-Archive direct-runner.zip -DestinationPath "$env:LOCALAPPDATA\direct-runner"
```

Start it with `& "$env:LOCALAPPDATA\direct-runner\direct-runner.exe"`. If Windows SmartScreen warns about the
file, check its checksum against `checksums.txt` (`Get-FileHash direct-runner.zip`) before choosing to run it.

### Start the runner at login

`direct-runner install` is not available on Windows; use the scheduled task below.

Create a scheduled task that starts the runner when you sign in:

```powershell
$action = New-ScheduledTaskAction -Execute "$env:LOCALAPPDATA\direct-runner\direct-runner.exe"
$trigger = New-ScheduledTaskTrigger -AtLogOn -User $env:USERNAME
$settings = New-ScheduledTaskSettingsSet -ExecutionTimeLimit 0 -RestartCount 3 -RestartInterval (New-TimeSpan -Minutes 1)
Register-ScheduledTask -TaskName "Direct Agent Runner" -Action $action -Trigger $trigger -Settings $settings
```

Start it now with `Start-ScheduledTask -TaskName "Direct Agent Runner"`. Docker Desktop must also start at login.

## Enroll the runner with a code

To start the runner already enrolled, pass a code a maintainer created under **Settings → Runners → Add runner**:

```shell
direct-runner --enroll ABCD-EFGH-2345-6789-JKMN-PQRS
```

The runner saves the code in `runner.json` only until it has used it, then removes it. See [Enroll without a browser](./getting-started.md#enroll-without-a-browser).

## Verify the installation

Start the runner and open `http://127.0.0.1:7190`. The runner's page opens. Continue with
[Get started](./getting-started.md#3-connect-it-to-direct) to connect it.

## Check the runner

```shell
direct-runner status
```

This shows whether the login service is installed and running, and whether the runner is connected to Direct.
While it is waiting for approval, it shows the approval code and link. If it is not connected, it shows the local
page to open.

## Update the runner

Install the new version over the old one and restart the runner. The runner keeps its identity and settings in `~/.cratis/direct`, so it stays connected as the same runner. See [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files).

## Uninstall the runner

1. In Direct, open **Settings → Runners** and remove the runner. Its token stops working at once. Selecting
   **Disconnect this machine** on the runner's page only makes this machine forget its identity. Direct still lists the runner until you remove it.
2. Stop and remove the login service with `direct-runner uninstall`, or use `brew services stop direct-runner`
   if Homebrew manages it. On Windows, stop the scheduled task. Then remove the program
   (`brew uninstall direct-runner`, or delete the file). `direct-runner uninstall` keeps the identity and settings
   in `~/.cratis/direct`. For a hand-written systemd unit, disable it with `systemctl --user disable --now direct-runner`
   and remove the unit yourself; `uninstall` only removes a service it wrote.
3. If you also want to remove the identity and settings, delete the runner's folder `~/.cratis/direct`
   (see [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files)).
