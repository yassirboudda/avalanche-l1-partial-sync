# Cost model — Partial Sync L1

## The problem

Deploying an Avalanche L1 traditionally implies AvalancheGo nodes that also carry Primary Network state. Full C-Chain sync dominates disk and makes “small VPS L1” unrealistic.

## The optimization

Enable **partial sync** so L1 validators keep:

- P-Chain (required)
- L1 subnet state (required)

…and skip full Primary Network C-Chain sync (not required for L1-only validation).

## Rough comparison

Use these as planning brackets, not guarantees:

| Item | Full Primary + L1 | Partial sync + L1 |
|------|-------------------|-------------------|
| Bootstrap | Long (C-Chain heavy) | Much shorter |
| Disk | Large & growing | Sized mainly to L1 |
| VPS class | High storage SKUs | Commodity VPS viable |
| Ops complexity | Simpler local C-Chain access | Need public/full node for C-Chain ops |

### Example monthly thinking

If a full-sync node forces you into a **large SSD** instance, and partial sync lets you drop to a **standard VPS**, the monthly delta is often:

- storage surcharge avoided
- ability to run **more validators** for the same budget
- lower time-to-online for new regions / replacements

Track your own numbers:

```bash
df -h ~/.avalanchego
du -sh ~/.avalanchego/db
```

Compare a partial-sync L1 node vs any full Primary Network node you already operate.

## What still costs money

Partial sync does **not** eliminate:

- L1 block / state growth on disk
- bandwidth for P-Chain + L1 peers
- validator stake / continuous fee economics on Avalanche
- explorer, relayer, RPC replicas, monitoring

It removes the **Primary Network C-Chain storage tax** from L1 validator machines.

## Recommended sizing mindset

1. Size disk for **L1 headroom** (growth + logs + upgrades).
2. Keep partial sync on L1 validators / L1 RPCs.
3. Run **one** shared full/C-Chain-capable node (or use public APIs) for rare C-Chain jobs.
4. Separate “validate cheaply” from “archive everything”.
