# Release log

Each runbook release is tied to a celestia-app release. The tag reads `<upstream>-r<n>`: `v10.6.0-r1` is the first runbook cut against `v10.6.0`, and `v10.6.0-r2` would be a correction without a new upstream release.

| Runbook | Upstream (mainnet / mocha-5) | Checked on | Cards touched |
|:--|:--|:--|:--|
| `v10.6.0-r1` | `v10.6.0` / `v10.6.0-mocha` | 2026-11-05 | all (first cut) |

---

### `v10.6.0-r1`

**First cut**, written against v10.5.0 and re-checked on the day v10.6.0 shipped. On that day the Foundation announced the Mocha upgrade only. The mainnet tag was out, but not yet announced.

What v10.6.0 means for a Fibre operator: no new settings, no new flags, no new ports, no coordinated upgrade. It is a binary swap by [RB-03](runbooks/RB-03-minor-upgrade.md). Fibre now rejects `MsgDepositToEscrow` with a non-canonical signer or a non-`utia` denom, re-checks the payment promise budget when escrow balance drops, and the Fibre client no longer crashes on an incomplete provider response. celestia-core goes to v0.42.8. Source builds need Go 1.26.10.

| Card / file | What it covers |
|:--|:--|
| `anatomy/how-fibre-works.md` | One upload through the host · the six readiness checks |
| RB-01 | Bring Fibre up next to a running validator |
| RB-02 | Register or move the host. The `valaddr` entry is deleted only after 7+ days jailed and unbonded, or removal from staking |
| RB-03 | Minor upgrade of node + Fibre, with rollback. Example now `v10.5.0 → v10.6.0` |
| RB-04 | Move shards to an `https://` bucket |
| RB-05 | Remote signer over mutual TLS. Links point to celestia-core v0.42.8 |
| RB-06 | Not-ready triage by failing check |
| `reference/` | Ports, `fibre start` flags, `server_config.toml` incl. the `[health]` table (v10.5.0+), alert → card |
| `systemd/celestia-fibre.service` | `Restart=on-failure`, 60 s drain |
