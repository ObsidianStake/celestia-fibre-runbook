# RB-05 · Put the signer on another host (mutual TLS)

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| Key moves to a KMS or a separate host | ~45 min | **High** | A node restart, plus whatever your KMS move needs |

We run Fibre on the validator host with the signer on loopback, and that needs none of this. This card is for the day the key moves.

## Before

- The signer supports `SignRawBytes` and signs with a median latency ≤ 10 ms.
- One CA, a **server** cert and key for the node's privval listener, and a **client** cert and key for Fibre.
- The server cert's SAN matches the exact address Fibre dials, e.g. `IP:10.0.0.5`.
- A private network path between Fibre and the node. Port 26669 is never public.
- **Only one place signs with the key.** Moving a key is when double signs happen.

## Steps

Node, `~/.celestia-app/config/config.toml`:

```toml
priv_validator_grpc_laddr = "10.0.0.5:26669"
priv_validator_grpc_cert_file = "/etc/celestia/privval/server.crt"
priv_validator_grpc_key_file = "/etc/celestia/privval/server.key"
priv_validator_grpc_client_ca_file = "/etc/celestia/privval/ca.crt"
```

Fibre, `$FIBRE_HOME/config/server_config.toml`:

```toml
signer_grpc_address = "10.0.0.5:26669"
signer_grpc_ca_file = "/etc/celestia/privval/ca.crt"
signer_grpc_cert_file = "/etc/celestia/privval/client.crt"
signer_grpc_key_file = "/etc/celestia/privval/client.key"
```

Change `--signer-grpc-address` in the Fibre unit to match, or remove it so the config file wins. Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart celestia-appd
sudo systemctl restart celestia-fibre
```

## Verify

- Readiness `SERVING`, check `signer` green.
- `cometbft_privval_signing_latency_*` median ≤ 10 ms, under real load.
- `fibre.server.sign.duration` close to what it was on loopback.

## Rollback

Set both sides back to `127.0.0.1:26669`, clear the TLS fields on both sides (all three, or startup fails), restart node then Fibre.

## Startup errors you will meet

| Error is about | Fix |
|:--|:--|
| Insecure / non-loopback signer | TLS fields missing on one side |
| Missing TLS file | Only one or two of the three fields set |
| Certificate name mismatch | SAN does not match the dialled address |
| `localhost` | Use an IP literal |

Upstream: [privval gRPC TLS](https://github.com/celestiaorg/celestia-core/blob/v0.42.3/docs/guides/privval-grpc-tls.md) · [Fibre signing](https://github.com/celestiaorg/celestia-app/blob/v10.5.0/fibre/cmd/README.md#signing)
