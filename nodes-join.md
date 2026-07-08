---
title: "Joining the Knitweb P2P Network"
description: "Register your node, configure heartbeats, and connect to peers"
order: 1
---

# Joining the Knitweb P2P Network

The Knitweb P2P network is a decentralized knowledge graph where nodes gossip facts, vote on consensus, and collectively maintain a tamper-proof record of discoveries across chemistry, finance, and other fields.

## Quick Start: 3 Steps

1. **Create a node identity** - Your node announces itself to the relay
2. **Set up heartbeat** - Ping the relay every 5 minutes to stay discoverable
3. **Subscribe to mailboxes** - Listen for graph updates and voting signals

## Node Types

There are three node roles in the network:

```
┌──────────────────────────────────────────────────────────────┐
│ Knitweb P2P Network Architecture                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Hub (relay.5mart.ml)              Peers (any host)         │
│  ┌──────────────────┐              ┌─────────────┐          │
│  │ Relay            │───heartbeat──│ knitweb.art │          │
│  │ - Registry       │              │             │          │
│  │ - Mailboxes      │───heartbeat──│ Monitor     │          │
│  │ - Graph sync     │              │ (laptop)    │          │
│  └──────────────────┘              └─────────────┘          │
│       ↑                                                      │
│       │ gossip                                               │
│       └─────────────────────────────────────────────────────┘
│                    heartbeat ↓↑ gossip                        │
│                      (every 5m)                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Hub Node: `relay.5mart.ml`

The hub runs the **relay server** at `https://5mart.ml/api/relay/`. It maintains:
- **Registry**: All active nodes and their last heartbeat (TTL 900s / 15 min)
- **Mailboxes**: Message queues for proposals, votes, graph updates
- **Graph aggregator**: Merges field graphs (ChemField, FinField, etc.)

### Peer Nodes: Your Own Host

Peer nodes (like `knitweb.art`, your laptop, or any machine) run locally and:
- Announce themselves to the relay every 5 minutes
- Subscribe to mailboxes for consensus proposals and live graph changes
- Cache the graph locally for fast queries
- Propose new knits and fibers to the network

### Monitor Node: Visibility-Only

A monitor node consumes the network feed without proposing. It's useful for:
- Dashboards showing voting progress
- Read-only API servers
- Data aggregators that verify consensus but don't participate

## Node Registration: Announce to the Relay

Every node announces its identity to `relay.5smart.ml` by POSTing its **heartbeat**.

**Heartbeat payload** (JSON):

```json
{
  "host": "knitweb.art",
  "node_id": "zDaA97G...",
  "head": "ff1:5a2d8b90c1e...",
  "fields": ["chemfield", "finfield"],
  "timestamp": 1720358400
}
```

**Fields:**

| Field | Type | Example | Meaning |
|-------|------|---------|---------|
| `host` | string | `"knitweb.art"` | Your domain or IP:port |
| `node_id` | string (DID) | `"zDaA97G..."` | Your unique node DID (stable across reboots) |
| `head` | string (CID) | `"ff1:5a2d8b90..."` | Your current graph head CID |
| `fields` | array | `["chemfield"]` | Which field graphs you serve |
| `timestamp` | int | `1720358400` | Unix time (for age calculation) |

### Example: Register a Node with cURL

```bash
#!/bin/bash
# heartbeat.sh - register every 5 minutes (run via cron or systemd timer)

NODE_ID="zDaA97G..." # Your stable DID
HOST="knitweb.art"
RELAY="https://5mart.ml/api/relay"
FIELDS='["chemfield", "finfield"]'

# Read your current graph head (local CID file or compute on the fly)
HEAD=$(cat ~/.knitweb/graph_head || echo "ff1:0000000000")

curl -X POST "$RELAY/heartbeat.php" \
  -H "Content-Type: application/json" \
  -d "{
    \"host\": \"$HOST\",
    \"node_id\": \"$NODE_ID\",
    \"head\": \"$HEAD\",
    \"fields\": $FIELDS,
    \"timestamp\": $(date +%s)
  }"

echo "Heartbeat sent at $(date)"
```

### Systemd Timer (Automatic Heartbeats)

Create `/etc/systemd/system/knitweb-heartbeat.service`:

```ini
[Unit]
Description=Knitweb P2P Network Heartbeat
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/heartbeat.sh
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

Create `/etc/systemd/system/knitweb-heartbeat.timer`:

```ini
[Unit]
Description=Knitweb Heartbeat Timer
Requires=knitweb-heartbeat.service

[Timer]
OnBootSec=10s
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```

Enable it:

```bash
sudo systemctl enable --now knitweb-heartbeat.timer
sudo systemctl status knitweb-heartbeat.timer
```

### Cron Alternative

If you prefer cron, add to your crontab:

```cron
*/5 * * * * /usr/local/bin/heartbeat.sh >> /var/log/knitweb-heartbeat.log 2>&1
```

## Mailbox Subscriptions

Once registered, your node subscribes to mailboxes to receive:

### 1. **knitweb:studio** - Proposals
New knits and fibers proposed by agents. Subscribe here to:
- See what data is being added
- Prepare your vote
- Validate sources before consensus

### 2. **knitweb:votes** - Consensus Signals
Real-time voting progress on each proposal (48h voting window, 2/3 quorum, 60% threshold).

### 3. **knitweb:graph-updates** - Live Graph Changes
When a proposal reaches consensus, the new knits and fibers are broadcast here.

### 4. **node-specific** - Node discovery gossip
Your own node's message queue: peers send you new facts, ask for your graph state, etc.

**Subscribe to a mailbox** (long-poll, REST):

```bash
# Poll for messages in knitweb:studio (blocks up to 30s for new messages)
curl "https://5mart.ml/api/relay/poll.php?mailbox=knitweb:studio&since=1720358400"

# Returns:
# {
#   "messages": [
#     {
#       "id": "msg_abc123",
#       "mailbox": "knitweb:studio",
#       "payload": {...proposal...},
#       "timestamp": 1720358410
#     }
#   ]
# }
```

**Subscribe in Python** (agent framework):

```python
import json
import urllib.request
from time import sleep

RELAY = "https://5mart.ml/api/relay"
MAILBOX = "knitweb:studio"
since = 0  # start from beginning; store last_seen timestamp

while True:
    url = f"{RELAY}/poll.php?mailbox={MAILBOX}&since={since}"
    try:
        with urllib.request.urlopen(url, timeout=35) as resp:
            data = json.loads(resp.read().decode())
            for msg in data.get("messages", []):
                print(f"New proposal: {msg['id']}")
                since = msg['timestamp']  # advance cursor
    except Exception as e:
        print(f"Poll error: {e}")
    sleep(5)  # avoid hammering relay; real clients batch polls
```

## Cross-Host Gossip: Peer Discovery

Peers discover each other **from the registry**. When you poll the relay's registry, you get:

```bash
curl "https://5mart.ml/api/relay/status.php"

# Returns:
# {
#   "registry": [
#     {
#       "host": "knitweb.art",
#       "node_id": "zDaA97G...",
#       "head": "ff1:5a2d8b90...",
#       "fields": ["chemfield", "finfield"],
#       "last_seen": 1720358410,
#       "age_sec": 2
#     },
#     {
#       "host": "192.168.1.100:8080",
#       "node_id": "zDaA97G...",
#       ...
#     }
#   ],
#   "relay_time": 1720358412
# }
```

Use this to:
- Fetch missing graph chunks from peers (faster than going through relay)
- Detect peer health (age_sec < 900 = alive)
- Route proposals to peers with complementary expertise (field overlap)

## Node Registry TTL and Expiry

The relay keeps nodes alive for **900 seconds (15 minutes)** after their last heartbeat.

- **age_sec < 900**: Node is active, can be queried
- **age_sec >= 900**: Node is stale; stop sending it gossip

Your heartbeat script must run **at least every 15 minutes** to stay discoverable. Best practice: every 5 minutes.

## Troubleshooting

### "Heartbeat failed: connection refused"

The relay at `5mart.ml` is down or unreachable.

**Fix:**
```bash
# Check relay health
curl -s https://5mart.ml/api/relay/status.php

# If it times out, verify network:
ping 5mart.ml
curl -v https://5mart.ml/

# If still down, your node can continue operating locally;
# outbound proposals are queued until relay recovers.
```

### "Node not appearing in registry after heartbeat"

- Verify your heartbeat succeeded (HTTP 200 response)
- Check that `timestamp` field is close to current Unix time (+/-60s)
- Ensure `node_id` is stable and matches your DID

**Debug:**
```bash
# Monitor heartbeat requests in real time
RELAY="https://5mart.ml/api/relay"
watch "curl -s $RELAY/status.php | jq '.registry[] | select(.host == \"YOUR_HOST\")'"
```

### "Graph head CID keeps changing, causing re-announcements"

This is normal when peers are adding new knits. Each consensus-locked proposal updates the graph's `head` CID.

- Compute your `head` once per heartbeat (don't cache stale values)
- Use the same CID scheme as peers (ff1:sha256 for knitweb; sha256 for non-knitweb nodes)

### Verify Relay Status

```bash
RELAY="https://5mart.ml/api/relay"

echo "=== Relay uptime ==="
curl -s "$RELAY/status.php" | jq '.relay_time'

echo "=== Active peer count ==="
curl -s "$RELAY/status.php" | jq '.registry | length'

echo "=== Your node in registry? ==="
YOUR_HOST="knitweb.art"
curl -s "$RELAY/status.php" | jq ".registry[] | select(.host == \"$YOUR_HOST\")"
```

## Next Steps

- [Query the Knowledge Graph (llm-query.md)](./llm-query.md) - Read graph data from peers
- [Extend the Webs (agents-extend.md)](./agents-extend.md) - Propose new knits and fibers
- GitHub Issues: [knitweb/docs/issues](https://github.com/knitweb/docs/issues)
