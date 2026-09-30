# Avalanche L1 Partial Sync for Saving Cost on Deploying an L1

Run Avalanche **L1 validators** on small VPS disks by enabling **Primary Network partial sync**.

Instead of downloading the full Primary Network (especially the C-Chain), an L1-only node syncs what it actually needs: the **P-Chain** plus your **L1 / subnet**. That cuts bootstrap time and monthly storage cost drastically.

Battle-tested on production L1s including **Amichain**, **Martian Chain**, and **CodeNekt**.

---

## Why this matters

A default AvalancheGo node joining Mainnet will sync **X-Chain + P-Chain + C-Chain**.

For a node that only validates (or serves) an **Avalanche L1**:

| What the node needs | Why |
|---|---|
| **P-Chain** | Validator set, staking / continuous fees, Warp / ICM, subnet tracking |
| **Your L1 blockchain** | Blocks, state, RPC, consensus for your chain |
| **Full C-Chain / X-Chain** | Usually **not required** for L1-only validators |

Full Primary Network sync is the expensive part. On modest cloud disks it can fill the machine before the L1 ever becomes healthy.

**Partial sync** flips that: Primary Network sync stays limited to the P-Chain.

---

## The flag

```bash
--partial-sync-primary-network
```

Or in `node.json`:

```json
{
  "partial-sync-primary-network": true,
  "track-subnets": "<YOUR_SUBNET_ID>"
}
```

Official behavior (AvalancheGo): *Partial sync enables non-validators to optionally sync only the P-chain on the primary network.*

> **Critical:** do **not** enable this if the node is (or will become) a **Primary Network validator**. AvalancheGo treats that combination as unhealthy. Partial sync is for **L1 / subnet nodes that are not Primary Network validators**.

---

## Architecture (L1-only node)

```text
┌──────────────────────────────────────────────┐
│                 AvalancheGo                  │
│                                              │
│  Primary Network                             │
│  ├─ P-Chain .............. synced (needed)   │
│  ├─ C-Chain .............. skipped via       │
│  │                         partial sync      │
│  └─ X-Chain .............. skipped           │
│                                              │
│  Tracked subnet / L1                         │
│  └─ Your blockchain ...... fully synced      │
└──────────────────────────────────────────────┘
```

Optional: keep **one larger RPC node** (or use public APIs) when you need full C-Chain reads / atomic imports. L1 validators can stay small.

---

## Cost impact

Rough order-of-magnitude for **Mainnet L1 validators** (disk-driven VPS pricing varies by provider):

| Setup | Typical disk pressure | Practical VPS profile |
|---|---|---|
| Full Primary Network sync + L1 | High (C-Chain dominates) | Large SSD, higher monthly cost |
| **Partial sync + L1** | Low–moderate (P-Chain + L1 only) | Small/medium SSD, much cheaper fleet |

What you save in practice:

1. **Disk** — no full C-Chain history on every validator
2. **Bootstrap time** — nodes become useful faster
3. **Fleet cost** — more validators / RPCs for the same budget
4. **Ops risk** — fewer “disk full during sync” failures on cheap clouds

This is one of the highest-leverage knobs for **lowering the cost of deploying and operating an Avalanche L1**.

---

## Quick start

### 1. Install AvalancheGo + subnet-evm plugin

Use matching AvalancheGo / subnet-evm versions for your network. Place the plugin at:

```text
~/.avalanchego/plugins/<VM_ID>
```

### 2. Write node config

See [`examples/node.partial-sync.json`](examples/node.partial-sync.json):

```json
{
  "partial-sync-primary-network": true,
  "track-subnets": "REPLACE_WITH_SUBNET_ID",
  "http-host": "127.0.0.1",
  "http-port": 9650,
  "staking-port": 9651,
  "log-level": "info"
}
```

### 3. Write L1 chain config (validators)

Keep ETH APIs minimal on validators to save CPU/RAM:

```json
{
  "eth-apis": [
    "eth",
    "eth-filter",
    "net",
    "web3",
    "internal-eth",
    "internal-blockchain",
    "internal-transaction"
  ]
}
```

Path:

```text
~/.avalanchego/configs/chains/<BLOCKCHAIN_ID>/config.json
```

### 4. Run with systemd

See [`examples/avalanchego.service`](examples/avalanchego.service).

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now avalanchego
journalctl -u avalanchego -f
```

### 5. Verify

- Node health becomes ready after **P-Chain + L1** bootstrap (not full C-Chain).
- `track-subnets` includes your subnet ID.
- Confirm the node is **not** registered as a Primary Network validator.

If you need C-Chain UTXO / atomic operations from a partial-sync validator, query a **public API** or a dedicated full node (same pattern used in production: partial-sync validators + public P/C APIs when required).

---

## Production checklist

- [ ] `partial-sync-primary-network: true`
- [ ] `track-subnets` set to your L1 subnet ID
- [ ] subnet-evm plugin installed under the correct VM ID
- [ ] chain config present for your blockchain ID
- [ ] node is **not** a Primary Network validator
- [ ] disk monitoring still enabled (L1 state grows over time)
- [ ] at least one full/public endpoint available for C-Chain ops if needed
- [ ] backups of staking TLS keys (`staker.crt` / `staker.key`)

---

## When NOT to use partial sync

| Scenario | Use partial sync? |
|---|---|
| L1 / subnet validator only | **Yes** |
| L1 RPC that does not need C-Chain | **Yes** |
| Primary Network validator | **No** |
| Archive / indexer needing full C-Chain | **No** |
| Relayer / tooling that must read local C-Chain state | Prefer a full node or public RPC |

---

## Related Avalanche concepts

- **Partial sync Primary Network** — sync P-Chain only on Primary Network
- **State sync (subnet-evm)** — separate feature for faster EVM state bootstrap on the L1 itself (`state-sync-enabled`)
- **Track subnets** — subscribe the node to your L1 subnet

You can combine **partial sync (Primary Network)** with sensible L1 chain configs for the best cost profile.

---

## Repo layout

```text
examples/
  node.partial-sync.json   # AvalancheGo node config
  chain.config.json        # minimal L1 ETH APIs
  avalanchego.service      # systemd unit
docs/
  COST_AND_OPS.md          # cost model + ops notes
```

---

## Credits / real-world use

This guide documents a pattern used to deploy Avalanche L1s on storage-constrained servers for:

- Amichain
- Martian Chain
- CodeNekt L1

The goal: make sovereign L1 infrastructure affordable without sacrificing validator security on the L1 itself.

---

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

This is an operational pattern around official AvalancheGo flags. Always validate versions, security, and consensus requirements for your network before production use.
