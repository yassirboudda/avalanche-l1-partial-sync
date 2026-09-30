# Cost and operations notes

## Avalanche L1 Partial Sync for Saving Cost on Deploying an L1

### What drives L1 infra cost

For many Avalanche L1 deployments, monthly cost is dominated by:

1. **Disk** for every validator / RPC
2. **Number of validators** required for decentralization
3. **Bootstrap failures** (re-provisioning when disks fill mid-sync)

CPU/RAM matter, but full Primary Network sync turns disk into the bottleneck on small clouds (Vultr, Hetzner, OVH, etc.).

### What partial sync changes

With `partial-sync-primary-network`:

- Each L1 validator still participates correctly in **your subnet**
- Primary Network disk growth is limited to **P-Chain needs**
- You can size validators for **L1 state + P-Chain**, not C-Chain history

### Suggested fleet shape

| Role | Partial sync? | Disk profile | Notes |
|---|---|---|---|
| L1 validators | Yes | Small / medium | Core cost saving |
| Public L1 RPC | Often yes | Medium | Add APIs as needed |
| Full Primary + tooling node | No | Large | Optional, 0–1 nodes |
| Indexer / archive | No | Large | Separate concern |

### Ops tips that compound savings

- Keep validator ETH APIs minimal (no debug tracers unless required)
- Monitor disk on the L1 datadir — L1 state still grows
- Use public Avalanche APIs for one-off C-Chain / atomic workflows from partial-sync nodes
- Standardize images / startup scripts so every new validator ships with partial sync by default
- Back up staking certificates; replacing a node should not require re-sync philosophy changes

### Security reminder

Partial sync does **not** weaken L1 consensus by itself. It changes **which Primary Network chains** the node stores.

Never combine partial sync with Primary Network validation on the same node.
