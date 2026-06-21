# How the monitor separates chem-web knits & pulses from the rest

> **Use case:** in one unified knitweb, the relay and the bar carry *everything* —
> chemistry concept-knits, bond-knits, governance votes, agent pulses, presence
> heartbeats, and (eventually) records from other domain-knitwebs. The monitor's
> **chem-web** view must select only the chemistry knowledge graph and leave the
> rest out. This note explains the selection, layer by layer, and how the monitor
> implements it.

The golden rule: **selection is done on signed, canonical fields — never on
free-text guessing.** Every separator below is part of a signed record or a
relay envelope, so it cannot be spoofed and the same filter works on any node.

---

## The five selection layers (coarse → fine)

```
                       ┌─────────────────────────── one shared knitweb ──────────────────────────┐
 relay envelope:       │  topic = "knit"        topic = "vote"    topic = "presence"  topic = …   │
 record body (op):     │  op=knit | op=link     op=vote           ping                governance  │
 knitweb namespace:    │  chemistry (L5)        chemistry         —                   vbank, …     │
 canonical schema/CID: │  ChemistryNode|BondEdge                  —                   Ballot, …    │
 monitor facet:        │  /api/monitor/kg/*  ◄── the chem-web slice ──►   /api/monitor/{status,…} │
                       └──────────────────────────────────────────────────────────────────────────┘
```

### 1. Relay topic — the transport partition
The HTTP relay stores every message with a `topic` (`php/src/Relay.php`,
`signedPreimage(to, topic, body)`), and the sender **signs the topic** as part
of the preimage `"knitweb-relay:v1\n{to}\n{topic}\n{body}"`. So the topic is
tamper-proof.

- Chem knits are relayed on `topic = "knit"` (the bridge's `relay_send("knit", …)`).
- Votes/pulses, presence, and other domains use their own topics.
- **Select:** `GET /api/relay/fetch?topic=knit&since=<cursor>` returns only the
  chem-web stream; `topic=vote` or `topic=presence` are excluded by construction.

### 2. Record `op` / `kind` — the semantic partition
Inside the (signed) body, every record declares what it is:

| op / kind | meaning | chem-web? |
|---|---|---|
| `op:"knit"` / `kind:"concept"` | a term/concept claim (a node) | ✅ |
| `op:"link"` / `kind:"link"` (a `BondEdge`) | a bond/relation between concepts (an edge) | ✅ |
| `op:"vote"` (a **pulse**) on a chem `pid` | endorsement that may settle/weave a Fiber | ✅ (chem pulse) |
| governance `kind:"poll" / "ballot"` | a vBank vote | ❌ (governance) |
| `ping` / presence | a node heartbeat | ❌ |

**Select chem nodes+edges:** `kind ∈ {concept, link}`. **Select chem pulses:**
`op == "vote"` *whose target pid resolves to a chem knit* (see layer 4).

### 3. Knitweb namespace (L5) — the domain partition
chem-web is the **`chemistry` knitweb** — an L5 domain plugin
(`knitweb/pulse → src/knitweb/knitwebs/chemistry`). The fabric tags every woven
record with the knitweb that produced it, so a single query separates domains
even when they share the relay:

- **Select:** `knitweb == "chemistry"`.
- **Exclude:** `knitweb == "vbank"` (governance), or any other plugin.

This is the most durable separator: it survives topic/kind conventions changing,
because the namespace is part of the canonical record.

### 4. Canonical schema / CID — the cryptographic partition
A chem record is a `ChemistryNode` or a `BondEdge` whose canonical-CBOR shape is
**CID-stability-locked** (see pulse #210 / #215 / #216: composition is normalised
at construction and pinned to a golden CIDv1). That means:

- A chem record is **self-identifying** by its schema/field-set; you don't need
  to trust the topic — you can recompute the CID and check it parses as a
  `ChemistryNode`/`BondEdge`.
- Non-chem records (governance ballots, presence) fail that parse and are dropped
  from the chem-web view.

A chem **pulse** (vote) is linked to a chem node by the `pid` it targets; resolve
the `pid` to its knit, confirm the knit is a chem record, and the vote belongs to
the chem-web. Votes whose `pid` resolves to a non-chem knit are excluded.

### 5. The monitor `kg/*` facet — the rendered slice
The monitor already exposes a dedicated **knowledge-graph (`kg`) family** that
*is* the chem-web slice, separate from the node/economic facets:

| endpoint | chem-web view |
|---|---|
| `GET /api/monitor/kg/stats` | `{nodes, edges, concepts, languages}` — chem-web size |
| `GET /api/monitor/kg/hubs?n=` | top-`n` hub concepts |
| `GET /api/monitor/kg/tension` | taut / slack / **snapped** fibers — i.e. how **pulses (votes)** settled chem bonds |
| `GET /api/monitor/kg/subgraph?term=&depth=` | a focused chem subgraph for the viz |
| `GET /api/monitor/kg/concept?key=` | one concept's detail |

Everything under `kg/*` is the chem-web use case. The node/p2p health
(`/api/monitor/status`), the relay roster (`/api/relay/online`), and governance
live on **other** monitor facets, so they never bleed into the chem graph.

> Note: `kg/tension` is exactly where **pulses** show up *inside* the chem-web —
> a vote that reaches quorum weaves a Fiber (a "taut" bond); a contested one
> "snaps". So chem pulses are rendered as edge-state on the chem graph, not mixed
> into a generic vote feed.

---

## Practical selector cheat-sheet

| I want… | Selector |
|---|---|
| chem concept-knits (nodes) | `topic=knit` **&** `kind∈{concept}` **&** `knitweb=chemistry` |
| chem bond-knits (edges) | `kind=link` / parses as `BondEdge` |
| chem pulses (votes) | `op=vote` **&** `pid → chem knit` (or read `kg/tension`) |
| the whole chem-web for the viz | `GET /api/monitor/kg/subgraph` **or** the published `explorer-graph.json` snapshot |
| exclude governance | `knitweb≠vbank` **&** `kind∉{poll,ballot}` |
| exclude agent presence | `topic≠presence` **&** `op≠ping` |

The 3D explorer's **"Live chem-web"** view consumes exactly this slice: it fetches
`explorer-graph.json` (the published chem-KG snapshot) — concepts as nodes,
bonds/relations as edges — which is the layer-5 `kg` projection already filtered
to the chemistry knitweb.

---

## Why this is robust (defence-in-depth)

- **Four of the five layers are signed or canonical** (topic, op/kind in the
  signed body, knitweb namespace, CID schema) → a record can't lie about being
  chem-web, and a non-chem record can't sneak in.
- The layers are **redundant**: even if the `topic` convention drifts, the
  `knitweb` namespace + CID schema still identify chem records. Pick the coarsest
  filter that's cheap (topic) for the stream, and verify with the finest
  (CID/schema) before weaving.
- The monitor reads through the **`kg/*` facet only**, so the chem-web use case is
  a single, well-defined projection — governance, economics, presence and other
  knitwebs are *different facets of the same monitor*, never a filter you have to
  remember to apply.

---

*Related: pulse `#210/#215/#216` (chem-record CID stability), the molgang relay
(`php/src/Relay.php`, `Onboard.php`), and the 3D explorer (`monitor/knitweb-graph-3d.html`).*
