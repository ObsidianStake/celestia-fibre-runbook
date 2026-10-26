# RB-03 · Minor upgrade of node + Fibre

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| New `v10.x.y` release (not a new app version) | ~10 min | Medium | ~1–2 min of missed blocks |

Example: `v10.4.0 → v10.5.0`. Node and Fibre come from the same release and are swapped together. A **new app version** is a different procedure (`x/signal`, multiplexer). See upstream release notes.

## Before

```bash
export NEW=v10.5.0                  # mocha-5: v10.5.0-mocha
export ARCH=x86_64
export FIBRE_HOME=/var/lib/celestia-fibre

# Keep the old binaries for rollback
sudo cp /usr/local/bin/celestia-appd /usr/local/bin/celestia-appd.prev
sudo cp /usr/local/bin/fibre         /usr/local/bin/fibre.prev

# Download and verify while the node is still running
BASE=https://github.com/celestiaorg/celestia-app/releases/download/$NEW
mkdir -p /tmp/$NEW && cd /tmp/$NEW
curl -fsSLO $BASE/celestia-app_Linux_$ARCH.tar.gz
curl -fsSLO $BASE/fibre_Linux_$ARCH.tar.gz
curl -fsSLO $BASE/checksums.txt
sha256sum --ignore-missing -c checksums.txt
tar -xzf celestia-app_Linux_$ARCH.tar.gz && tar -xzf fibre_Linux_$ARCH.tar.gz
```

Read the release notes for config changes. v10.5.0, for example, added a `[health]` table and made object storage https-only.

## Steps

```bash
# 1. Stop Fibre, then the node
sudo systemctl stop celestia-fibre celestia-appd

# 2. Swap both binaries
sudo install -m 0755 "$(find . -type f -name celestia-appd | head -n1)" /usr/local/bin/celestia-appd
sudo install -m 0755 "$(find . -type f -name fibre | head -n1)"         /usr/local/bin/fibre

# 3. Add new config keys (keeps your values, backs files up)
celestia-appd config sync --home ~/.celestia-app --fibre-home $FIBRE_HOME --dry-run
celestia-appd config sync --home ~/.celestia-app --fibre-home $FIBRE_HOME

# 4. Node first, then Fibre
sudo systemctl start celestia-appd
sudo systemctl start celestia-fibre
```

## Verify

```bash
celestia-appd version; fibre version                                     # both = $NEW
curl -s localhost:26657/status | jq .result.sync_info.catching_up        # false within a minute or two
grpc_health_probe -addr=127.0.0.1:7980 -tls -tls-no-verify               # SERVING
```

Then check on your explorer that blocks are being signed again.

## Rollback

```bash
sudo systemctl stop celestia-fibre celestia-appd
sudo mv /usr/local/bin/celestia-appd.prev /usr/local/bin/celestia-appd
sudo mv /usr/local/bin/fibre.prev         /usr/local/bin/fibre
sudo systemctl start celestia-appd celestia-fibre
```

Minor releases are not consensus-breaking, so rolling back within the same app version is safe. Keys added by `config sync` stay in the files. If the older binary refuses one of them, restore the backup that `config sync` left next to the original.
