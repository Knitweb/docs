# Knitweb - the repositories and how they relate

Knitweb is not one codebase - it is a small constellation of focused repositories
that compose into one system. Each repo does one thing and exposes a stable seam;
nothing reaches into another repo's internals. This is the "Begrippen" tab you see
at the top of the 3D graph: the nodes are these repositories and the edges are the
relationships below.

The guiding rule: **one substrate, many plugins.** Everything rides the `pulse`
engine; domains and tools attach at well-defined boundaries, never by patching each
other.

## The ten repositories

| Repo | Role | One line |
|---|---|---|
| **pulse** | the engine / substrate | the pure-Python P2P crypto web: content-addressed CBOR records (CIDv1), integer ledger (Braid/Knit/Fiber), PoUW-minted PLS, the Pulse heartbeat. Everything depends on this. |
| **lens** | the reasoning lobe | the LRM/interpret layer: deterministic retrieve + gated, provenance-cited distill over the web. Consumes pulse's read-only export boundary, never its internals. |
| **molgang** | a domain plugin (L5) | the chemistry knowledge bar game; rides pulse + hosts the HTTP relay/bar that other nodes join. |
| **vbank** | governance plugin (L5) | poll/ranked/liquid voting + treasury; governs emission policy and which plugins are admitted. |
| **monitor** | observability | health, the knowledge-graph `kg/*` facets, and the 3D explorer. Reads pulse's `web_snapshot` (read-only). |
| **virtualpc** | multi-agent coordination | the agent runtime: registry, shared memory, verifiable compute - where agents coordinate on the shared fabric. |
| **bt** | off-grid transport | Bluetooth-LE air-mesh carrier, so the web survives with no internet. |
| **docs** | documentation | central, per-repo docs + whitepaper + plans + how-tos (this repo). |
| **knitweb.github.io** | public face | the landing page + whitepaper at knitweb.github.io. |
| **news** | announcements | project news / changelog surface. |

## How they relate (the edges)

Read it as layers, substrate at the bottom:

```
              news        knitweb.github.io          <- public surface
                \              /
   docs  ----  describes everything  ----  monitor   <- meta: document + observe
                       |                       |
        lens (reasoning)        vbank (governance)    <- brain + rules ON the web
                 \                    /
                  \                  /
        molgang (domain) --- rides --- pulse           <- a plugin on the engine
                                |
            virtualpc (agents) -+- bt (transport)      <- run agents / carry off-grid
                                |
                       pulse  = THE engine             <- the substrate everything rides
```

- **pulse is the substrate.** `lens`, `molgang`, `vbank`, `monitor`, `virtualpc`
  and `bt` all depend on pulse; pulse depends on none of them. It exposes stable,
  documented seams (e.g. the Lens read-only export boundary) so consumers never
  reach into `_`-prefixed internals.
- **lens reasons over the web** that pulse holds, and `monitor` observes it - both
  read-only consumers of the same content-addressed graph.
- **molgang and vbank are plugins (knitwebs)** that live *on* pulse: molgang is a
  domain (chemistry), vbank is governance. vbank governs molgang (and any plugin)
  via voting; molgang weaves knowledge; both settle in pulse's integer ledger.
- **virtualpc coordinates agents** that act on the fabric; **bt** is an alternative
  transport so the same web reaches off-grid.
- **docs documents all of them; monitor watches all of them; news/knitweb.github.io
  are the public face.** These are different *facets of the same system*, not
  dependencies you have to thread through code.

## Why multiple repos (the answer to "how does a multi-repo project work?")

- **No spaghetti by construction.** Separate repos cannot import each other's
  internals - the only contact points are published interfaces (e.g. pulse's
  export boundary, molgang's `/api`). A change in one repo can only break another
  if it breaks a *documented seam*, which is exactly what you test.
- **Where does a change go?** By the seam it touches: engine/ledger/canonical ->
  `pulse`; reasoning/retrieval -> `lens`; the game/relay -> `molgang`;
  voting/treasury -> `vbank`; dashboards/graph view -> `monitor`; agent runtime ->
  `virtualpc`; off-grid transport -> `bt`; docs -> `docs`. The repo *is* the
  routing.
- **How is the whole tested?** Each repo has its own test suite against its seam;
  cross-repo contracts are pinned (e.g. a CID/byte-stability vector that both the
  Python engine and a PHP node must reproduce, so the wire format can't drift).
  Integration is verified at the boundary (one node talks to another over the same
  signed, content-addressed records), not by one giant build.

## Map to the 3D "Begrippen" view

The Begrippen tab in the 3D graph renders exactly this: each repo is a node,
coloured by layer (substrate / brain / apps+governance / meta / edge-layers), and
each edge carries the 2-word relation (e.g. pulse->lens "engine substrate",
vbank->pulse "controls emission", monitor->pulse "observes the web"). Hover a link
to read the relation; the doc above is the prose version of that graph.
