# knitweb/molgang — repository instructions

**What it is.** A peer-to-peer **chemistry knowledge game** on knitweb. Players walk into a bar, take a seat, **knit chemistry terms** (spend silk), peers **vote with a pulse**; a **quorum weaves** the term into the shared fabric as a **Fiber** (a real knowledge-graph node/edge), anchored to OriginTrail. Economy = **PLS + silk + reputation, no NFTs**.

## Run the bar (Python)
```bash
git clone https://github.com/knitweb/molgang && cd molgang
git clone https://github.com/knitweb/pulse ../pulse          # the engine, as a sibling
python3 -m venv .venv && . .venv/bin/activate && pip install -e . -e ../pulse
molgang serve --port 8765 --world ~/.molgang/world.json --db ~/.molgang/registry.db
# explorer over the woven web:
molgang.explorer --web ~/.molgang/world.json --port 8990
```

## The machine API (same one bots/agents drive)
`GET /api/state?sid=` · `POST /api/join {name,avatar,device}` → `{sid,address}` · `POST /api/sit {sid,table}` · `POST /api/propose {sid,term}` · `POST /api/vote {sid,pid,verdict}` · `GET /api/web` · `GET /api/graph` · `POST /api/certificate {sid}` (PoUW PDF — **embeds the private key**, see #55) · `POST /api/spiral/*`.

The browser client is **path-prefix-safe** — works at `/` or under `/molgang/`.

## Three deploy shapes (pick one canonical — #58)
1. **Python `molgang serve`** — the authentic engine (needs an always-on box).
2. **PHP/MySQL dapp** (`php/`) — request-driven, runs on **shared hosting** (live at 5mart.ml/molgang). See the 5mart.ml runbook.
3. **Static UI + containerized backend** (#35/#47).

## Open priorities
- **Must** #53 weave-quorum scales with seat count (crowded/stale tables never weave) · #55 cert leaks private key · #74 'back to floor' missing refresh.
- **Should** #54 e2e+agentplay · #57 faucet redeploy · #75 dynamic 24-seat tables · #76 email-subscribe (Blowfish) + daily cert · #30/#32/#33 multilingual chemistry KG.

## Test
```bash
PYTHONPATH=src python3 -m pytest -q          # Python
php php/tests/smoke.php                       # PHP dapp (SQLite, no server)
```
