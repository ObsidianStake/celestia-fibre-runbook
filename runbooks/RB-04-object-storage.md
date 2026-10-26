# RB-04 · Move shards to object storage

| Trigger | Time | Risk | Validator downtime |
|:--|:--|:--|:--|
| Local Fibre disk is the bottleneck | ~15 min + provider setup | Medium | None (Fibre restarts, consensus does not) |

We run `local` on a dedicated NVMe. This card is for when local disk runs out. The change is easy to make and awkward to undo, and the bucket becomes something Fibre needs in order to start.

## Before

- An S3-compatible bucket with an **`https://`** endpoint. `http://` is rejected since v10.5.0.
- Credentials with read, write, list and delete on that bucket.
- Reachable from the Fibre host: `curl -sI https://<endpoint>` answers.
- Decide the bucket and prefix **for good**. They are pinned while object shards exist.

## Steps

1. Back up the config:

   ```bash
   cp $FIBRE_HOME/config/server_config.toml{,.bak}
   ```

2. In `server_config.toml`, set `storage_backend = "object"` and fill the `[object_storage]` section: endpoint, bucket, prefix, credentials. Key names are in the upstream reference ([`fibre/cmd/README.md`](https://github.com/celestiaorg/celestia-app/blob/v10.5.0/fibre/cmd/README.md)). Copy them from there, not from memory.

3. Restart:

   ```bash
   sudo systemctl restart celestia-fibre
   journalctl -u celestia-fibre -n 50 -o cat
   ```

   An unreachable bucket stops startup here, with an error. That is the intended behaviour.

## Verify

- Readiness `SERVING`, check `store` green.
- New objects appear under your prefix as uploads arrive.
- `fibre.server.store.*` and `fibre.server.backend.get.*` latencies look sane.

## Rollback

Set `storage_backend = "local"` and restart. But:

- shards already written to the bucket **stay there** and are still served from there;
- keep `[object_storage]` configured and the credentials valid **until those shards are pruned**;
- do not change bucket, prefix or endpoint meanwhile, or startup is blocked by the saved namespace.

Full migration between buckets: stop Fibre, move every object, start once with `--override-object-namespace`. Follow the upstream steps exactly.
