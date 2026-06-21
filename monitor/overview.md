# knitweb/monitor — repository instructions

**What it is.** A **zero-dependency, cross-platform** dashboard for knitweb nodes, the woven knowledge graph, and live MOLGANG sessions. Pure stdlib Python — identical on **Windows / macOS / Linux**, no build step.

## Run (no install needed)
```bash
git clone https://github.com/knitweb/monitor && cd monitor
python -m knitweb_monitor --molgang http://localhost:8765 --open
# Windows:  py -m knitweb_monitor --molgang http://localhost:8765
```
or `pip install knitweb-monitor && knitweb-monitor --molgang http://localhost:8765 --open` → http://127.0.0.1:8990

## Options
`--molgang URL` (repeatable) · `--node LABEL=WALLET[:PORT]` (repeatable) · `--port` · `--host` · `--knitweb-src PATH` · `--open`. Env: `KNITWEB_MONITOR_PORT`, `KNITWEB_MONITOR_MOLGANG`, `KNITWEB_SRC`.

## Tabs
- **Nodes** — balance / address / transfer feed / daemon liveness (port check).
- **Knowledge graph** — ledger transfers (blue) + MOLGANG fabric (`subject—rel→object`, ⚓ = OriginTrail-anchored); force-directed (networkx if present, else a pure-Python fallback).
- **MOLGANG** — every session's tables, seated players, woven fabric.

## Optional extras (auto-detected, never required)
`networkx` → nicer layout · `knitweb` → reads node ledger state. Without them it still works (built-in layout + HTTP-only MOLGANG/graph).

**Read-only & localhost-bound by default** — it polls HTTP + reads wallet snapshots; never writes state or moves funds.

Open: give it a web home at 5mart.ml (#59/#60) — deploy a **static** export of the graph since shared hosting can't run the Python server.
