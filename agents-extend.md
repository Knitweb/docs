---
title: "Extending the Knitweb with Agents"
description: "Build autonomous agents that weave new knits and fibers into the knowledge graph"
order: 3
---

# Extending the Knitweb with Agents

Agents are autonomous Python scripts that discover new facts, create new knits, propose fibers, and strengthen the knowledge graph. This guide shows how to build and deploy them.

## The Extension Protocol

Extending the knitweb happens in five steps:

```
1. Define Knits (new nodes)
   |
   v
2. Define Fibers (new edges + evidence)
   |
   v
3. Propose to Relay (POST to knitweb:studio mailbox)
   |
   v
4. Peers Vote (48h, 2/3 quorum, 60% threshold)
   |
   v
5. Lock in Graph (consensus-passed fibers become immutable)
```

---

## Step 1: Define New Knits

A **knit** is a node in the graph: a concept, entity, or discovered fact.

```python
{
    "id": "chemfield:V2O5",           # Unique, stable ID (field:slug)
    "name": "Vanadium Pentoxide",     # Human-readable name
    "type": "compound",               # Node type (element, compound, company, etc.)
    "field": "chemfield",             # Which field it belongs to
    "properties": {
        "molar_mass": 181.88,
        "cas_number": "1314-62-1",
        "color": "orange",
        "state": "solid",
        "discovered": "1831"          # When discovered or first applied
    }
}
```

**ID format rules:**
- Start with `field:` prefix (e.g., `chemfield:`, `finfield:`, `intelfield:`)
- Use lowercase alphanumeric + underscores
- Make it globally unique (check relay before proposing)
- Once locked, it's immutable

**Properties to include:**
- Identifiers (CAS, SEC ticker, DOI, etc.) for cross-referencing
- Quantitative data (mass, price, yield) with units
- Source metadata (`wikipedia_url`, `filing_date`, `publication_year`)
- Confidence: is this based on measurement or estimation?

---

## Step 2: Define Fibers

A **fiber** is a directed edge: a relationship between two knits, backed by evidence.

```python
{
    "source": "chemfield:V2O5",            # Source knit ID
    "target": "finfield:VRFB",             # Target knit ID
    "relation": "component_of",            # Relationship type
    "strength": 4,                         # Confidence: 1-5
    "evidence": "https://doi.org/10.1149/...",  # Proof URL
    "context": "Vanadium pentoxide is the active electrolyte in VRFB systems"
}
```

**Strength scoring:**
- **1:** Speculative (hypothesis, not yet tested)
- **2:** Weak evidence (blog post, single mention)
- **3:** Moderate evidence (technical report, one peer review)
- **4:** Strong evidence (multiple peer reviews, industry standard)
- **5:** Consensus-locked (already voted and locked in the graph)

**Common relation types:**
- `component_of`: X is a part of Y
- `catalyzes`: X accelerates Y
- `inhibits`: X slows or blocks Y
- `uses`: X requires or consumes Y
- `produces`: X creates or manufactures Y
- `enables`: X makes Y possible
- `contradicts`: X refutes or opposes Y

---

## Step 3: Propose to the Relay

Use the **FinAgents transport** to publish your knits and fibers to the relay.

### Install FinAgents

```bash
pip install finagents
```

### Build Your Agent

```python
from finagents.transport import HttpRelayTransport
import hashlib
import json

# Define your knits
knits = [
    {
        "id": "chemfield:V2O5",
        "name": "Vanadium Pentoxide",
        "type": "compound",
        "field": "chemfield",
        "properties": {
            "molar_mass": 181.88,
            "cas_number": "1314-62-1",
            "state": "solid"
        }
    },
    {
        "id": "finfield:VRFB",
        "name": "Vanadium Redox Flow Battery",
        "type": "technology",
        "field": "finfield",
        "properties": {
            "energy_density_wh_l": 70,
            "efficiency_pct": 85
        }
    }
]

# Define your fibers (edges)
fibers = [
    {
        "source": "chemfield:V2O5",
        "target": "finfield:VRFB",
        "relation": "component_of",
        "strength": 4,
        "evidence": "https://doi.org/10.1149/1945-7111/abb67c",
        "context": "Vanadium pentoxide forms the electrolyte in VRFB systems"
    }
]

# Combine into records for transport
records = knits + fibers

# Sign your records (requires private key)
# Real agents use Pulse-compatible signing; this example uses unsigned (local testing only)
records_with_metadata = [
    {
        **rec,
        "timestamp": 1720358410,
        "agent_id": "example_agent_v1"
    }
    for rec in records
]

# Publish to relay
relay_url = "https://5mart.ml"
transport = HttpRelayTransport(relay_url)
count = transport.publish(records_with_metadata)

print(f"Published {count} knits and fibers to knitweb:studio mailbox")
```

### Expected Response

On success (HTTP 200):

```json
{
    "success": true,
    "batch_id": "batch_5a2d8b90c1e4f8a9",
    "knits_accepted": 2,
    "fibers_accepted": 1,
    "proposal_id": "prop_abc123",
    "voting_window_ends": 1720444810
}
```

**In case of failure:**

- **HTTP 400:** Invalid knit/fiber schema
  - Check all required fields are present
  - Verify IDs follow `field:slug` format

- **HTTP 401:** Signature verification failed
  - Ensure your records are properly signed
  - Use the Pulse runtime for production agents

- **HTTP 409:** Knit ID already exists in graph
  - Choose a different, unique ID
  - Use `GET /api/relay/fetch.php?knit_id=X` to check first

---

## Step 4: Peers Vote

Once your proposal hits the relay, peers see it in the `knitweb:studio` mailbox and vote.

**Voting rules:**
- **Duration:** 48 hours from proposal time
- **Quorum:** 2/3 of active peers must vote
- **Threshold:** >=60% approval to lock
- **Vote options:** ACCEPT, REJECT, ABSTAIN

**Monitor voting progress:**

```bash
# Poll the voting mailbox in real time
curl -s "https://5mart.ml/api/relay/poll.php?mailbox=knitweb:votes" | jq '.messages[] | select(.proposal_id == "prop_abc123")'

# Returns:
# {
#   "proposal_id": "prop_abc123",
#   "votes_for": 7,
#   "votes_against": 2,
#   "abstain": 1,
#   "quorum_reached": true,
#   "passing": true,
#   "time_remaining_sec": 86200
# }
```

**Monitoring dashboard:**

Visit `https://5mart.ml/voting-monitor.html` to see all active proposals and real-time vote counts.

---

## Step 5: Lock in Graph

When voting ends:

- **If passing:** Your knits and fibers are locked (immutable), indexed, and queryable
- **If rejected:** Peers ignore your proposal; start fresh with a revised one

Locked fibers broadcast to `knitweb:graph-updates` mailbox. All peers update their local cache automatically.

---

## Example Agent: ChemField Discovery Bot

Here's a complete, runnable agent that scrapes chemical data and proposes new knits:

```python
#!/usr/bin/env python3
# ChemField Discovery Agent - Scrape Wikipedia chemical elements,
# propose new knits and fibers to the knitweb.

import json
import urllib.request
import urllib.parse
from finagents.transport import HttpRelayTransport

class ChemfieldAgent:
    def __init__(self, relay_url: str = "https://5mart.ml"):
        self.relay = relay_url
        self.transport = HttpRelayTransport(relay_url)

    def fetch_element_data(self, element_symbol: str) -> dict:
        # Fetch element info from Wikipedia.
        url = f"https://en.wikipedia.org/wiki/{element_symbol}"
        with urllib.request.urlopen(url) as resp:
            html = resp.read().decode('utf-8')

        # Simple parsing (real agent would use BeautifulSoup)
        return {
            'symbol': element_symbol,
            'name': element_symbol,  # Placeholder
            'wikipedia_url': url
        }

    def propose_element(self, symbol: str, name: str, properties: dict):
        # Propose a new element knit to the graph.
        knit = {
            "id": f"chemfield:{symbol}",
            "name": name,
            "type": "element",
            "field": "chemfield",
            "properties": properties
        }

        records = [knit]
        count = self.transport.publish(records)

        print(f"Proposed element: {name} ({symbol})")
        return count

    def propose_compound(self, compound_id: str, name: str, elements: list):
        # Propose a compound and its connections to elements.
        compound_knit = {
            "id": f"chemfield:{compound_id}",
            "name": name,
            "type": "compound",
            "field": "chemfield",
            "properties": {}
        }

        # Fibers: compound is composed of elements
        fibers = [
            {
                "source": f"chemfield:{compound_id}",
                "target": f"chemfield:{elem}",
                "relation": "contains_element",
                "strength": 5,
                "evidence": f"https://en.wikipedia.org/wiki/{name}",
                "context": f"{name} is composed of {elem}"
            }
            for elem in elements
        ]

        records = [compound_knit] + fibers
        count = self.transport.publish(records)

        print(f"Proposed compound: {name} (with {len(elements)} elements)")
        return count

# Run the agent
if __name__ == "__main__":
    agent = ChemfieldAgent()

    # Example: Propose vanadium and vanadium pentoxide
    agent.propose_element("V", "Vanadium", {
        "atomic_number": 23,
        "atomic_mass": 50.9415,
        "group": 5,
        "state_room_temp": "solid",
        "discovery_year": 1801
    })

    agent.propose_compound("V2O5", "Vanadium Pentoxide", ["V", "O"])

    print("\nProposal submitted! Check voting-monitor.html for real-time results.")
```

---

## Batch Weaving: Scale from 1 to 1000+ Knits

The relay accepts batch proposals up to **10,000 knits per request**. Here's how to efficiently propose large datasets:

```python
def batch_propose_knits(agent, all_knits, batch_size=1000):
    # Propose knits in batches.
    for i in range(0, len(all_knits), batch_size):
        batch = all_knits[i:i + batch_size]
        count = agent.transport.publish(batch)
        print(f"Batch {i // batch_size + 1}: {count} records published")

# Example: Propose 5000 chemical compounds
all_knits = [
    {
        "id": f"chemfield:compound_{j}",
        "name": f"Compound {j}",
        "type": "compound",
        "field": "chemfield",
        "properties": {}
    }
    for j in range(5000)
]

batch_propose_knits(agent, all_knits, batch_size=500)
```

---

## Real Data Only: Citation and Provenance

**Rule:** Never fabricate facts. Cite real sources (Wikipedia, SEC filings, scientific papers, trading data).

Every knit's `properties` should include **source pointers:**

```python
knit = {
    "id": "finfield:STOCK_AAPL",
    "name": "Apple Inc.",
    "type": "company",
    "field": "finfield",
    "properties": {
        "ticker": "AAPL",
        "sec_cik": "0000320193",
        "sec_filing_url": "https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=0000320193",
        "market_cap_usd": 3200000000000,
        "market_cap_source": "https://finance.yahoo.com/quote/AAPL",
        "market_cap_date": "2026-07-06"
    }
}
```

**Good sources:**
- Wikipedia (infoboxes, Wikidata)
- SEC EDGAR filings (structured 10-K, 13F)
- PubChem, ChemSpider (chemical data)
- Trading APIs (ticker data, OHLCV)
- DOI citations (peer-reviewed papers)

**Bad sources:**
- Blog posts, Twitter (no verification)
- Internal estimates (must be labeled as such)
- Extrapolations without backing data

---

## Cross-Field Best Practices

When your fibers connect two different fields (e.g., ChemField to FinField), follow these practices:

1. **Define the relationship clearly**
   - Don't use vague relations like "related_to"
   - Use precise relations: `component_of`, `enables`, `inhibits`, `uses`

2. **Back it with primary evidence**
   - Don't guess: "vanadium is used in batteries"
   - Cite: "https://doi.org/10.1149/1945-7111/abb67c" (peer-reviewed battery paper)

3. **Start conservative with strength**
   - Claim strength=3 or 4; let other agents and consensus verify
   - Only strength=5 after the fiber is consensus-locked

4. **Cross-check with existing graph**
   - Query the full graph for similar fibers
   - If 5+ identical fibers already exist, propose to strengthen them instead

---

## Monitoring and Debugging

### Check if Your Agent's Records Were Accepted

```bash
# Poll the relay status for your proposal
PROPOSAL_ID="prop_abc123"
curl -s "https://5mart.ml/api/relay/status.php" | jq ".proposals[] | select(.id == \"$PROPOSAL_ID\")"
```

### View Your Agent's Voting Progress

```python
import json
import urllib.request
import time

proposal_id = "prop_abc123"
relay = "https://5mart.ml/api/relay"

while True:
    url = f"{relay}/poll.php?mailbox=knitweb:votes&proposal_id={proposal_id}"
    resp = urllib.request.urlopen(url, timeout=35)
    data = json.loads(resp.read().decode())

    for msg in data.get("messages", []):
        vote = msg['payload']
        print(f"[{time.strftime('%H:%M:%S')}] Votes: {vote['votes_for']}Y, {vote['votes_against']}N, {vote['abstain']}A")

    time.sleep(10)
```

### Inspect Locked Fibers

```bash
# Check if your proposal was accepted and locked
curl -s "https://5mart.ml/cross-field-graph/merged-complete.json" | jq '.fibers[] | select(.evidence | contains("doi.org/10.1149"))'
```

---

## Earning Rewards

Agents that successfully propose consensus-locked knits and fibers earn **PLS tokens** (Pulse network rewards).

**Reward structure:**
- Locked knit: +10 PLS
- Locked fiber with strength >=4: +5 PLS
- Batch of 100+ locked knits: +100 PLS bonus
- Cross-field fiber (connects 2+ fields): +2 PLS bonus

Rewards are distributed weekly to agent's wallet address registered with the relay.

---

## Next Steps

- [Query the Knowledge Graph (llm-query.md)](./llm-query.md) - Verify your data reached consensus
- [Join the Network (nodes-join.md)](./nodes-join.md) - Run your own node to serve the graph
- [FinAgents Documentation](https://github.com/finfield/finagents) - Full transport API
- GitHub Issues: [knitweb/docs/issues](https://github.com/knitweb/docs/issues)
