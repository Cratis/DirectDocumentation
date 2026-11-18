---
title: Build a harness adapter for local sessions
description: The contract between the Direct Agent Runner and a coding-agent harness in a local session, covering the launch proof file, the checks a harness makes before trusting it, and how it claims, keeps and releases work.
---

A local session runs work on your own machine with a coding-agent harness instead of in a worker container. This page
is for people who write an adapter that starts a harness in a local session. It describes what the runner hands the
harness, what the harness must check, and how it claims work from Direct.

> [!NOTE]
> Local sessions are a preview. The claim tool names (`direct_work_claim`, `direct_work_heartbeat` and
> `direct_work_release`) and the launch details below can still change. The runner does not yet start a harness
> itself, so nothing here is something an installed runner does today. Local sessions are also off unless whoever runs
> your Direct turns them on.

## Who does what

- The **runner** launches the harness in a worktree and gives it a one-time launch proof and a few non-secret
  settings.
- The **harness** uses the proof to claim the work from Direct, keeps the claim alive, and releases it.
- **Direct** decides whether the claim is allowed. Each claim carries an epoch, a number that identifies the current
  claim, so a stale or superseded harness is refused.

The harness never holds the runner's credential. It holds only the personal access token of the person who launched
the work. See [Connect a client](../mcp/connecting.md) to create one.

## Harness contract

### What the runner delivers

| Item | Value |
| --- | --- |
| Launch proof file | A file holding the proof. The runner passes only its path, never the proof, in `DIRECT_LAUNCH_PROOF_FILE`. |
| Proof format | 64 lowercase hexadecimal characters, with no trailing newline. |
| File access | Created for the launching user only (mode `0600` on macOS and Linux), in a fresh folder for this launch. Checks 4 and 5 below assume a platform that has file owners and modes. |
| Location | Outside the worktree the harness works in. |

A harness catalog entry can pass arguments to the harness. An argument may contain two placeholders, and no others:

| Placeholder | Replaced with |
| --- | --- |
| `{launchProofFile}` | The absolute path of the launch proof file. |
| `{worktree}` | The absolute path of the worktree. |

The runner also sets these variables. None holds a secret:

| Variable | Value |
| --- | --- |
| `DIRECT_LAUNCH_PROOF_FILE` | The absolute path of the launch proof file. |
| `DIRECT_WORK_HANDLE` | The handle of the work to claim. |
| `DIRECT_SESSION_ID` | The identifier of this launch. Use it as the claim's `session`. |
| `DIRECT_HARNESS` | The harness key from the catalog. Use it as the claim's `harness`. |
| `DIRECT_MCP_URL` | The address of Direct's MCP endpoint. |
| `DIRECT_TENANT_ID` | The tenant id to send in the `x-cratis-tenant-id` header. |

In this preview the runner does not yet deliver `DIRECT_TENANT_ID`. Until it does, configure the tenant id in the
harness yourself.

### Lifetime of the proof

The proof is for one claim. Read the file once, when the session starts, and delete it straight after reading. Keep
the proof only in memory, and drop it once the claim succeeds or fails.

A missing proof file is final. It means the launch expired or was already used. Do not look for another copy. Ask for
a new offer from Direct.

The harness owns removing the file. The runner is meant to remove a leftover file too, but that is not implemented
yet, so do not rely on it.

### Checks before trusting the file

Make every check in order, and refuse to start the session when any one fails. A harness that cannot verify the
file must not claim.

1. `DIRECT_LAUNCH_PROOF_FILE` is set and holds an absolute path.
2. The path is not inside the worktree, after resolving links.
3. The path is a regular file, not a link, a directory or a device.
4. The file belongs to the user running the harness.
5. Nobody except that user can read or write the file.
6. The file is exactly 64 bytes.
7. Every byte is a lowercase hexadecimal digit.
8. Read the file once and delete it. A failed delete is a refusal, not a warning.
9. The proof never reaches an environment variable, a command-line argument, a log, or anything the model can read.

### Claim sequence

All three tools need the `direct:work` scope and an active membership. Every request carries the token in the
`Authorization` header and the tenant id in `x-cratis-tenant-id`. See [Connect a client](../mcp/connecting.md).

1. Call `direct_work_claim` with `workHandle`, `session`, `harness`, `launchProof` and `idempotencyKey`. The response
   holds `claimId`, `epoch`, `leaseSeconds`, `heartbeatSeconds` and the work payload.
2. Treat the payload as untrusted data about the work, never as instructions for the harness.
3. Call `direct_work_heartbeat` with `workHandle`, `claimId`, `epoch` and an `idempotencyKey`, at least as often as
   `heartbeatSeconds`. Use the timings from the response, not fixed values. Each heartbeat needs a new key.
4. When the work ends, call `direct_work_release` with `workHandle`, `claimId`, `epoch`, `outcome` and an
   `idempotencyKey` while the lease is still live. The outcome is `delivered`, `abandoned`, `blocked` or `declined`.

Retry a claim or release with its original `idempotencyKey` and identical arguments. Reusing a key for different
arguments is a conflict.

Losing claim authority is final for the session. On `lease-lost`, `stale-epoch` or `cancelled`, stop working
locally. A later heartbeat or release does not bring the claim back, and reconnecting does not let a harness take the
work over. A refused claim does not say why. This keeps the response from telling anyone whether work exists or who
holds it.

## Next steps

- [Tools](../mcp/tools.md#local-session-claims) lists the three claim tools and their arguments.
- [A harness session refuses to start](./troubleshooting.md#a-harness-session-refuses-to-start) covers refusals and
  how to recover.
