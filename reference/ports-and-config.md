# Reference · Ports and config

## Ports

| Port | Process | Exposure |
|:--|:--|:--|
| **7980** | `fibre` (gRPC + TLS, health service) | **Public** |
| 26656 | `celestia-appd` P2P | Public |
| 9090 | `celestia-appd` app gRPC, no TLS | Loopback or trusted private network |
| 26669 | `celestia-appd` privval gRPC | Loopback, or mTLS on a private network |
| 26657 | `celestia-appd` RPC | Loopback |
| 26660 | `celestia-appd` Prometheus | Monitoring host |
| 6060 | `fibre --pprof` (opt-in) | Loopback |
| 4318 | OTLP/HTTP collector (yours) | Loopback / monitoring network |

## `fibre start` flags

Precedence: **flag > `server_config.toml` > default**.

| Flag | Env | Default |
|:--|:--|:--|
| `--home` | `FIBRE_HOME` | `~/.celestia-fibre` |
| `--app-grpc-address` | | `127.0.0.1:9090` |
| `--signer-grpc-address` | | `127.0.0.1:26669` |
| `--server-listen-address` | | port `7980` |
| `--log-level` | `FIBRE_LOG_LEVEL` | `info` |
| `--log-format` | `FIBRE_LOG_FORMAT` | `text` (`json` for shippers) |
| `--otel-endpoint` | `FIBRE_OTEL_ENDPOINT` | off. OTLP/HTTP base URL |
| `--pprof` | | off. `localhost:6060` when enabled |
| `--override-object-namespace` | | one-time, for bucket migrations |

## `$FIBRE_HOME/config/server_config.toml`

| Key | Default | Notes |
|:--|:--|:--|
| `max_connections` | `16` | See [RB-01](../runbooks/RB-01-bring-up.md), step 4 |
| `max_concurrent_streams` | `13` | Per connection |
| `min_upload_size` | `262144` | Bytes, max 128 MiB. Restart to apply |
| `storage_backend` | | `local` or `object`, new shards only |
| `signer_grpc_address` | | Same as the flag |
| `signer_grpc_ca_file` / `_cert_file` / `_key_file` | | All three or none |
| `signer_grpc_allow_insecure` | off | Lab only |

`[health]` (v10.5.0+):

| Key | Default |
|:--|:--|
| `check_interval` | `10s` |
| `probe_timeout` | `3s` |
| `max_block_age` | `2m` |

## Node keys Fibre depends on

| File | Key | Value for the default setup |
|:--|:--|:--|
| `app.toml` | `[grpc] enable` | `true` |
| `app.toml` | `[grpc] address` | `"127.0.0.1:9090"` |
| `config.toml` | `priv_validator_grpc_laddr` | `"127.0.0.1:26669"` (not `localhost`) |
| `config.toml` | `priv_validator_grpc_cert_file` / `_key_file` / `_client_ca_file` | Remote signer only |

`celestia-appd config sync --home … --fibre-home … [--dry-run]` adds missing keys to `config.toml` and `server_config.toml`. It keeps your values, makes backups, and never edits `app.toml`.

## On-chain

| Command | Purpose |
|:--|:--|
| `celestia-appd tx valaddr set-host <host>:7980 --from <key>` | Publish your Fibre host |
| `celestia-appd query valaddr provider <celestiavalcons…>` | Your registered host |
| `celestia-appd query valaddr providers` | All registered hosts |
| `celestia-appd comet show-address` | Your `celestiavalcons…` address |
