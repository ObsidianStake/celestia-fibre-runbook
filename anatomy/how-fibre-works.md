# Anatomy · How Fibre works on a validator

Read this once. Everything in the runbooks assumes it.

## What Fibre adds

Before app version 10, a Celestia validator did one job: it ordered and signed blocks. From v10 on it has a second job. It runs a **Fibre server** that takes blob shards directly from clients, stores them, and signs that it holds them.

The chain does not carry those shards. It carries the commitments and the signatures. So a validator that runs Fibre badly does not break consensus. It just stops being useful to Fibre clients, and they route around it.

## The path of one upload

```
client
  │ 1. opens TLS gRPC to :7980 on 16 validators' Fibre servers (one upload = 16 signers)
  ▼
fibre ── 2. checks the upload against chain state ───────────▶ celestia-appd  app gRPC :9090
  │
  ├──── 3. writes the shard ─────────────────────────────────▶ Fibre disk, or an https bucket
  │
  └──── 4. asks for a signature over the shard ──────────────▶ celestia-appd  privval gRPC :26669
                                                                    │
                                                    validator key (file or KMS)
```

Three conclusions follow.

1. **Fibre is only as fast as your signer.** Step 4 happens on every upload. Upstream requires median signing latency ≤ 10 ms.
2. **Fibre is only as current as your node.** Step 2 reads chain state. A lagging node makes Fibre report `chain_stalled`.
3. **Memory goes to step 1.** Each open stream can hold up to 132 MiB while it receives. That is why the connection caps exist. See step 4 of [RB-01](../runbooks/RB-01-bring-up.md).

## How clients find you

Through the chain. `x/valaddr` maps each validator to a Fibre host. You publish yours with `celestia-appd tx valaddr set-host <host>:7980`. A server that is running but not registered gets **no traffic at all**. A registered host that is unreachable makes you the validator whose shard is missing.

## What "ready" means

Fibre's gRPC health service on :7980 answers two questions:

| Service | Question | Fails when |
|:--|:--|:--|
| *(default)*, readiness | Can I take new uploads? | Any of the six checks below fails |
| `fibre-liveness` | Is the process sane? | The server itself is broken |

The six readiness checks, in the order they usually break:

| Check | Depends on |
|:--|:--|
| `app` | `celestia-appd` running, app gRPC on :9090 |
| `signer` | privval gRPC on :26669 answering |
| `store` | Fibre disk writable / bucket reachable |
| `fibre_module` | Chain on app version ≥ 10 |
| `validator` | Your validator bonded, not jailed |
| `registration` | A `valaddr` host registered for you |

Not-ready still **serves downloads** of shards already stored. It only stops new uploads.

## Two processes, one release

`celestia-appd` and `fibre` ship in the same release and are versioned together. Upgrade them together, and run `config sync` after each upgrade. See [RB-03](../runbooks/RB-03-minor-upgrade.md).

| Process | Home | Config written by |
|:--|:--|:--|
| `celestia-appd` | `~/.celestia-app` | `celestia-appd init`, then `config sync` |
| `fibre` | `~/.celestia-fibre` (`FIBRE_HOME`) | first `fibre start`, then `config sync --fibre-home` |
