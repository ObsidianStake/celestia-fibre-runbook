# Reference · Signals

Every alert we run points at the card that answers it.

## Probes

```bash
grpc_health_probe -addr=<host>:7980 -tls -tls-no-verify                          # readiness
grpc_health_probe -addr=<host>:7980 -tls -tls-no-verify -service=fibre-liveness  # liveness
```

Install: `go install github.com/grpc-ecosystem/grpc-health-probe@latest`. Run the readiness probe from a machine outside your network, not just from the host.

## Alert → card

| Alert | Fires when | Severity | Open |
|:--|:--|:--|:--|
| Fibre not ready | readiness ≠ SERVING from outside, 2 min | page | [RB-06](../runbooks/RB-06-not-ready.md) |
| Fibre down | liveness fails | page | [RB-06](../runbooks/RB-06-not-ready.md) |
| Not registered | `query valaddr provider` comes back empty | warn | [RB-02](../runbooks/RB-02-register-host.md) |
| Public address changed | host IP differs from the registered one | warn | [RB-02](../runbooks/RB-02-register-host.md) |
| New release | a celestia-app release is published | info | [RB-03](../runbooks/RB-03-minor-upgrade.md) |
| Fibre disk | > 80 % used | warn | [RB-04](../runbooks/RB-04-object-storage.md) |
| Signer slow | `cometbft_privval_signing_latency_*` median > 10 ms, 10 min | warn | [RB-05](../runbooks/RB-05-remote-signer.md) if the signer is remote, otherwise host load |

Fibre itself pushes metrics over OTLP/HTTP (`--otel-endpoint`, base URL only). The metric list and dashboards are in the upstream [Fibre server reference](https://github.com/celestiaorg/celestia-app/blob/v10.6.0/fibre/cmd/README.md).
