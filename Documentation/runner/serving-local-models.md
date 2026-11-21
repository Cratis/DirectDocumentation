---
title: Serve local AI models with the runner
description: Expose models from Ollama, LM Studio, vLLM or llama.cpp on your machine to Cratis Direct, add them as AI providers and see if the runner is online.
---

The runner can serve AI models that run on your own machine to Cratis Direct. You point it at a model
server such as Ollama, LM Studio, vLLM or llama.cpp, and the runner reports which models that server has.
An administrator can then add one as an AI provider in Direct, and agents use it like any other provider,
while the model keeps running on your hardware.

This is separate from running worker containers. A runner can do both, or only serve models: start it with
`--models-only` and it needs no Docker.

## How it works

The runner already connects out to Cratis Direct, so your machine needs no open port. When it serves
models, the same connection carries each request to your model server and brings the answer back, including
answers that stream as the model writes them.

1. You tell the runner where your model server is, on the runner's local page.
2. The runner asks the server which models it has and reports them to Cratis Direct. It asks again about
   once a minute, so a model you pull shows up without a restart.
3. In Direct, **Settings → AI → Local models** lists what each connected runner reports. A maintainer adds
   a model as an AI provider.
4. Direct sends requests for that provider through the runner to your model server.

Cratis Direct only ever learns each server's name, the protocols it speaks, its models and whether the
runner could reach it. It never receives the server's address or its API key.

## Tell the runner about a model server

Run `direct-runner open` as the user who runs the daemon and open the one-time address it prints. Under
**Models on this machine**, select **Add a model server** and fill in the form.

| Field | What to enter |
| --- | --- |
| **Name** | What people see in Direct, for example `Workstation`. |
| **Identity** | Lowercase letters, digits and hyphens, unique on this machine, for example `workstation`. |
| **Address of the model server** | Its base address, for example `http://localhost:11434` for Ollama or `http://localhost:1234` for LM Studio. Use `http` or `https`, without a user name or password. |
| **API key, if it wants one** | The key the server expects, if any. It is kept in the runner's credential store and is never shown again; select **Forget the saved key** to remove it. |
| **OpenAI** and **Claude** | The protocols the server answers. Ollama and LM Studio answer both. |
| **Serve it** | Clear it to stop serving this server without deleting it. |
| **Models to serve** | One model per line. Leave it empty to serve every model the server lists. |

Select **Detect** before you save. The runner asks the address which protocols and models it finds, and
shows them so you can accept them. Detect uses a saved key only while the address is unchanged, so a typo
cannot send your key to another host.

Changes apply straight away. Each server shows whether the runner can reach it, how many models it serves
and over which protocols, and whether Cratis Direct has them.

### Settings file

The same servers live under `Models` in `runner.json`, which is useful for a machine you set up without the
page. Each entry has these settings.

| Setting | Meaning |
| --- | --- |
| `Id` | The identity, as above. |
| `Name` | The name people see. Defaults to the identity. |
| `Url` | The server's base address. |
| `Protocols` | `openai`, `claude` or both. Empty uses what the server answers to. |
| `Models` | The models to serve. Empty serves every model the server lists. |
| `Enabled` | Whether the server is served. Defaults to `true`. |

Give an API key on a headless machine with an environment variable such as `DIRECT_RUNNER__Models__0__ApiKey`.
The runner never writes it to `runner.json` and holds it only in memory.

### Run only to serve models

Start the runner with `--models-only`, or set `Mode` to `ModelsOnly`, to serve models without running
workers. In this mode the runner needs no Docker, takes no worker work and shows no Docker problems.
`direct-runner status` prints the mode and each server it serves, with any problem. Changing the mode
needs a restart.

> [!NOTE]
> `Capabilities` → `LocalModels` is a different setting: it tells Direct about models that worker
> containers on this machine can already reach. Serving models, as described here, makes the model available
> to Cratis Direct itself.

## Add a model as an AI provider

In Cratis Direct, open **Settings → AI**. The **Local models** panel lists every runner that serves models.
Each runner shows a dot and a status for whether it is online or offline, in the same wording as the
Runners page, then its model servers with their models and protocols. A server shows **Not responding** when the runner could not reach it at its last check.

Select **Add as provider** next to a server to add it as an AI provider. Only a maintainer can add one, and
a server is added once. The button is disabled, with the reason beside it, when:

- the server answers only the Claude protocol, because Cratis Direct reaches added models over the OpenAI
  protocol
- the runner is offline, or cannot reach the server
- the server lists no models
- the server is already added

An added provider appears with the other AI providers and shows whether the runner behind it is online,
offline or not responding, so you can tell a machine that is switched off from a provider that is broken.
Removing the provider stops the access it was given.

## Keep it safe

A model server on your machine now answers requests that come from Cratis Direct, so treat it like any other
service you expose to your organization.

- **Prompts reach your machine.** Everything an agent sends to the provider is sent to the model on your
  machine, and its answers come back from there.
- **Only model requests pass.** The runner forwards only the model API paths, never a server's management
  paths such as pulling or deleting models, and it refuses anything else.
- **Your key stays here.** The runner talks to your server with its own key. The credentials of whoever
  called Cratis Direct are never passed on.
- **Pausing does not stop it.** Pausing the runner stops new worker work, not requests to models it serves.
  Switch off **Serve it** or stop the runner to stop serving.
- **Limits.** The runner handles up to eight requests at once and refuses large uploads.

## Troubleshoot

| What you see | What to check |
| --- | --- |
| The server shows a problem on the local page | The address, the port and the key. **Detect** shows what the server answered. |
| Nothing appears under **Settings → AI → Local models** | The runner is connected and the server is switched on. A runner older than this feature serves nothing; update it. |
| The runner shows as offline | Whether the machine is awake and the runner is running. See [Troubleshooting](./troubleshooting.md). |
| **Add as provider** is disabled | The reason is shown beside it. |
| A model you pulled is missing | The runner asks about once a minute, and **Models to serve** may list only some models. |
