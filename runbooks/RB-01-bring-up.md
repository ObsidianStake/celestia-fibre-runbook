# RB-01 · Bring Fibre up on a running validator

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| Chain reached app version 10, or a new validator host | ~20 min | Low | One `celestia-appd` restart, only if gRPC was off |

Starting point: a synced, bonded validator on the multiplexer build of `celestia-app`. Installing the node itself is upstream's job: [docs.celestia.org](https://docs.celestia.org/operate/consensus-validators/validator-node/).

## Before

```bash
export APP_VERSION=v10.5.0          # mocha-5: v10.5.0-mocha
export ARCH=x86_64                  # or arm64
export FIBRE_HOME=/var/lib/celestia-fibre

curl -s localhost:26657/abci_info | jq -r .result.response.app_version             # ≥ 10
sed -n '/^\[grpc\]/,/^\[/p' ~/.celestia-app/config/app.toml                         # enable = true, 127.0.0.1:9090
grep priv_validator_grpc_laddr ~/.celestia-app/config/config.toml                   # "127.0.0.1:26669"
ldd --version | head -n1                                                           # glibc ≥ 2.34
df -h "$FIBRE_HOME"                                                                # its own disk
```

Any check fails → fix it first. Turning on app gRPC means editing `app.toml` and restarting `celestia-appd`. Note that `config sync` never touches `app.toml`.

## Steps

**1. Install the binary** from the same release as the node:

```bash
BASE=https://github.com/celestiaorg/celestia-app/releases/download/$APP_VERSION
mkdir -p /tmp/fibre && cd /tmp/fibre
curl -fsSLO $BASE/fibre_Linux_$ARCH.tar.gz
curl -fsSLO $BASE/checksums.txt
sha256sum --ignore-missing -c checksums.txt
tar -xzf fibre_Linux_$ARCH.tar.gz
sudo install -m 0755 "$(find . -type f -name fibre | head -n1)" /usr/local/bin/fibre
fibre version
```

**2. Fill in config keys** on both sides:

```bash
celestia-appd config sync --home ~/.celestia-app --fibre-home $FIBRE_HOME --dry-run
celestia-appd config sync --home ~/.celestia-app --fibre-home $FIBRE_HOME
```

**3. Start it under systemd** with [`systemd/celestia-fibre.service`](../systemd/celestia-fibre.service). Edit `User` and `--home` first.

```bash
sudo cp systemd/celestia-fibre.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now celestia-fibre
```

**4. Leave the caps alone at first.** `max_connections = 16` and `max_concurrent_streams = 13` are sized upstream for a 32 GiB host. On a bigger host, grow `max_connections` and keep streams where they are: each upload holds a connection on all 16 of its signers at once, so readers only get in when there are connections beyond those 16. Restart `celestia-fibre` after any change and watch real memory for a day.

**5. Open the port.**

```bash
sudo ufw allow 7980/tcp
```

**6. Register.** Run [RB-02](RB-02-register-host.md).

## Verify

```bash
systemctl is-active celestia-fibre                                                  # active
grpc_health_probe -addr=127.0.0.1:7980 -tls -tls-no-verify -service=fibre-liveness  # SERVING
```

From another machine:

```bash
grpc_health_probe -addr=<public-ip>:7980 -tls -tls-no-verify                        # SERVING after RB-02
```

## Rollback

Fibre is a separate process. Stopping it does not touch consensus.

```bash
sudo systemctl disable --now celestia-fibre
```

The validator keeps signing blocks. It just serves no Fibre traffic, and readiness elsewhere will show it missing.
