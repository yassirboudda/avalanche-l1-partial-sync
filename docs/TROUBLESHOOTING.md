# Troubleshooting

## Node unhealthy with partial sync

**Symptom:** health endpoint fails; logs mention partial sync / validator.

**Cause:** partial sync is enabled on a node that is (or tries to become) a Primary Network validator.

**Fix:** use partial sync only for L1-only nodes. Primary Network validators need full sync.

## Bootstrap stuck on C-Chain

**Symptom:** waiting forever on C-Chain bootstrap.

**Cause:** partial sync not actually enabled, or config file not loaded.

**Fix:**

```bash
# Confirm the process flags / config
ps aux | grep avalanchego
cat ~/.avalanchego/configs/node.json
```

Ensure:

```json
"partial-sync-primary-network": true
```

and that AvalancheGo is started with `--config-file=...` pointing at that file.

## Local RPC missing C-Chain data

**Expected** under partial sync. Use:

- `https://api.avax.network` (or another public endpoint), or
- a dedicated full node

for C-Chain / atomic workflows.

## L1 not appearing

Checklist:

1. `track-subnets` includes the correct subnet ID
2. subnet-evm plugin installed under the correct VM ID
3. peers / network connectivity OK
4. blockchain ID config directory matches your L1

## Disk still growing fast

Partial sync removes Primary C-Chain history, not L1 growth.

Inspect:

```bash
du -sh ~/.avalanchego/db/* | sort -h
```

If L1 state dominates, plan pruning / retention / larger disk for the subnet itself.
