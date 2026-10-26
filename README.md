<img src="https://raw.githubusercontent.com/ObsidianStake/celestia-fibre-runbook/main/media/title.svg?v=1" width="55%" alt="Celestia + Fibre">

**This is not an install guide.** It starts where those end: a synced, bonded Celestia validator on app version 10, and a Fibre server to run next to it. It is the runbook from our own [Mocha validator](https://celestia-dashboard.krews.xyz/celestia-mocha/validators/celestiavaloper17awy8avky6rmnxketpf46dywegd86sla44vx24), written down so the next person on call does not have to remember it.

## What do you need right now?

| Situation | Open |
|:--|:--|
| The chain is on v10 and Fibre is not running yet | [RB-01 · Bring Fibre up](runbooks/RB-01-bring-up.md) |
| Fibre runs, but clients can't find it | [RB-02 · Register or move the host](runbooks/RB-02-register-host.md) |
| A new `v10.x.y` release is out | [RB-03 · Minor upgrade](runbooks/RB-03-minor-upgrade.md) |
| The Fibre disk is filling up | [RB-04 · Object storage](runbooks/RB-04-object-storage.md) |
| The key is moving to a KMS or another host | [RB-05 · Remote signer, mTLS](runbooks/RB-05-remote-signer.md) |
| **An alert says Fibre is not ready** | [RB-06 · Not ready](runbooks/RB-06-not-ready.md) |

Every runbook card has the same shape: **trigger · time · risk · downtime**, then *Before*, *Steps*, *Verify*, *Rollback*.

## Three drawers

| Drawer | What's in it | Read it |
|:--|:--|:--|
| [`anatomy/`](anatomy/how-fibre-works.md) | How one upload travels through your host, what "ready" means | Once, before anything else |
| [`runbooks/`](runbooks) | RB-01 … RB-06 | When the situation comes up |
| [`reference/`](reference) | [Ports and config](reference/ports-and-config.md) · [alert → card](reference/signals.md) | When you need a value |

Plus [`systemd/celestia-fibre.service`](systemd/celestia-fibre.service).

## What the cards assume

If your layout differs, the cards still apply, but RB-04 and RB-05 stop being optional.

- Fibre runs **on the validator host** and talks to the signer over loopback.
- Shards go to a **dedicated NVMe**, not to a bucket.
- Fibre runs from the **same release** as the node and is upgraded in the same window.
- The host is registered by **DNS name** where possible, so an IP change is a DNS edit, not a transaction.

## Versions

Checked **2026-10-26** against [celestia-app releases](https://github.com/celestiaorg/celestia-app/releases) and the upstream [Fibre server reference](https://github.com/celestiaorg/celestia-app/blob/v10.5.0/fibre/cmd/README.md).

| Network | Chain ID | celestia-app + fibre |
|:--|:--|:--|
| Mainnet | `celestia` | `v10.5.0` |
| Mocha | `mocha-5` | `v10.5.0-mocha` |

Fibre needs glibc ≥ 2.34 (Ubuntu 24.04 is the upstream-tested OS).

## Out of scope

Installing `celestia-appd`, syncing, creating the validator, bridge nodes: follow [docs.celestia.org](https://docs.celestia.org/operate/consensus-validators/validator-node/). This repo picks up after that.

<sub>Obsidian Stake · [obsidianstake.xyz](https://obsidianstake.xyz) · [GitHub](https://github.com/ObsidianStake) · Corrections welcome as issues or PRs.</sub>
