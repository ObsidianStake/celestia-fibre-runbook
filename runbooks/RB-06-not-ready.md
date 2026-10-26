# RB-06 · Fibre is not ready

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| Readiness ≠ `SERVING`, or the "Fibre not ready" alert | 5–30 min | Depends on the cause | Usually none |

Not-ready means **no new uploads come to you**. Downloads of stored shards keep working. Consensus is not affected unless the cause is the node itself.

## 1. Which check?

```bash
grpc_health_probe -addr=127.0.0.1:7980 -tls -tls-no-verify                          # from the host
grpc_health_probe -addr=<public-host>:7980 -tls -tls-no-verify                      # from outside
grpc_health_probe -addr=127.0.0.1:7980 -tls -tls-no-verify -service=fibre-liveness  # process sane?
journalctl -u celestia-fibre -n 100 -o cat
```

| From host | From outside | Go to |
|:--|:--|:--|
| SERVING | fails | **Network**: firewall, security group, NAT, wrong registered host |
| NOT_SERVING | — | The failing check, below |
| no answer | no answer | **Process**: `systemctl status celestia-fibre`, liveness |

The logs name the failing check. `--log-level debug` names it louder.

## 2. By check

| Check | Most likely | Do |
|:--|:--|:--|
| `app` | Node down or app gRPC off | `systemctl status celestia-appd` · `[grpc]` in `app.toml` |
| `signer` | Signer address mismatch, KMS down | Compare `priv_validator_grpc_laddr` with Fibre's signer address · KMS health |
| `store` | Disk full or read-only · bucket unreachable | `df -h $FIBRE_HOME` · `curl -sI https://<endpoint>` |
| `fibre_module` | Chain below app v10 (or wrong network) | `abci_info` app version · chain ID |
| `validator` | Jailed or unbonded | Explorer / `query staking validator`. If jailed, fix downtime, then unjail |
| `registration` | Never registered, or the entry was deleted after 7+ days jailed and unbonded | [RB-02](RB-02-register-host.md) |

`chain_stalled` in the logs means the latest block is older than `max_block_age` (2 min). That is a **node** problem: peers, disk, catching up.

## 3. Process will not start

| Log says | Fix |
|:--|:--|
| `GLIBC_2.34 not found` | Newer OS |
| insecure / non-loopback signer | `127.0.0.1:26669` on both sides, or [RB-05](RB-05-remote-signer.md) |
| object storage / bucket | `https://`, credentials, reachability. See [RB-04](RB-04-object-storage.md) |
| object namespace | Bucket/prefix/endpoint changed while shards remain. Change back |
| `address already in use` | `sudo ss -ltnp 'sport = :7980'` |

## Close the incident

- Readiness `SERVING` from **outside** the host.
- Note the cause, and whether an alert would have caught it earlier.
