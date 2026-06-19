# Arc Network Node Master Guide

A complete beginner-friendly guide for running an Arc Network node.

This guide explains:

- Arc node architecture
- Snapshot installation
- Execution Layer (EL)
- Consensus Layer (CL)
- Systemd services
- Monitoring
- Troubleshooting

---

# What Is Arc?

Arc is an open, EVM-compatible Layer 1 blockchain.

Running your own node allows you to:

- Verify blocks independently
- Execute transactions locally
- Access your own RPC endpoint
- Remove dependence on third-party RPC providers

---

# Node Architecture

Arc consists of two components:

## Execution Layer (EL)

Responsibilities:

- Executes EVM transactions
- Maintains blockchain state
- Provides Ethereum JSON-RPC

## Consensus Layer (CL)

Responsibilities:

- Downloads finalized blocks
- Verifies validator signatures
- Passes blocks to the Execution Layer

Always start the Execution Layer first.

---

# Recommended Hardware

| Component | Recommended |
|------------|------------|
| CPU | High Clock Speed |
| RAM | 64GB+ |
| Storage | 1TB+ NVMe SSD |
| Network | Stable 24 Mbps+ |

---

# Installation

## Create Environment Variables

```bash
cat << "EOF" > ~/.arc_env
ARC_HOME="${ARC_HOME:-$HOME/.arc}"
ARC_BIN_DIR="${ARC_BIN_DIR:-$ARC_HOME/bin}"
ARC_RUN="/run/arc"

ARC_EXECUTION=$ARC_HOME/execution
ARC_CONSENSUS=$ARC_HOME/consensus

export ARC_HOME ARC_BIN_DIR ARC_RUN ARC_EXECUTION ARC_CONSENSUS
export PATH="$ARC_BIN_DIR:$PATH"
EOF

source ~/.arc_env
```

## Create Directories

```bash
mkdir -p "$ARC_EXECUTION" "$ARC_CONSENSUS" "$ARC_BIN_DIR"
sudo install -d -o $USER "$ARC_RUN"
```

## Verify Installation

```bash
arc-snapshots --version
arc-node-execution --version
arc-node-consensus --version
```

---

# Download Snapshot

```bash
arc-snapshots download \
  --chain=arc-testnet \
  --execution-path "$ARC_EXECUTION" \
  --consensus-path "$ARC_CONSENSUS"
```

---

# Initialize Consensus Layer

```bash
arc-node-consensus init --home $ARC_CONSENSUS
```

---

# Start Execution Layer

```bash
arc-node-execution node \
  --chain arc-testnet \
  --datadir $ARC_EXECUTION \
  --full \
  --ipcpath $ARC_RUN/reth.ipc \
  --auth-ipc \
  --auth-ipc.path $ARC_RUN/auth.ipc \
  --http \
  --http.addr 127.0.0.1 \
  --http.port 8545 \
  --http.api eth,net,web3,txpool,trace,debug \
  --rpc.forwarder https://rpc.quicknode.testnet.arc.network/ \
  --metrics 127.0.0.1:9001 \
  --disable-discovery \
  --enable-arc-rpc
```

---

# Start Consensus Layer

Open a second terminal:

```bash
arc-node-consensus start \
  --home $ARC_CONSENSUS \
  --full \
  --eth-socket $ARC_RUN/reth.ipc \
  --execution-socket $ARC_RUN/auth.ipc \
  --rpc.addr 127.0.0.1:31000 \
  --follow \
  --follow.endpoint https://rpc.drpc.testnet.arc.network,wss=rpc.drpc.testnet.arc.network \
  --follow.endpoint https://rpc.quicknode.testnet.arc.network,wss=rpc.quicknode.testnet.arc.network \
  --follow.endpoint https://rpc.blockdaemon.testnet.arc.network,wss=rpc.blockdaemon.testnet.arc.network/websocket \
  --execution-persistence-backpressure \
  --execution-persistence-backpressure-threshold=50 \
  --metrics 127.0.0.1:29000
```

---

# Verify Sync

```bash
curl -s -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

If the block number increases over time, the node is syncing correctly.

---

# Monitoring

Execution Metrics:

```text
localhost:9001/metrics
```

Consensus Metrics:

```text
localhost:29000/metrics
```

Logs:

```bash
journalctl -u arc-execution -f
journalctl -u arc-consensus -f
```

---

# License

MIT
