# Arc Network Node Installation Guide

This guide explains how to install and run an Arc Network node from scratch.

No advanced Linux knowledge is required.

---

# Step 1 — Verify System Requirements

Before starting, make sure your server meets the minimum requirements.

| Component | Recommended |
|------------|------------|
| CPU | High Clock Speed |
| RAM | 64GB+ |
| Storage | 1TB+ NVMe SSD |
| Network | Stable 24 Mbps+ |

---

# Step 2 — Create Arc Environment

Create the Arc environment file:

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
```

Load it:

```bash
source ~/.arc_env
```

---

# Step 3 — Create Required Directories

```bash
mkdir -p "$ARC_EXECUTION"
mkdir -p "$ARC_CONSENSUS"
mkdir -p "$ARC_BIN_DIR"

sudo install -d -o $USER "$ARC_RUN"
```

These folders will store:

- Execution Layer data
- Consensus Layer data
- Runtime sockets

---

# Step 4 — Verify Arc Installation

Check that all Arc binaries are available.

```bash
arc-snapshots --version

arc-node-execution --version

arc-node-consensus --version
```

Expected result:

Each command should return a version number.

---

# Step 5 — Download Node Snapshots

Arc currently requires snapshots for initial synchronization.

Run:

```bash
arc-snapshots download \
  --chain=arc-testnet \
  --execution-path "$ARC_EXECUTION" \
  --consensus-path "$ARC_CONSENSUS"
```

What this does:

- Downloads the latest blockchain state
- Extracts Execution Layer data
- Extracts Consensus Layer data

Depending on internet speed this may take several minutes.

---

# Step 6 — Initialize Consensus Layer

Run:

```bash
arc-node-consensus init \
--home $ARC_CONSENSUS
```

This generates the node identity key.

Only needs to be done once.

---

# Step 7 — Start Execution Layer

Start the Execution Layer first.

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

Wait until logs appear normally.

Do not close the terminal.

---

# Step 8 — Start Consensus Layer

Open a second terminal.

Run:

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

The node should now begin syncing.

---

# Step 9 — Verify Synchronization

Wait approximately 30 seconds.

Run:

```bash
curl -s -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

Example output:

```json
{
  "jsonrpc":"2.0",
  "id":1,
  "result":"0x123456"
}
```

The block number should continue increasing over time.

---

# Step 10 — Check Node Health

Check Execution Layer logs:

```bash
journalctl -u arc-execution -f
```

Check Consensus Layer logs:

```bash
journalctl -u arc-consensus -f
```

---

# Step 11 — Enable Automatic Startup

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable services:

```bash
sudo systemctl enable arc-execution
sudo systemctl enable arc-consensus
```

Start services:

```bash
sudo systemctl start arc-execution
sudo systemctl start arc-consensus
```

Verify:

```bash
sudo systemctl status arc-execution

sudo systemctl status arc-consensus
```

Both services should show:

```text
active (running)
```

---

# Installation Complete

Your Arc node is now:

- Downloading finalized blocks
- Verifying validator signatures
- Executing transactions locally
- Providing an Ethereum-compatible RPC endpoint

RPC Endpoint:

```text
http://localhost:8545
```

Congratulations, your Arc node is running successfully.
