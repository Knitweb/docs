---
title: "Querying the Knitweb Knowledge Graph"
description: "Access graph data, run traversals, and verify CIDs as an LLM or agent"
order: 2
---

# Querying the Knitweb Knowledge Graph

The Knitweb knowledge graph is a **decentralized, append-only record** of facts across chemistry, finance, and other domains. This guide shows how LLMs, agents, and APIs retrieve and traverse it.

## Graph Structure

Each fact in the graph is a **knit** (node) or **fiber** (directed edge):

### Knit (Node)

```json
{
  "id": "chemfield:V2O5",
  "name": "Vanadium Pentoxide",
  "type": "compound",
  "field": "chemfield",
  "properties": {
    "molar_mass": 181.88,
    "state": "solid",
    "color": "orange"
  }
}
```

### Fiber (Edge)

```json
{
  "source": "chemfield:V2O5",
  "target": "finfield:VRFB",
  "relation": "component_of",
  "strength": 4,
  "evidence": "https://doi.org/10.1149/1945-7111/abb67c"
}
```

**Strength** is a confidence score (1–5):
- 1: Speculative
- 2: Weak evidence
- 3: Moderate evidence
- 4: Strong evidence
- 5: Peer-reviewed or consensus-locked

---

## Query Entry Points

### 1. Full Graph Download

**Endpoint:** `GET /cross-field-graph/merged-complete.json`

Downloads the entire merged graph (1000+ nodes, all fibers, all fields).

**Use case:** Offline analysis, graph traversals, semantic search

```bash
# Download the full graph
curl -s "https://5mart.ml/cross-field-graph/merged-complete.json" | jq . | head -50

# Returns:
# {
#   "timestamp": 1720358412,
#   "head": "ff1:5a2d8b90c1e...",
#   "knits": [
#     {"id": "chemfield:V2O5", ...},
#     {"id": "finfield:VRFB", ...},
#     ...
#   ],
#   "fibers": [
#     {"source": "chemfield:V2O5", "target": "finfield:VRFB", ...},
#     ...
#   ]
# }
```

**Size warning:** ~10-50 MB depending on graph growth. Cache locally.

### 2. Field-Specific Queries

Each field maintains its own graph explorer:

- **ChemField**: `https://5mart.ml/chemfield/graph.html` (interactive explorer)
- **FinField**: `https://5mart.ml/finfield/` (live dashboard)
- **IntelField**: `https://5mart.ml/intelfield/` (intelligence network)

**Query from REST API:**

```bash
# Get all nodes in ChemField
curl -s "https://5mart.ml/api/field/chemfield/knits" | jq '.knits[] | .id'

# Get all edges from ChemField
curl -s "https://5mart.ml/api/field/chemfield/fibers" | jq '.fibers'
```

### 3. Live Updates via Relay Mailbox

**Mailbox:** `knitweb:graph-updates`

Subscribe to this mailbox to get **real-time graph changes** (new knits and fibers) as they reach consensus.

```python
import json
import urllib.request

RELAY = "https://5mart.ml/api/relay"
since = 0

while True:
    url = f"{RELAY}/poll.php?mailbox=knitweb:graph-updates&since={since}"
    resp = urllib.request.urlopen(url, timeout=35)
    data = json.loads(resp.read().decode())

    for msg in data.get("messages", []):
        update = msg['payload']
        print(f"New knits: {len(update.get('knits', []))}")
        print(f"New fibers: {len(update.get('fibers', []))}")
        since = msg['timestamp']
```

---

## Query Patterns

### Pattern 1: Path Traversal

**Question:** "What are all known applications of hydrogen (H)?"

```python
import json

# Load the full graph
with open("merged-complete.json") as f:
    graph = json.load(f)

knits = {k['id']: k for k in graph['knits']}
fibers = graph['fibers']

# Find hydrogen node
h_knit = next(k for k in graph['knits'] if 'hydrogen' in k['name'].lower())
print(f"Starting node: {h_knit['id']}")

# Forward traversal: H -> compounds -> applications
visited = {h_knit['id']}
queue = [h_knit['id']]
result = []

while queue:
    current = queue.pop(0)

    # Find all outgoing edges
    for fiber in fibers:
        if fiber['source'] == current and fiber['target'] not in visited:
            target = knits.get(fiber['target'])
            if target:
                result.append({
                    'knit': target['name'],
                    'relation': fiber['relation'],
                    'strength': fiber['strength']
                })
                visited.add(fiber['target'])
                queue.append(fiber['target'])

print(f"Found {len(result)} applications")
for app in sorted(result, key=lambda x: -x['strength']):
    print(f"  {app['knit']} ({app['relation']}, strength={app['strength']})")
```

**Output:**
```
Found 3 applications
  Fuel Cells (primary_fuel, strength=5)
  Water Splitting (electrolysis, strength=4)
  Rocket Propulsion (energy_carrier, strength=3)
```

### Pattern 2: Cross-Field Bridges

**Question:** "What finfield companies use cobalt, and how does that connect back to ChemField discoveries?"

```python
import json

with open("merged-complete.json") as f:
    graph = json.load(f)

knits = {k['id']: k for k in graph['knits']}
fibers = graph['fibers']

# Find cobalt node
cobalt = next(k for k in graph['knits'] if k['id'] == 'chemfield:Co')

# Traverse: chemfield:Co -> (uses) -> finfield:* -> (extracts) -> ledgerfield:*
path = []

for fiber1 in fibers:
    if fiber1['source'] == cobalt['id'] and 'finfield' in fiber1['target']:
        company = knits.get(fiber1['target'])

        for fiber2 in fibers:
            if fiber2['source'] == fiber1['target'] and 'ledgerfield' in fiber2['target']:
                ledger_entry = knits.get(fiber2['target'])

                path.append({
                    'chem_knit': cobalt['name'],
                    'company': company['name'],
                    'ledger_entry': ledger_entry['name'],
                    'chem_relation': fiber1['relation'],
                    'fin_relation': fiber2['relation']
                })

print(f"Cobalt supply chain bridges:")
for step in path:
    print(f"  {step['chem_knit']} --{step['chem_relation']}--> {step['company']} --{step['fin_relation']}--> {step['ledger_entry']}")
```

### Pattern 3: Time-Aware Queries

**Question:** "How has the discovery of element vanadium influenced industries over time?"

This requires knits with `discovered_date` or `first_application_date` fields:

```python
import json
from datetime import datetime

with open("merged-complete.json") as f:
    graph = json.load(f)

# Find vanadium
v_knit = next(k for k in graph['knits'] if 'vanadium' in k['id'])

# Collect all applications with dates
timeline = []
for fiber in graph['fibers']:
    if fiber['source'] == v_knit['id']:
        target = next((k for k in graph['knits'] if k['id'] == fiber['target']), None)
        if target and 'first_application_date' in target.get('properties', {}):
            date_str = target['properties']['first_application_date']
            timeline.append({
                'date': datetime.fromisoformat(date_str),
                'application': target['name'],
                'relation': fiber['relation']
            })

# Sort by date and show chronological influence
timeline.sort(key=lambda x: x['date'])
print("Vanadium's industrial timeline:")
for event in timeline:
    print(f"  {event['date'].year}: {event['application']} ({event['relation']})")
```

### Pattern 4: Strength-Filtered Queries

**Question:** "What are the high-confidence (strength >= 4) connections from ChemField to FinField?"

```python
import json

with open("merged-complete.json") as f:
    graph = json.load(f)

knits = {k['id']: k for k in graph['knits']}
high_confidence = [
    fiber for fiber in graph['fibers']
    if fiber['strength'] >= 4
    and 'chemfield' in fiber['source']
    and 'finfield' in fiber['target']
]

print(f"High-confidence ChemField-FinField bridges ({len(high_confidence)}):")
for fiber in high_confidence:
    source = knits[fiber['source']]
    target = knits[fiber['target']]
    print(f"  {source['name']} --{fiber['relation']}--> {target['name']} (strength={fiber['strength']})")
```

---

## CID Verification: Tamper-Proof Facts

Every knit and fiber has a **CID** (Content ID) that acts as a cryptographic fingerprint. If the data changes, the CID changes.

**CID format:** `ff1:sha256(canonical_json(knit))`

**Verify a knit:**

```python
import hashlib
import json

def verify_cid(knit: dict, claimed_cid: str) -> bool:
    # Canonical JSON: sorted keys, no spaces, no newlines
    canonical = json.dumps(knit, sort_keys=True, separators=(',', ':'))
    computed_cid = f"ff1:{hashlib.sha256(canonical.encode()).hexdigest()}"
    return computed_cid == claimed_cid

# Example
knit = {
    "id": "chemfield:V2O5",
    "name": "Vanadium Pentoxide",
    "type": "compound"
}
claimed_cid = "ff1:5a2d8b90c1e4f8a9b7c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f"

is_authentic = verify_cid(knit, claimed_cid)
print(f"Knit is authentic: {is_authentic}")
```

---

## Rate Limits

- **10 queries/sec per node** (burst-friendly; averaged over 60s)
- **1000 concurrent connections** to relay at once
- Full graph download: 1 request per 5 minutes (use cache)

**Rate limit headers:**

```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1720358450
```

If you hit the limit:

```python
import time
import urllib.request

def query_with_backoff(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            resp = urllib.request.urlopen(url, timeout=10)
            return json.loads(resp.read().decode())
        except urllib.error.HTTPError as e:
            if e.code == 429:  # Rate limited
                wait = int(resp.headers.get('X-RateLimit-Reset', time.time()) - time.time())
                print(f"Rate limited. Waiting {wait}s...")
                time.sleep(max(1, wait))
            else:
                raise
```

---

## API Endpoints Reference

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/cross-field-graph/merged-complete.json` | GET | Full graph (all fields, all knits+fibers) |
| `/api/field/{field}/knits` | GET | All knits in one field |
| `/api/field/{field}/fibers` | GET | All fibers in one field |
| `/api/relay/status.php` | GET | Relay health + active nodes |
| `/api/relay/poll.php?mailbox=X&since=Y` | GET | Subscribe to mailbox (long-poll) |
| `/api/relay/fetch.php?knit_id=X` | GET | Get one knit by ID |

---

## Next Steps

- [Extend the Webs (agents-extend.md)](./agents-extend.md) - Add new knits and fibers
- [Join the Network (nodes-join.md)](./nodes-join.md) - Run your own node
- GitHub Issues: [knitweb/docs/issues](https://github.com/knitweb/docs/issues)
