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

With Homebrew, let it manage the runner as a background service:

```shell
brew services start direct-runner
```

The service restarts the runner if it stops and writes its log to `$(brew --prefix)/var/log/direct-runner.log`.
Stop it with `brew services stop direct-runner`.

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

Run the runner as a systemd user service. Create `~/.config/systemd/user/direct-runner.service`:

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
loginctl enable-linger "$USER"   # keep it running when you are not logged in
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

Create a scheduled task that starts the runner when you sign in:

```powershell
$action = New-ScheduledTaskAction -Execute "$env:LOCALAPPDATA\direct-runner\direct-runner.exe"
$trigger = New-ScheduledTaskTrigger -AtLogOn -User $env:USERNAME
$settings = New-ScheduledTaskSettingsSet -ExecutionTimeLimit 0 -RestartCount 3 -RestartInterval (New-TimeSpan -Minutes 1)
Register-ScheduledTask -TaskName "Direct Agent Runner" -Action $action -Trigger $trigger -Settings $settings
```

Start it now with `Start-ScheduledTask -TaskName "Direct Agent Runner"`. Docker Desktop must also start at login.

## Verify the installation

Start the runner and open `http://127.0.0.1:7190`. The runner's page opens. Continue with
[Get started](./getting-started.md#3-connect-it-to-direct) to connect it.

## Update the runner

Install the new version over the old one and restart the runner. The runner keeps its identity and settings in `~/.cratis/direct`, so it stays connected as the same runner. See [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files).

## Uninstall the runner

1. In Direct, open **Settings → Runners** and remove the runner. Its token stops working at once. Selecting
   **Disconnect this machine** on the runner's page only makes this machine forget its identity. Direct still lists the runner until you remove it.
2. Stop the service or scheduled task, and remove the program (`brew uninstall direct-runner`, or delete the file).
3. Delete the runner's folder `~/.cratis/direct` (see [Where the runner keeps its files](./configuring.md#where-the-runner-keeps-its-files)).
