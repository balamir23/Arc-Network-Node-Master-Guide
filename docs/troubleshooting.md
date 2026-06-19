# Troubleshooting

## RPC Returns 0x0

Possible causes:

- Snapshot missing
- Snapshot corrupted
- Consensus Layer not running
- Execution Layer started after Consensus Layer

---

## IPC Errors

Verify:

```bash
ls /run/arc
```

Expected:

- reth.ipc
- auth.ipc

---

## Check Logs

```bash
journalctl -u arc-execution -f
journalctl -u arc-consensus -f
```
