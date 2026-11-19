# RB-07 · New app version: install, signal, activate

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| A release that introduces a new **app version** (e.g. `v11.0.0` → app version 11) | ~15 min of work, then days of waiting | **High** if skipped | ~1–2 min at install, none at activation |

A new app version is not a binary swap you can do whenever. It is a coordinated upgrade through `x/signal`:

1. every validator installs the new multiplexer binary, which keeps running the **current** app version;
2. validators signal for the new version;
3. once 5/6 of voting power has signalled, anyone submits `try-upgrade`;
4. the chain switches a fixed number of blocks later.

A validator still on the old binary at that height stops. So does a bridge or light node on an old celestia-node.

## Before

```bash
export NEW=v11.0.0                  # mocha-5: v11.0.0-mocha
export TARGET=11                    # the app version the release introduces
export CHAIN_ID=celestia            # or mocha-5
export ARCH=x86_64
export FIBRE_HOME=/var/lib/celestia-fibre
```

- Read the release notes. For v11: **no configuration changes**, glibc ≥ 2.38 for the multiplexer and ≥ 2.34 for Fibre, Go 1.26.10 for source builds.
- Tell whoever runs your bridge or light nodes. For v11 they need **celestia-node `v0.36.0`** (mocha: `v0.36.0-mocha`) **before** activation. Older nodes reject v11 headers and stop syncing.

## Steps

**1. Install node and Fibre from the new release.** Same commands as [RB-03](RB-03-minor-upgrade.md): keep `.prev` copies, verify `checksums.txt`, stop Fibre then the node, swap both binaries, start the node then Fibre.

**2. Confirm the node still runs the old app version.** The multiplexer embeds the previous release and keeps using it until activation:

```bash
curl -s localhost:26657/abci_info | jq -r .result.response.app_version   # still 10
celestia-appd version                                                    # v11.0.0
```

**3. Signal** with the validator's account key:

```bash
celestia-appd tx signal signal $TARGET \
  --from=<validator-account-key> --chain-id=$CHAIN_ID \
  --gas=auto --gas-adjustment=1.5 --gas-prices=0.002utia
```

**4. Watch the tally:**

```bash
celestia-appd query signal tally $TARGET                 # voting power signalled vs. threshold
celestia-appd query signal missing-validators $TARGET    # who has not signalled yet
```

**5. Once the threshold is reached,** anyone can schedule the upgrade. It is enough for one validator to do it:

```bash
celestia-appd tx signal try-upgrade \
  --from=<validator-account-key> --chain-id=$CHAIN_ID \
  --gas=1000000 --gas-prices=0.002utia
```

`try-upgrade` needs about 300,000 gas. Upstream recommends `--gas 1000000`.

**6. Note the height:**

```bash
celestia-appd query signal upgrade                       # app version and activation height
```

For v11 the delay is **232,616 blocks on mainnet** (about 7.6 days) and **66,462 on Mocha** (about 53 hours).

## Verify

After the activation height:

```bash
curl -s localhost:26657/abci_info | jq -r .result.response.app_version   # 11
grpc_health_probe -addr=127.0.0.1:7980 -tls -tls-no-verify               # SERVING
```

Fibre needs nothing new for v11. If readiness fails right after the switch, follow [RB-06](RB-06-not-ready.md). Look first at `app` and `fibre_module`.

## Rollback

| When | Possible? |
|:--|:--|
| Before you signal | Yes. Put the `.prev` binaries back, as in RB-03 |
| After you signal, before activation | The binary can go back, but your signal stays on-chain. You will still need the new binary by the activation height |
| After activation | **No.** The chain runs the new app version, and an old binary cannot follow it |
