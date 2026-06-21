# knitweb/pulse — repository instructions

**What it is.** The pure-Python **knitweb** engine: a credibly-neutral peer-to-peer crypto *web*. Pay-token **PLS** ("pulses"), value-unit **Fiber**, validation **Knit/Loom→knitweb**, heartbeat **Pulse**. PoUW issuance (no premine), a fabric **Web** of knowledge, **OriginTrail** anchoring, and a P2P **FabricNode**.

## Get it running
```bash
git clone https://github.com/knitweb/pulse && cd pulse
python3 -m venv .venv && . .venv/bin/activate
pip install -e .                      # needs Python ≥ 3.12 + `cryptography`
PYTHONPATH=src python3 -m pytest tests/property -q   # ~hundreds of proofs, green
```

## Run a node / move value
```bash
python -m knitweb.app.cli wallet  --out alice.json --genesis 100   # dev faucet only (no premine on mainnet)
python -m knitweb.app.cli address --wallet alice.json
python -m knitweb.app.cli node    --wallet alice.json --listen 127.0.0.1:8900
python -m knitweb.app.cli pay      --wallet alice.json --peer 127.0.0.1:8901 --to <pub> --amount 5
```
Wallets are canonical-CBOR `AccountNode` snapshots (full braid). Identity helpers: `AccountNode.from_seed(id)` derives a deterministic wallet (⚠️ a *public* id → a publicly-derivable key — never custody value on one).

## Non-negotiables
- Crypto = **secp256k1 ECDSA + SHA-256**; money/state are **integers** (no floats); all hashing via `core.canonical` (float-free CBOR + CIDv1). **Never change a signed-record field/kind** — it changes every CID and signature.
- One pipeline, proofs-first: every change ends with a runnable test + commit.

## Layers
L0 core · L1 ledger (blob/fiber/loom/knit/braid/node) · L2 p2p (asyncio feed sync) · L3 fabric (Web) · L4 pouw (sampled re-exec + BFT quorum + dispute) · L5 looms · L6 token (demand-gated mint) + anchor (OriginTrail).

## Open priorities (MoSCoW on the board)
- **Should** #19 version drift · #22 JSON-LD/DKG export · #24 FabricNode hardening (equivocation/Merkle/DHT).
- Known finding: FabricNode convergence witness (`web_state_root`) covers **nodes, not edges** — propagate links + fold edge CIDs in.
