# Movement Networks

Configuration files and genesis data for connecting to Movement Network environments.

## Networks

| Network | Status | Description |
|---------|--------|-------------|
| `mainnet/` | Active | Movement mainnet - production network |
| `testnet/` | Active | Movement testnet - community testing and validation |
| `devnet/` | Active | Movement devnet - internal testing by Movement Labs |

## Files Per Network

Each network directory contains:

| File | Description |
|------|-------------|
| `genesis.blob` | Genesis block data |
| `waypoint.txt` | Current waypoint for state sync |
| `genesis_waypoint.txt` | Genesis waypoint |
| `configs/fullnode.yaml` | Standard public fullnode configuration |
| `configs/archival-fullnode.yaml` | Archival fullnode configuration (complete history) |

## Seed peers

The fullnode configs bootstrap from the four Movement validator fullnodes (VFNs) through fixed DNS
seeds and keep on-chain discovery on (`discovery_method: "onchain"`), so a node finds the VFNs from the
seeds when it starts and follows the on-chain `0x1::stake::ValidatorSet` after it syncs. Each seed is
keyed by its validator's **account address** (the peer id on-chain discovery uses), has role `Upstream`,
and puts the VFN's network key in the address: `/dns/<name>/tcp/6182/noise-ik/<network key>/handshake/0`.
Keying seeds this way makes the seed and the on-chain entry the same peer, so the node keeps one stable
connection to each VFN (tested 2026-10-22).

| Network | Name | Seed peer id (validator account) | VFN network key |
|---------|------|----------------------------------|-----------------|
| mainnet | `consensus-1.mainnet.movementnetwork.xyz` | `830bfd0cd58b06dc938d409b6f3bc8ee97818ffcf9b32d714c068454afb644c7` | `9967ebf40ac8c2ccb38709488952da1826176584ea3067b63b1695362ecb3d1f` |
| mainnet | `consensus-2.mainnet.movementnetwork.xyz` | `1ef54ef84e7fb389095f83021755dd71bb51cbfbc8124a4349ec619f9d901f1f` | `6ebcb891dde06e126939099f0a17f42273e036f20a0fc13d80ae6f60ed270541` |
| mainnet | `consensus-3.mainnet.movementnetwork.xyz` | `39f116ee9ef048895bff51a5ce62229d153a6fe855798fa75810fd2b85008b9c` | `340f8291558e676d8ab1ffd291a9419bbb2f708aa737e4cbd479bbe525445f39` |
| mainnet | `consensus-4.mainnet.movementnetwork.xyz` | `ccba2d929183a642f64d10d27bae0947c112ed7f5427ca3c64a1f0dd0b4b76ea` | `5eb1d58f4bed9ed88c8416c3018ee5470552a39af8898b88bb97ddbb3b4bcd3a` |
| testnet | `consensus-1.testnet.movementnetwork.xyz` | `bd7327ed25db839bd97d11f621afd7c6bd9ef7fa9e4a5044e02bc4e449ad18a6` | `5f3ccf14d5217249f94bdd752c2fa42d7e309522845001b62f0a05fbee7e1f2b` |
| testnet | `consensus-2.testnet.movementnetwork.xyz` | `5fb9e7d3c56e2036549b1fd0c4ca3b920ca5f7a1efeb5a14f87a6ab8a39974ba` | `5f6930a1eff6d52bb452434dbbd7fbfd86cff9b63c19513c00d8ebf70dcc1f4c` |
| testnet | `consensus-3.testnet.movementnetwork.xyz` | `d963d9415c267862a1d72ccf9f26aa1c4091442f0bbc73cbcc59bfe734a00f7` | `ba9898cd14fe8b995991fbed8f84f6d981c62faa82e7fca1a95995aef1b55134` |
| testnet | `consensus-4.testnet.movementnetwork.xyz` | `a1ef53a76fe31c0844c7b87988a3e2b905287ec687a8e081d793597eb351ed5c` | `9264dad024936304469b904b8c80366246dff8121de2cfa4cacad96d67b85f06` |

These keys match the `fullnode_addresses` registered in the on-chain `0x1::stake::ValidatorSet`.
Do not add other peers, and do not keep the older single `consensus.mainnet...` seed: it still
resolves for existing configs but is not part of the recommended set, and
`consensus.testnet.movementnetwork.xyz` does not exist.

## Restore Scripts

| Script | Description |
|--------|-------------|
| `l1_restore.sh` | Restore L1 node data from Movement Labs backups |
| `l2_restore.sh` | Restore L2 node data from Movement Labs backups (deperecated since l1 migration) |

### L1 Database Restore

To bootstrap a fullnode or archival node with existing blockchain data:

```bash
# For testnet (default)
./l1_restore.sh testnet ./data

# For mainnet
./l1_restore.sh mainnet ./data
```

**Prerequisites:**

- [restic](https://restic.net/) installed
- Sufficient disk space (mainnet ~700GB, testnet ~260GB)

The backups are updated hourly and maintain 14 days of snapshot history.

### If you restored before 2026-10-14: delete the shared node identity

Snapshots taken of the whole data directory include the backup node's persisted network
identity, `db/ephemeral_identity_key`. Every node restored from such a snapshot presents the
same peer id to the Movement validator fullnodes, which keep one connection per peer id and
close the older one whenever another node with the same id connects. Affected nodes see
`NoConnectedPeers` errors and frequent reconnects, and they slow down every other node that
shares the id. This affects mainnet restores taken on or after 2026-05-05 and testnet restores
from snapshots dated 2026-01-20 to 2026-02-06.

The fix is a one-time change on your node:

```bash
# Stop the node, then remove the copied identity from its data directory
rm -f /opt/data/aptos/db/ephemeral_identity_key   # adjust to your data_dir
# Start the node; it generates its own key and keeps it from then on
```

No configuration change is needed. `l1_restore.sh` now removes the file automatically after
a restore.

## Node Types

### Standard Fullnode

A standard public fullnode (PFN) syncs recent blockchain state and maintains pruned history.

- Use `configs/fullnode.yaml` configuration
- Faster initial sync via state snapshots
- Lower storage requirements

### Archival Fullnode

An archival fullnode maintains complete blockchain history from genesis by disabling ledger pruning.

- Use `configs/archival-fullnode.yaml` configuration
- Complete transaction history access
- Required for historical queries and indexers
- Higher storage requirements

## Documentation

For detailed deployment instructions, see the Movement documentation:

- [Full Node Deployment](https://docs.movementnetwork.xyz/general/nodes/full-node/run/deploy)
- [Archival Node Deployment](https://docs.movementnetwork.xyz/general/nodes/archival-node/run/deploy)

## Quick Start

```bash
# 1. Clone this repository
git clone https://github.com/movement-network/movement-networks.git
cd movement-networks

# 2. Choose your network (mainnet or testnet)
cd mainnet  # or testnet

# 3. (Optional) Restore existing data for faster sync
../l1_restore.sh mainnet ../data

# 4. Run with Docker
docker run --pull=always \
  --rm -p 8080:8080 -p 9101:9101 -p 6180:6180 \
  -v $(pwd):/opt/aptos \
  -v $(pwd)/../data:/opt/aptos/data \
  --workdir /opt/aptos \
  --name=movement-fullnode \
  ghcr.io/movementlabsxyz/aptos-node:f24a5bc \
  -f /opt/aptos/configs/fullnode.yaml
```
