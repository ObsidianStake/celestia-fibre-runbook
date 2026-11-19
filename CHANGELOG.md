# Release log

Each runbook release is tied to a celestia-app release. The tag reads `<upstream>-r<n>`: `v10.6.0-r1` is the first runbook cut against `v10.6.0`, and `v10.6.0-r2` would be a correction without a new upstream release.

| Runbook | Upstream (mainnet / mocha-5) | Checked on | Cards touched |
|:--|:--|:--|:--|
| `v11.0.0-r1` | `v11.0.0` / `v11.0.0-mocha` | 2026-11-19 | new RB-07, README, RB-01, RB-03, links |
| `v10.6.0-r2` | `v10.6.0` / `v10.6.0-mocha` | 2026-11-10 | README only |
| `v10.6.0-r1` | `v10.6.0` / `v10.6.0-mocha` | 2026-11-05 | all (first cut) |

---

### `v11.0.0-r1`

**App version 11.** celestia-app `v11.0.0` is an emergency release cut from v10.x. It restricts accounts whose keys were exposed in the Keplr/Web3Auth social-login incident, so they can only send funds to a recovery address. Unlike v10.5 and v10.6, it is a **coordinated upgrade**: install, signal `11`, `try-upgrade`, and the switch comes 232,616 blocks later on mainnet. On Mocha it activated on 2026-11-19. For Fibre: no new settings, flags or ports.

| Card / file | Change |
|:--|:--|
| **RB-07** (new) | Install, signal, `try-upgrade`, watch the tally, verify after activation, and what can still be rolled back at each stage |
| `README.md` | Versions → `v11.0.0` / `v11.0.0-mocha`. New "new app version" row in the situation table. Bridge/light nodes need celestia-node `v0.36.0` before activation |
| RB-01 | `APP_VERSION` → `v11.0.0`. Trigger reads "app version 10 or later" |
| RB-03 | Points to RB-07 for new app versions |
| `reference/signals.md` | New alert: "Upgrade scheduled". "New release" routes to RB-03 or RB-07 |
| RB-04, RB-05, `reference/signals.md` | Upstream links → `v11.0.0` |

### `v10.6.0-r2`

**Mainnet upgrade announced.** The Foundation published the Mainnet Beta notice for `v10.6.0`. The Versions table now says "upgrade now" for mainnet as well. The runbook steps did not change: it is still a binary swap by [RB-03](runbooks/RB-03-minor-upgrade.md).

| Card / file | Change |
|:--|:--|
| `README.md` | Mainnet row: "not yet announced" → "announced, upgrade now". Checked-on date updated |

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
