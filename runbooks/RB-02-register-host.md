# RB-02 · Register or move the Fibre host

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| First bring-up · new IP or DNS name · entry deleted after a long jail | ~5 min | Low | None |

Clients find your server only through `x/valaddr`. Unregistered means no traffic.

## Before

- The server is running **and reachable from outside** at the address you are about to publish:

  ```bash
  # from another machine
  grpc_health_probe -addr=<host>:7980 -tls -tls-no-verify
  ```

- The chain is on app version ≥ 10.
- The validator's account key is available to sign.

## Steps

```bash
celestia-appd tx valaddr set-host <host>:7980 \
  --from=<validator-account-key> --chain-id=<celestia|mocha-5> \
  --gas=auto --gas-adjustment=1.5 --gas-prices=0.002utia
```

Format rules: `host:port` only, with no `https://` and no path. Port 1–65535, at most 100 characters. A DNS name works and survives an IP change.

## Verify

```bash
celestia-appd query valaddr provider $(celestia-appd comet show-address)   # shows <host>:7980
celestia-appd query valaddr providers | grep "<host>"
```

The readiness check `registration` turns green on the next health cycle (10 s by default).

## When to run it again

| Situation | Again? |
|:--|:--|
| Restart, upgrade, reboot | No. Registration persists |
| New public IP and you registered an IP | **Yes** |
| DNS name registered and only the A record changed | No |
| Validator stayed jailed and unbonded for more than 7 days, or was removed from staking | **Yes**, the entry was deleted |
| Short jail, unjailed within 7 days | No |

## Rollback

Register the previous address with the same command. There is no downtime in either direction, but clients only see the change once the transaction is in a block.
